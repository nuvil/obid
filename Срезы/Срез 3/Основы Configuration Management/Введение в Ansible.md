## Первый блок
1. **Требования к хосту для установки Ansible**:  
   - **Операционная система**: Linux (Ubuntu, CentOS, RHEL и др.), macOS или Windows с WSL.  
   - **Python**: Версия 2.7 или 3.5+ (для Ansible 2.10 и выше предпочтительно Python 3).  
   - **Зависимости**: Библиотеки Python, такие как `paramiko`, `PyYAML`, `Jinja2`.  
   - **Сеть**: Доступ по SSH к управляемым хостам.  
   - **Прочее**: Утилиты `ssh`, `scp`, права на установку пакетов.  

2. **Хосты, управляемые Ansible**:  
   Любые хосты с поддержкой SSH и Python (2.7 или 3.x):  
   - Linux/Unix (Ubuntu, CentOS, etc.).  
   - macOS.  
   - Windows (с PowerShell и WinRM).  
   - Сетевые устройства (через модули, поддерживающие API или CLI).  
   - Облачные сервисы (AWS, Azure, GCP) через соответствующие модули.  

3. **Файл .ansible.cfg**:  
   - Это конфигурационный файл Ansible, задающий глобальные настройки.  
   - Используется для настройки путей (например, к inventory), параметров SSH, привилегий, путей к модулям и плагинам.  
   - Располагается в `/etc/ansible/ansible.cfg`, `~/.ansible.cfg` или в директории проекта.  

4. **Inventory file**:  
   - Файл, содержащий список управляемых хостов и их групп.  
   - Форматы:  
     - **INI**: Простой текстовый формат (например, `[webservers] host1 ansible_host=192.168.1.10`).  
     - **YAML**: Структурированный формат (например, `all: hosts: host1: ansible_host: 192.168.1.10`).  
     - **JSON** (реже, через плагины).  
   - Может быть статическим (файл) или динамическим (генерируется скриптами).  

5. **Отличие модуля и плагина в Ansible**:  
   - **Модуль**: Выполняет конкретную задачу на хосте (например, `copy`, `user`, `apt`). Запускается в playbook’ах.  
   - **Плагин**: Расширяет функциональность Ansible (например, обработка вывода, фильтры Jinja2, callback’ы). Работает на стороне управляющего хоста, не выполняется на целевом хосте.  

6. **Ansible-playbook, play, task**:  
   - **Ansible-playbook**: Исполняемый файл (скрипт YAML), содержащий одну или несколько play. Запускается командой `ansible-playbook playbook.yml`.  
   - **Play**: Блок в playbook, определяющий задачи для группы хостов (например, установка пакета на `[webservers]`).  
   - **Task**: Отдельная задача внутри play, выполняющая модуль (например, `copy` для копирования файла).  

7. **Ansible-роль**:  
   - Модульная структура для организации задач, переменных, шаблонов и файлов.  
   - Содержит поддиректории: `tasks`, `templates`, `vars`, `defaults`, `files`, `handlers`.  
   - Используется для повторного использования и упрощения управления конфигурацией.  

8. **Задание имени пользователя для подключения**:  
   - В inventory: `ansible_user=username` (например, `host1 ansible_host=192.168.1.10 ansible_user=admin`).  
   - В `.ansible.cfg`: `[defaults] remote_user = username`.  
   - В командной строке: `ansible-playbook -u username playbook.yml`.  

9. **Задание пути к SSH-ключу**:  
   - В inventory: `ansible_ssh_private_key_file=/path/to/key.pem`.  
   - В `.ansible.cfg`: `[defaults] private_key_file = /path/to/key.pem`.  
   - В командной строке: `ansible-playbook --private-key=/path/to/key.pem playbook.yml`.  

10. **Копирование файла на целевой хост**:  
    Используется модуль `copy`:  
    ```yaml
    - name: Copy file to remote host
      ansible.builtin.copy:
        src: /local/path/to/file
        dest: /remote/path/to/file
        mode: '0644'
    ```  

11. **Создание пользователя в Linux**:  
    Используется модуль `user`:  
    ```yaml
    - name: Create a new user
      ansible.builtin.user:
        name: newuser
        password: "{{ 'password' | password_hash('sha512') }}"
        state: present
        shell: /bin/bash
        home: /home/newuser
    ```  

12. **Модуль template**:  
    - Используется для создания файлов на основе шаблонов Jinja2, подставляя переменные.  
    - Пример: Генерация конфигурационного файла с динамическими значениями (например, IP-адреса).  
    ```yaml
    - name: Deploy config file
      ansible.builtin.template:
        src: template.j2
        dest: /etc/app/config.conf
        mode: '0644'
    ```  

13. **Выполнение задач от привилегированного пользователя**:  
    - Используйте параметр `become`:  
      ```yaml
      - name: Task with sudo
        ansible.builtin.apt:
          name: nginx
          state: present
        become: yes
        become_method: sudo
        become_user: root
      ```  
    - В inventory: `ansible_become=yes`, `ansible_become_user=root`.  
    - В командной строке: `ansible-playbook --become playbook.yml`.  

Если нужны уточнения или примеры, дайте знать!

## Второй блок
1. **Способы эскалации привилегий в Ansible (become methods)**:  
   Ansible использует параметр `become` для эскалации привилегий. Доступные методы:  
   - `sudo`: Использует команду `sudo` (по умолчанию).  
   - `su`: Переключается на другого пользователя через `su`.  
   - `pbrun`: Использует PowerBroker для эскалации.  
   - `pfexec`: Использует Solaris Privilege Framework.  
   - `doas`: Использует OpenBSD `doas`.  
   - `dzdo`: Centrify’s DirectAuthorize.  
   - `ksu`: Kerberos `ksu`.  
   - `runas`: Для Windows.  
   - `machinectl`: Для systemd containers.  
   Настройка:  
   ```yaml
   - name: Run task with sudo
     ansible.builtin.apt:
       name: nginx
       state: present
     become: yes
     become_method: sudo
   ```

2. **Handler и его назначение**:  
   - **Handler** — это специальная задача, которая выполняется только при уведомлении (notify) от другой задачи, обычно для реакции на изменения (например, перезапуск службы).  
   - Используется для минимизации ненужных действий (например, перезапуска службы только при изменении конфигурации).  
   - Пример использования:  
     ```yaml
     - name: Copy nginx config
       ansible.builtin.copy:
         src: nginx.conf
         dest: /etc/nginx/nginx.conf
       notify: Restart nginx

     handlers:
       - name: Restart nginx
         ansible.builtin.service:
           name: nginx
           state: restarted
     ```

3. **Этап Gathering Facts**:  
   - На этапе **Gathering Facts** Ansible собирает информацию о целевом хосте (ОС, версия, IP, диски, процессор и т.д.) и сохраняет её в переменных (например, `ansible_facts`).  
   - Эти данные используются в playbook’ах для условных операций или шаблонов.  
   - Выполняется автоматически перед началом выполнения задач, если не отключено.  

4. **Узнать имя и версию дистрибутива Linux**:  
   Используйте переменные из `ansible_facts`:  
   - Имя дистрибутива: `{{ ansible_facts['distribution'] }}`  
   - Версия дистрибутива: `{{ ansible_facts['distribution_version'] }}`  
   Пример:  
   ```yaml
   - name: Print distribution info
     ansible.builtin.debug:
       msg: "OS: {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }}"
   ```

5. **Пропуск этапа Gathering Facts**:  
   Да, можно пропустить, указав `gather_facts: no` в playbook:  
   ```yaml
   - hosts: all
     gather_facts: no
     tasks:
       - name: Task without facts
         ansible.builtin.debug:
           msg: "Skipping facts gathering"
   ```  
   Или в командной строке: `ansible-playbook -e 'gather_facts=false' playbook.yml`.  
   Это ускоряет выполнение, но переменные `ansible_facts` будут недоступны.  

6. **Найти и заменить строку с помощью регулярного выражения**:  
   Используется модуль `replace`:  
   ```yaml
   - name: Replace string in file
     ansible.builtin.replace:
       path: /path/to/file
       regexp: 'old_string.*'
       replace: 'new_string'
       backup: yes
   ```  
   - `regexp`: Регулярное выражение для поиска.  
   - `replace`: Новая строка для замены.  
   - `backup`: Создаёт резервную копию файла перед изменением.  

7. **Регистрация переменной в runtime**:  
   Используется параметр `register` для сохранения результата задачи в переменную:  
   ```yaml
   - name: Check file existence
     ansible.builtin.stat:
       path: /path/to/file
     register: file_status

   - name: Print file status
     ansible.builtin.debug:
       msg: "File exists: {{ file_status.stat.exists }}"
   ```  
   Переменная `file_status` содержит результат выполнения модуля `stat`.  

8. **Ansible Galaxy**:  
   - **Ansible Galaxy** — это репозиторий для обмена ролями, коллекциями и модулями Ansible.  
   - Содержит готовые роли и коллекции, созданные сообществом, для упрощения автоматизации.  
   - Доступен через сайт `galaxy.ansible.com` или команду `ansible-galaxy`.  

9. **Установка модуля/плагина из Ansible Galaxy**:  
   - Для установки **роли**:  
     ```bash
     ansible-galaxy install username.rolename
     ```  
     Пример: `ansible-galaxy install geerlingguy.nginx`.  
   - Для установки **коллекции** (содержит модули, плагины и роли):  
     ```bash
     ansible-galaxy collection install namespace.collection
     ```  
     Пример: `ansible-galaxy collection install community.general`.  
   - Роли устанавливаются в `~/.ansible/roles` или в директорию, указанную в `ansible.cfg`.  
   - Коллекции устанавливаются в `~/.ansible/collections` или в проект.  
   - Использование в playbook:  
     ```yaml
     - hosts: all
       roles:
         - geerlingguy.nginx
       collections:
         - community.general
     ```

Если нужны дополнительные примеры или уточнения, дайте знать!

## Третий блок

1. **Поиск всех файлов с расширением .log и запись их путей в переменную в playbook**:  
   Для поиска файлов используется модуль `ansible.builtin.find`, а результат сохраняется в переменную с помощью `register`. Пример:  
   ```yaml
   - name: Find all .log files
     ansible.builtin.find:
       paths: /path/to/directory
       patterns: "*.log"
       recurse: yes
     register: log_files

   - name: Print found log files
     ansible.builtin.debug:
       msg: "Found log files: {{ log_files.files | map(attribute='path') | list }}"
   ```  
   - `paths`: Директория для поиска.  
   - `patterns`: Шаблон для фильтрации файлов (например, `*.log`).  
   - `recurse: yes`: Включает рекурсивный поиск в поддиректориях.  
   - `log_files.files`: Содержит список найденных файлов, где `path` — полный путь к каждому файлу.  
   Переменная `log_files.files` содержит пути, которые можно использовать дальше в playbook.  

2. **Что можно сделать с помощью плагина lookup**:  
   Плагин `lookup` позволяет получать данные из внешних источников на управляющем хосте (не на целевом). Примеры использования:  
   - Чтение содержимого файла:  
     ```yaml
     - name: Read file content
       ansible.builtin.debug:
         msg: "{{ lookup('file', '/path/to/file.txt') }}"
     ```  
   - Получение переменных из JSON/YAML:  
     ```yaml
     - name: Read JSON file
       ansible.builtin.debug:
         msg: "{{ lookup('ansible.builtin.file', 'vars.json') | from_json }}"
     ```  
   - Получение данных из окружения:  
     ```yaml
     - name: Get environment variable
       ansible.builtin.debug:
         msg: "{{ lookup('env', 'HOME') }}"
     ```  
   - Чтение строк из CSV:  
     ```yaml
     - name: Read CSV file
       ansible.builtin.debug:
         msg: "{{ lookup('csvfile', 'user1 file=/path/to/users.csv delimiter=,') }}"
     ```  
   - Получение данных из внешних источников (например, URL):  
     ```yaml
     - name: Get data from URL
       ansible.builtin.debug:
         msg: "{{ lookup('url', 'https://api.example.com/data') }}"
     ```  
   Плагины `lookup` работают на стороне управляющего узла и не требуют выполнения на целевых хостах.  

3. **Как ускорить выполнение playbook на большом количестве хостов**:  
   - **Увеличение количества форков**:  
     Увеличьте параметр `forks` в `ansible.cfg` или командной строке, чтобы Ansible выполнял задачи параллельно на нескольких хостах:  
     ```ini
     [defaults]
     forks = 50
     ```  
     Или: `ansible-playbook -f 50 playbook.yml`.  
   - **Отключение Gathering Facts**:  
     Пропустите сбор фактов, если они не нужны:  
     ```yaml
     - hosts: all
       gather_facts: no
     ```  
   - **Использование асинхронного режима**:  
     Запускайте задачи асинхронно с параметром `async` и `poll`:  
     ```yaml
     - name: Async task
       ansible.builtin.command: long_running_command
       async: 3600
       poll: 10
     ```  
   - **Включение pipelining**:  
     Включите SSH pipelining в `ansible.cfg` для уменьшения количества SSH-соединений:  
     ```ini
     [ssh_connection]
     pipelining = True
     ```  
   - **Оптимизация inventory**:  
     Используйте динамический inventory или фильтруйте хосты, чтобы уменьшить количество целевых узлов.  
   - **Кэширование фактов**:  
     Включите кэширование фактов (например, с Redis или JSON):  
     ```ini
     [defaults]
     fact_caching = redis
     fact_caching_timeout = 86400
     ```  
   - **Использование стратегий**:  
     Измените стратегию выполнения с `linear` на `free` (параллельное выполнение без ожидания):  
     ```yaml
     - hosts: all
       strategy: free
       tasks:
         ...
     ```  
   - **Оптимизация задач**:  
     Минимизируйте количество задач, используйте роли, избегайте ненужных модулей (например, `command` вместо `shell` для простых команд).  
   - **Ограничение хостов**:  
     Используйте параметр `--limit` для обработки подмножества хостов:  
     ```bash
     ansible-playbook playbook.yml --limit "web_servers"
     ```  

Если нужны дополнительные примеры или уточнения, дайте знать!