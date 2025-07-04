## Первый блок
1. **Что такое Linux? Что такое GNU/Linux?**

·         **Linux** — это ядро операционной системы, созданное Линусом Торвальдсом в 1991 году. Оно управляет аппаратными ресурсами и обеспечивает взаимодействие программ с железом.

·         **GNU/Linux** — это полная операционная система, объединяющая ядро Linux и набор утилит проекта GNU (например, компилятор GCC, библиотеки, утилиты командной строки). Название подчеркивает вклад проекта GNU в создание свободной ОС.

---

2. **Что такое дистрибутив Linux?**

Дистрибутив — это сборка на основе ядра Linux, включающая:

·         Пакеты программ (браузеры, офисные приложения, серверные утилиты).

·         Систему управления пакетами (APT, DNF, Pacman).

·         Средства настройки и установки.

·         Графическое окружение (GNOME, KDE и др.).

·         Примеры: Ubuntu, Debian, Fedora, Arch Linux.

---

3. **Чем отличаются дистрибутивы?**

·         **Пакетные менеджеры**: Debian-based (APT, .deb), RHEL-based (DNF/YUM, .rpm), Arch (Pacman).

·         **Политика обновлений**: Rolling Release (Arch) vs стабильные версии (Debian).

·         **Целевое использование**: Серверы (RHEL), десктоп (Ubuntu), минимализм (Alpine).

·         **Сообщество vs коммерция**: Ubuntu (Canonical), RHEL (Red Hat), openSUSE.

·         **Графические оболочки**: Разные предустановленные среды (GNOME, KDE, XFCE).

---

4. **Установка пакетов**

·         **Debian-based** (Ubuntu, Debian):

sudo apt update && sudo apt install <пакет>  # через APT

sudo dpkg -i <файл.deb>                     # вручную

·         **RHEL-based** (CentOS, Fedora):

sudo dnf install <пакет>      # Fedora/CentOS 8+

sudo yum install <пакет>      # CentOS 7 и старее

---

5. **Работа с файлами и дисками**

·         **Показать все директории (включая скрытые)**:

ls -la

·         **Размер файлов в директории (МБ)**:

du -sh *      # -s — суммарно, -h — читаемый формат

·         **Свободное место на дисках**:

df -h         # -h отображает размеры в GB/MB

---

6. **Перенаправление вывода**

·         **Передать вывод команды на вход другой**:

command1 | command2   # Например: ls | grep .txt

·         **Записать вывод в файл**:

command > output.txt           # Перезаписать

command >> output.txt          # Дописать

command **2**> errors.txt          # Только ошибки

command > output.txt **2**>**&1**      # Вывод + ошибки в один файл

---

7. **Exit Code**

·         **Exit Code** — код завершения программы (0–255).

o    **0** — успех.

o    **Не 0** — ошибка (код зависит от программы).

·         **Проверить код**:

echo $?      # После выполнения команды

---

8. **Linux File Hierarchy Structure**

Стандарт FHS определяет структуру каталогов:

·         **/** — корневая директория.

·         **/bin** — основные исполняемые файлы.

·         **/etc** — конфигурационные файлы.

·         **/home** — домашние директории пользователей.

·         **/var** — изменяемые данные (логи, базы данных).

·         **/usr** — пользовательские программы и библиотеки.

---

9. **Пользователи и группы**

·         **Создать пользователя**:

sudo useradd <имя>       # Базовое создание

sudo adduser <имя>       # Интерактивный вариант (в Debian)

·         **Изменить группу пользователя**:

sudo usermod -aG <группа> <пользователь>   # Добавить в группу

sudo gpasswd -d <пользователь> <группа>    # Удалить из группы

---

10. **Права доступа**

·         **Владелец** — пользователь, создавший файл.

·         **Группа-владелец** — группа, связанная с файлом.

·         **Права**:

o    **Буквенный формат**: u (владелец), g (группа), o (остальные).  
Пример: chmod u+rwx,g+rx,o-r file.

o    **Числовой формат**: 3 цифры (rwx).  
Пример: chmod 755 file → rwxr-xr-x.

·         **Изменить владельца**:

sudo chown <пользователь>:<группа> <файл>

·         **Изменить права**:

chmod <права> <файл>     # Например: chmod 644 file.txt

---

Примеры

·         **Создать пользователя "john" и добавить его в группу "admins"**:

sudo useradd john

sudo usermod -aG admins john

·         **Дать права на выполнение скрипта**:

chmod +x script.sh

·         **Проверить свободное место**:

df -h | grep /dev/sda1
## Второй блок
**1. Какие Linux-дистрибутивы лучше подходят для Docker-образов? Почему?**  
Лучше использовать минималистичные дистрибутивы:

·         **Alpine Linux**: основан на musl libc и BusyBox, образы очень компактные (5–10 МБ).

·         **Scratch**: "пустой" образ для самостоятельной сборки.

·         **Distroless** (от Google): содержит только приложение и его зависимости, без shell и пакетного менеджера.

·         **Debian Slim/Ubuntu Minimal**: уменьшенные версии, баланс между размером и совместимостью.

**2. Как добавить новый репозиторий в пакетный менеджер?**

·         **APT (Debian/Ubuntu)**:

sudo add-apt-repository "deb [URL] [распределение] [компоненты]" 

### Или вручную: 

echo "deb [URL] [распределение] [компоненты]" | sudo tee /etc/apt/sources.list.d/имя_репозитория.list 

sudo apt update 

·         **YUM/DNF (RHEL/CentOS)**:  
Создать файл имя.repo в /etc/yum.repos.d/ с содержимым:

 [имя-репозитория]

name=Описание

baseurl=URL

enabled=1

gpgcheck=0/1

**3. Выполнить команду, только если предыдущая успешна**Использовать оператор &&:

bash

Copy

Download

command1 && command2  # command2 запустится, только если command1 завершился с кодом 0.

**4. Выполнить команду, только если предыдущая неуспешна**Использовать оператор ||:

command1 || command2  # command2 запустится, только если command1 завершился с ненулевым кодом.

**5. Типы ссылок в Linux**

·         **Жёсткие ссылки (hard links)**:

o    Ссылаются на тот же inode, что и оригинал.

o    Не работают для директорий и разных файловых систем.

o    Удаление оригинала не влияет на ссылку.

·         **Символические ссылки (soft/symlinks)**:

o    Содержат путь к оригиналу.

o    Могут указывать на директории и файлы в других ФС.

o    Становятся битыми при удалении оригинала.

**6. Создать ссылку на файл/директорию**

·         **Жёсткая ссылка**:

ln /путь/к/оригиналу /путь/к/ссылке 

·         **Символическая ссылка**:

ln -s /путь/к/оригиналу /путь/к/ссылке 

**7. Создать пользователя в non-interactive режиме**Использовать useradd с параметрами:

sudo useradd -m -s /bin/bash -c "Комментарий" имя_пользователя 

·         -m: создать домашнюю директорию.

·         -s: указать оболочку.

**8. Поменять пароль пользователя**Через chpasswd:

echo "имя_пользователя:новый_пароль" | sudo chpasswd 

**9. Execute бит на директории**Позволяет войти в директорию (команда cd) и получить доступ к её содержимому. Без этого бита нельзя просматривать файлы внутри, даже если есть права на чтение.

**10. Минимальные права для доступа владельца**

·         **Директория**: --x (100) — войти, но без чтения списка файлов. Для чтения списка нужно r-x (500).

·         **Файл**: r-- (400) — чтение.

·         **Пример**:

o    Директория: 500 (dr-x------).

o    Файл: 400 (-r--------).
## Третий блок
1. Установка Linux на группу серверов в корпоративной сети

Для автоматизации установки используйте:

·         **PXE-загрузку** (сетевую установку) в сочетании с **Kickstart** (RHEL/CentOS) или **Preseed** (Debian/Ubuntu). Это позволяет серверам загружаться по сети и устанавливать ОС без ручного вмешательства.

·         Инструменты управления конфигурацией: **Ansible**, **Puppet**, **Chef**. Они помогают развертывать ОС и настраивать серверы через playbook/манифесты.

·         Образы дисков (ISO) с предварительной настройкой для массового клонирования (например, через **Cobbler**).

---

2. Фиксация версии пакета

·         **Debian/Ubuntu**:

sudo apt-mark hold <имя_пакета>  # запретить обновление

sudo apt-mark unhold <имя_пакета>  # снять запрет

·         **RHEL/CentOS/Fedora**:

o    Для yum: добавить в /etc/yum.conf строку exclude=<имя_пакета>.

o    Для dnf: установите плагин dnf-plugin-versionlock, затем:

sudo dnf versionlock add <имя_пакета>

---

3. Сборка .deb или .rpm пакетов

·         **.deb (Debian/Ubuntu)**:

1.    Создайте структуру проекта (например, myapp/DEBIAN/control).

2.    Напишите control-файл с метаданными.

3.    Соберите пакет:

dpkg-deb --build myapp

o    Альтернатива: используйте dh_make и debuild для автоматизации.

·         **.rpm (RHEL/CentOS)**:

1.    Создайте .spec-файл.

2.    Соберите пакет:

rpmbuild -ba myapp.spec

o    Утилита **fpm** упрощает сборку для обоих форматов.

---

4. Размер файлов в текущей директории (сортировка по убыванию)

du -sh * | sort -hr

·         du -sh * — показывает размер каждого элемента в текущей директории.

·         sort -hr — сортирует строки по числовому значению в обратном порядке (от большего к меньшему).

---

5. Назначение системных файлов

·         **/etc/sudoers**: Настройки прав для команды sudo (определяет, кто и какие команды может выполнять с привилегиями root).

·         **/etc/passwd**: Список пользователей, их UID, GID, домашние директории и шеллы.

·         **/etc/shadow**: Хранит хеши паролей пользователей и параметры их срока действия.

---

6. Разрешить выполнение всех команд без ввода пароля

Добавьте в /etc/sudoers (через visudo):

username ALL=(ALL) NOPASSWD: ALL

·         username — имя пользователя.

·         **Важно**: Используйте visudo, чтобы избежать синтаксических ошибок.

---

7. Sticky bit, SUID, SGID

·         **Sticky bit** (chmod +t):

o    Для директорий: разрешает удаление файлов только их владельцам (пример: /tmp).

·         **SUID** (chmod u+s):

o    Для файлов: программа запускается с правами владельца файла (пример: /usr/bin/passwd).

·         **SGID** (chmod g+s):

o    Для директорий: новые файлы наследуют группу директории.

o    Для файлов: программа запускается с правами группы файла.