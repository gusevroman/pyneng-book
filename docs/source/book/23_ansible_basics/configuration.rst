Конфигурационный файл
---------------------

Настройки Ansible можно менять в конфигурационном файле.

Конфигурационный файл Ansible может храниться в разных местах (файлы
перечислены в порядке уменьшения приоритета): \* ANSIBLE\_CONFIG
(переменная окружения) \* ansible.cfg (в текущем каталоге) \*
.ansible.cfg (в домашнем каталоге пользователя) \*
/etc/ansible/ansible.cfg

Ansible ищет файл конфигурации в указанном порядке и использует первый
найденный (конфигурация из разных файлов не совмещается).

В конфигурационном файле можно менять множество параметров. Полный
список параметров и их описание можно найти в
`документации <http://docs.ansible.com/ansible/devel/intro_configuration.html>`__.

В текущем каталоге должен быть инвентарный файл myhosts:

::

    [cisco-routers]
    192.168.100.1
    192.168.100.2
    192.168.100.3

    [cisco-switches]
    192.168.100.100

В текущем каталоге надо создать такой конфигурационный файл ansible.cfg:

::

    [defaults]

    inventory = ./myhosts
    remote_user = cisco
    ask_pass = True

Настройки в конфигурационном файле: \* ``[defaults]`` - эта секция
конфигурации описывает общие параметры по умолчанию \*
``inventory = ./myhosts`` - параметр inventory позволяет указать
местоположение инвентарного файла. \* Если настроить этот параметр, не
придется указывать, где находится файл, при каждом запуске Ansible \*
``remote_user = cisco`` - от имени какого пользователя будет
подключаться Ansible \* ``ask_pass = True`` - этот параметр аналогичен
опции --ask-pass в командной строке. Если он выставлен в конфигурации
Ansible, то уже не нужно указывать его в командной строке.

Теперь вызов ad-hoc команды будет выглядеть так:

::

    $ ansible cisco-routers -m raw -a "sh ip int br"

Теперь не нужно указывать инвентарный файл, пользователя и опцию
--ask-pass.

gathering
~~~~~~~~~

По умолчанию Ansible собирает факты об устройствах.

Факты - это информация о хостах, к которым подключается Ansible. Эти
факты можно использовать в playbook и шаблонах как переменные.

Сбором фактов, по умолчанию, занимается модуль
`setup <http://docs.ansible.com/ansible/devel/setup_module.html>`__.

Но для сетевого оборудования модуль setup не подходит, поэтому сбор
фактов надо отключить. Это можно сделать в конфигурационном файле
Ansible или в playbook.

    Для сетевого оборудования нужно использовать отдельные модули для
    сбора фактов (если они есть). Это рассматривается в разделе
    `ios\_facts <../3_network_modules/ios_facts.md>`__.

Отключение сбора фактов в конфигурационном файле:

.. code:: yml

    gathering = explicit

host\_key\_checking
~~~~~~~~~~~~~~~~~~~

Параметр host\_key\_checking отвечает за проверку ключей при подключении
по SSH. Если указать в конфигурационном файле
``host_key_checking=False``, проверка будет отключена.

Это полезно, когда с управляющего хоста Ansible надо подключиться к
большому количеству устройств первый раз.

Чтобы проверить этот функционал, надо удалить сохраненные ключи для
устройств Cisco, к которым уже выполнялось подключение.

    В линукс они находятся в файле ~/.ssh/known\_hosts.

Если выполнить ad-hoc команду после удаления ключей, вывод будет таким:

::

    $ ansible cisco-routers -m raw -a "sh ip int br"

.. figure:: https://raw.githubusercontent.com/natenka/PyNEng/master/images/15_ansible/host_key_checking.png
   :alt: host\_key\_checking

   host\_key\_checking
Добавляем в конфигурационный файл параметр host\_key\_checking:

::

    [defaults]

    inventory = ./myhosts

    remote_user = cisco
    ask_pass = True

    host_key_checking=False

И повторим ad-hoc команду:

::

    $ ansible cisco-routers -m raw -a "sh ip int br"

.. figure:: https://raw.githubusercontent.com/natenka/PyNEng/master/images/15_ansible/host_key_checking2.png
   :alt: host\_key\_checking2

   host\_key\_checking2
Обратите внимание на строки:

::

     Warning: Permanently added '192.168.100.1' (RSA) to the list of known hosts.

Ansible сам добавил ключи устройств в файл ~/.ssh/known\_hosts. При
подключении в следующий раз этого сообщения уже не будет.

    Другие параметры конфигурационного файла можно посмотреть в
    документации. Пример конфигурационного файла в `репозитории
    Ansible <https://github.com/ansible/ansible/blob/devel/examples/ansible.cfg>`__.
