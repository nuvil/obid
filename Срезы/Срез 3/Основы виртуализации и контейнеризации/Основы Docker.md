## Первый блок
### 1. Из каких компонентов состоит Docker?
Docker состоит из следующих основных компонентов:
- **Docker Engine**: Основной движок, включающий Docker Daemon (управляет контейнерами, образами, сетями и хранилищем) и REST API для взаимодействия.
- **Docker CLI**: Интерфейс командной строки для взаимодействия с Docker Engine.
- **Docker Images (Образы)**: Шаблоны для создания контейнеров, содержащие приложение, зависимости и конфигурации.
- **Docker Containers (Контейнеры)**: Экземпляры, созданные из образов, которые выполняются как изолированные процессы.
- **Docker Registry**: Хранилище для образов, например, Docker Hub или частные реестры.
- **Docker Compose**: Инструмент для определения и управления многоконтейнерными приложениями через YAML-файлы.
- **Docker Networking**: Система для управления сетевым взаимодействием между контейнерами.
- **Docker Storage**: Управление томами и хранилищем для сохранения данных контейнеров.

---

### 2. Что такое образ?
**Образ (Docker Image)** — это неизменяемый шаблон, содержащий приложение, его зависимости, библиотеки, конфигурации и инструкции для запуска. Образы создаются с помощью Dockerfile и хранятся в реестре (например, Docker Hub). Они используются для создания контейнеров.

Пример: Образ `nginx:latest` содержит веб-сервер Nginx и все необходимое для его работы.

---

### 3. Что такое контейнер?
**Контейнер** — это экземпляр, созданный из образа, который выполняется как изолированный процесс на хост-системе. Контейнеры используют ядро хоста, но имеют собственную файловую систему, процессы и настройки, обеспечивая изоляцию.

Пример: Запуск контейнера из образа `nginx:latest` создаст работающий веб-сервер.

---

### 4. Что такое Docker Registry?
**Docker Registry** — это хранилище для Docker-образов, где они сохраняются и распределяются. Пример публичного реестра — **Docker Hub**. Частные реестры (например, Nexus, Harbor) используются для хранения собственных образов.

Пример: `docker pull nginx` загружает образ из Docker Hub.

---

### 5. Как скачать образ с приватного Docker Registry, требующего логин/пароль или токен?
Чтобы скачать образ из приватного реестра, нужно выполнить следующие шаги:

1. **Аутентификация**:
   ```bash
   docker login <registry-url> -u <username> -p <password>
   ```
   Или, если используется токен:
   ```bash
   docker login <registry-url> -u <username> -p <token>
   ```
   Пример: `docker login myregistry.example.com -u user -p mypassword`

2. **Скачивание образа**:
   ```bash
   docker pull <registry-url>/<image-name>:<tag>
   ```
   Пример: `docker pull myregistry.example.com/myapp:latest`

**Примечание**: Убедитесь, что у вас есть доступ к реестру, и используйте безопасные способы передачи пароля/токена (например, через переменные окружения).

---

### 6. Что такое Docker Compose?
**Docker Compose** — это инструмент для определения и управления многоконтейнерными приложениями с помощью YAML-файлов. Он позволяет описать сервисы, сети и тома, необходимые для работы приложения, и управлять ими одной командой.

Пример `docker-compose.yml`:
```yaml
version: '3'
services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
  db:
    image: mysql:latest
    environment:
      - MYSQL_ROOT_PASSWORD=example
```

Запуск: `docker-compose up`

---

### 7. Как запустить Docker контейнер?
Чтобы запустить контейнер, используйте команду `docker run`. Пример:
```bash
docker run -d --name my-container -p 8080:80 nginx:latest
```
- `-d`: Запуск в фоновом режиме.
- `--name my-container`: Имя контейнера.
- `-p 8080:80`: Проброс порта (хост:контейнер).
- `nginx:latest`: Образ, из которого создается контейнер.

---

### 8. Как выполнить команду внутри Docker контейнера?
Для выполнения команды внутри контейнера используйте `docker exec`. Пример:
```bash
docker exec -it my-container bash
```
- `-i`: Интерактивный режим.
- `-t`: Выделение псевдотерминала.
- `my-container`: Имя контейнера.
- `bash`: Команда для выполнения (например, открытие оболочки).

Для одноразовой команды:
```bash
docker exec my-container ls /app
```

---

### 9. Как посмотреть логи Docker контейнера?
Чтобы просмотреть логи контейнера, используйте команду `docker logs`. Пример:
```bash
docker logs my-container
```
- Для постоянного отслеживания логов в реальном времени добавьте флаг `-f`:
  ```bash
  docker logs -f my-container
  ```
- Логи показывают вывод `stdout` и `stderr` контейнера.

---

## Второй блок
### 1. Что такое слои в Docker?
**Слои в Docker** — это промежуточные неизменяемые уровни, из которых состоит Docker-образ. Каждый слой создается при выполнении инструкции в `Dockerfile` (например, `FROM`, `COPY`, `RUN`) и представляет изменения файловой системы, вызванные этой инструкцией. Слои кэшируются Docker для ускорения сборки и экономии места, так как повторно используемые слои не пересоздаются.

Пример:
```dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y curl
COPY ./app /app
```
- Каждый `RUN` и `COPY` создает новый слой.
- Образ состоит из базового слоя (`ubuntu:20.04`) и дополнительных слоев для каждой инструкции.

**Преимущества**: Экономия места (общие слои используются разными образами), ускорение сборки за счет кэширования.

---

### 2. Чем директива ADD отличается от COPY?
Обе директивы копируют файлы/папки из хост-системы в образ, но есть различия:

- **COPY**:
  - Копирует файлы или папки с хоста в образ без дополнительных функций.
  - Синтаксис: `COPY <источник> <назначение>`
  - Пример: `COPY ./app /app`
  - Используется для простого копирования.

- **ADD**:
  - Делает то же, что `COPY`, но имеет дополнительные возможности:
    - Автоматически распаковывает архивы (tar, gzip, bzip2) при копировании.
    - Может копировать файлы по URL (не рекомендуется из-за непредсказуемости).
  - Синтаксис: `ADD <источник> <назначение>`
  - Пример: `ADD app.tar.gz /app` (распакует архив в `/app`).

**Рекомендация**: Используйте `COPY`, если не нужна распаковка или загрузка по URL, так как `COPY` более явная и предсказуемая.

---

### 3. Чем директива ARG отличается от ENV?
- **ARG**:
  - Определяет переменную, доступную **только во время сборки** образа.
  - Задается в `Dockerfile` и передается через флаг `--build-arg` при выполнении `docker build`.
  - Не доступна внутри запущенного контейнера.
  - Пример:
    ```dockerfile
    ARG VERSION=1.0
    RUN echo "Building version $VERSION"
    ```
    Команда сборки: `docker build --build-arg VERSION=2.0 -t myimage .`

- **ENV**:
  - Задает переменную окружения, доступную **внутри контейнера** во время выполнения.
  - Переменные сохраняются в образе и доступны процессам в контейнере.
  - Пример:
    ```dockerfile
    ENV APP_PORT=8080
    CMD ["python", "app.py"]
    ```
    В контейнере переменная `APP_PORT` будет доступна.

**Ключевые различия**:
- `ARG` — для сборки, `ENV` — для выполнения контейнера.
- `ARG` требует передачи значения при сборке, `ENV` задает значение в образе.

---

### 4. Чем директива CMD отличается от ENTRYPOINT? Могут ли они использоваться вместе?
- **CMD**:
  - Указывает **команду по умолчанию**, которая выполняется при запуске контейнера.
  - Может быть переопределена при запуске контейнера через аргументы `docker run`.
  - Форматы:
    - `CMD ["executable", "param1", "param2"]` (exec, предпочтительно).
    - `CMD command param1` (shell, запускается в `/bin/sh -c`).
  - Пример: `CMD ["nginx", "-g", "daemon off;"]`

- **ENTRYPOINT**:
  - Задает **основную команду**, которая всегда выполняется при запуске контейнера.
  - Аргументы из `CMD` или `docker run` передаются в `ENTRYPOINT` как параметры.
  - Форматы аналогичны `CMD`.
  - Пример: `ENTRYPOINT ["python", "app.py"]`

- **Использование вместе**:
  - Да, `CMD` и `ENTRYPOINT` могут использоваться вместе.
  - `ENTRYPOINT` задает основную команду, а `CMD` — ее аргументы по умолчанию.
  - Пример:
    ```dockerfile
    ENTRYPOINT ["python", "app.py"]
    CMD ["--debug"]
    ```
    Запуск: `docker run myimage` → выполнит `python app.py --debug`.
    Переопределение: `docker run myimage --release` → выполнит `python app.py --release`.

- **Ключевое различие**:
  - `CMD` легко переопределяется аргументами в `docker run`.
  - `ENTRYPOINT` требует флага `--entrypoint` для переопределения.

---

### 5. Можно ли (и если да, то как) переопределить директивы CMD и ENTRYPOINT из Dockerfile при запуске контейнера?
Да, обе директивы можно переопределить:

- **Переопределение CMD**:
  - Просто укажите новую команду в `docker run` после имени образа.
  - Пример:
    ```dockerfile
    CMD ["nginx", "-g", "daemon off;"]
    ```
    Запуск с переопределением: `docker run myimage bash` → выполнит `bash` вместо `nginx`.

- **Переопределение ENTRYPOINT**:
  - Используйте флаг `--entrypoint` в `docker run`.
  - Пример:
    ```dockerfile
    ENTRYPOINT ["python", "app.py"]
    ```
    Запуск с переопределением: `docker run --entrypoint /bin/bash myimage` → выполнит `bash`.
  - Аргументы после имени образа передаются в новый `ENTRYPOINT`.

**Примечание**: Если используется `CMD` и `ENTRYPOINT` вместе, аргументы в `docker run` заменяют `CMD`, но не `ENTRYPOINT`, если не указан `--entrypoint`.

---

### 6. Как подключить volume к контейнеру? Какие типы volume’ов поддерживает Docker?
**Подключение volume**:
- Используйте флаг `-v` или `--mount` в команде `docker run`.
- Пример с `-v`:
  ```bash
  docker run -v /host/path:/container/path myimage
  ```
  - `/host/path`: Путь на хосте.
  - `/container/path`: Путь в контейнере.
- Пример с `--mount`:
  ```bash
  docker run --mount type=volume,source=myvolume,destination=/app myimage
  ```

**Типы volume’ов в Docker**:
1. **Host volumes (хостовые тома)**:
   - Данные хранятся на хост-системе.
   - Пример: `-v /host/data:/container/data`.
   - Плюс: Простота, прямой доступ. Минус: Зависимость от хоста.

2. **Named volumes (именованные тома)**:
   - Управляются Docker, хранятся в системной директории Docker (обычно `/var/lib/docker/volumes`).
   - Пример: `-v myvolume:/container/data`.
   - Плюс: Переносимость, управление через `docker volume`.

3. **Anonymous volumes (анонимные тома)**:
   - Создаются Docker автоматически, без явного имени.
   - Пример: `-v /container/data`.
   - Плюс: Удобно для временных данных. Минус: Сложнее управлять.

4. **Bind mounts (подключение каталогов)**:
   - Похожи на host volumes, но позволяют подключать конкретные файлы или директории.
   - Пример: `--mount type=bind,source=/host/file,destination=/container/file`.
   - Плюс: Гибкость. Минус: Зависимость от хоста.

5. **tmpfs mounts**:
   - Временное хранилище в памяти хоста (не записывается на диск).
   - Пример: `--mount type=tmpfs,destination=/tmp`.
   - Плюс: Высокая производительность. Минус: Данные не сохраняются.

**Пример в Docker Compose**:
```yaml
version: '3'
services:
  app:
    image: myimage
    volumes:
      - myvolume:/app
      - /host/data:/data
volumes:
  myvolume:
```

**Управление томами**:
- Создать: `docker volume create myvolume`
- Просмотреть: `docker volume ls`
- Удалить: `docker volume rm myvolume`

---
## Третий блок

### 1. Какие типы сокетов поддерживает Docker CLI?
Docker CLI взаимодействует с Docker Engine через **Docker API**, который может использовать различные типы сокетов для соединения. Основные типы сокетов, поддерживаемые Docker CLI:

- **Unix Socket**:
  - По умолчанию используется на Linux/macOS.
  - Файл сокета обычно находится по пути `/var/run/docker.sock`.
  - Пример: `DOCKER_HOST=unix:///var/run/docker.sock` (по умолчанию).
  - Используется для локального взаимодействия с Docker Daemon.

- **TCP Socket**:
  - Используется для удаленного подключения к Docker Daemon.
  - Требует настройки Docker Daemon для прослушивания TCP (по умолчанию не включено).
  - Пример: `DOCKER_HOST=tcp://127.0.0.1:2375` (или 2376 для TLS).
  - Поддерживает TLS для безопасного соединения.

- **SSH Socket**:
  - Позволяет подключаться к удаленному Docker Daemon через SSH.
  - Пример: `DOCKER_HOST=ssh://user@remote-host`.
  - Удобно для управления Docker на удаленных серверах.

- **Windows Named Pipe** (на Windows):
  - Используется в Windows-системах вместо Unix-сокета.
  - Пример: `DOCKER_HOST=npipe:////./pipe/docker_engine`.

**Настройка**:
- Указать сокет можно через переменную окружения `DOCKER_HOST` или в конфигурации Docker CLI.
- Для TLS требуется настройка сертификатов (`DOCKER_TLS_VERIFY`, `DOCKER_CERT_PATH`).

---

### 2. Какие Best Practices надо следовать при написании Dockerfile?
Следование лучшим практикам при написании `Dockerfile` помогает создавать эффективные, безопасные и компактные образы. Основные рекомендации:

1. **Используйте минималистичные базовые образы**:
   - Выбирайте легковесные образы, такие как `alpine` (например, `python:3.9-alpine` вместо `python:3.9`).
   - Пример: `FROM alpine:3.18`.

2. **Минимизируйте количество слоев**:
   - Объединяйте команды `RUN`, чтобы уменьшить количество слоев.
   - Пример: Вместо
     ```dockerfile
     RUN apt-get update
     RUN apt-get install -y curl
     ```
     Используйте:
     ```dockerfile
     RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
     ```

3. **Удаляйте ненужные файлы**:
   - Очищайте кэш и временные файлы в одном слое, чтобы уменьшить размер образа.
   - Пример: `RUN apt-get update && apt-get install -y package && rm -rf /var/lib/apt/lists/*`.

4. **Используйте .dockerignore**:
   - Создайте файл `.dockerignore`, чтобы исключить ненужные файлы (например, `.git`, `node_modules`) из контекста сборки.
   - Пример `.dockerignore`:
     ```
     .git
     *.md
     node_modules
     ```

5. **Предпочитайте COPY вместо ADD**:
   - Используйте `COPY`, если не нужна автоматическая распаковка или загрузка по URL.
   - Пример: `COPY ./app /app`.

6. **Используйте многоступенчатую сборку (multi-stage builds)**:
   - Для уменьшения размера итогового образа разделяйте сборку и выполнение.
   - Пример:
     ```dockerfile
     FROM node:18 AS builder
     WORKDIR /app
     COPY package*.json ./
     RUN npm install
     COPY . .
     RUN npm run build

     FROM node:18-slim
     WORKDIR /app
     COPY --from=builder /app/dist /app
     CMD ["node", "app.js"]
     ```

7. **Указывайте точные версии образов**:
   - Избегайте тега `latest`, чтобы обеспечить предсказуемость.
   - Пример: `FROM nginx:1.25.3` вместо `FROM nginx:latest`.

8. **Запускайте контейнеры от не-root пользователя**:
   - Создавайте пользователя и используйте `USER` для повышения безопасности.
   - Пример:
     ```dockerfile
     RUN useradd -m myuser
     USER myuser
     ```

9. **Оптимизируйте порядок инструкций**:
   - Помещайте часто изменяемые инструкции (например, `COPY` для исходного кода) в конец `Dockerfile`, чтобы использовать кэш для неизменяемых слоев.

10. **Используйте exec-формат для CMD и ENTRYPOINT**:
    - Предпочитайте `CMD ["executable", "param1"]` вместо `CMD executable param1`, чтобы избежать запуска через `/bin/sh` и корректно обрабатывать сигналы.

11. **Добавляйте метаданные**:
    - Используйте `LABEL` для документирования образа.
    - Пример: `LABEL maintainer="user@example.com" version="1.0"`.

12. **Проверяйте образы на уязвимости**:
    - Используйте инструменты вроде `docker scan` или Trivy для анализа безопасности.

---

### 3. Что делает команда `docker commit`?
Команда `docker commit` создает новый Docker-образ на основе текущего состояния контейнера. Она фиксирует изменения файловой системы и конфигурации контейнера в новый образ.

**Синтаксис**:
```bash
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]
```

**Пример**:
```bash
docker run -it --name my-container ubuntu bash
# Внутри контейнера: установка пакета
apt-get update && apt-get install -y curl
exit
docker commit my-container myimage:1.0
```

**Результат**: Создается образ `myimage:1.0`, содержащий все изменения, сделанные в контейнере (например, установленный `curl`).

**Особенности**:
- Используется для создания образов из измененных контейнеров, но не рекомендуется для регулярного использования (лучше использовать `Dockerfile` для воспроизводимости).
- Флаги:
  - `-a`: Указать автора (например, `-a "Author Name <email>"`).
  - `-m`: Добавить сообщение о коммите (например, `-m "Added curl"`).

**Предупреждение**: Образы, созданные через `docker commit`, могут быть сложными для поддержки, так как они не документируют процесс создания.

---

### 4. Что такое Copy on Write?
**Copy on Write (CoW)** — это механизм, используемый Docker для эффективного управления хранилищем данных в контейнерах. Он позволяет контейнерам и образам совместно использовать данные, минимизируя дублирование, и создавать копии только при изменении.

**Как работает CoW в Docker**:
- Образы состоят из неизменяемых слоев, которые совместно используются всеми контейнерами, созданными из этого образа.
- Когда контейнер изменяет файл (например, записывает данные), Docker создает копию изменяемого файла в **верхнем слое** контейнера (writable layer), не затрагивая исходный слой образа.
- Это обеспечивает:
  - Экономию дискового пространства (общие слои не дублируются).
  - Быстрое создание контейнеров, так как они используют существующие слои.
  - Изоляцию изменений между контейнерами.

**Пример**:
- Образ `nginx:latest` содержит файл `/etc/nginx/nginx.conf`.
- Контейнер изменяет этот файл. Docker создает копию `nginx.conf` в верхнем слое контейнера, оставляя исходный файл в образе нетронутым.
- Другие контейнеры, созданные из того же образа, продолжают использовать оригинальный файл.

**Связанные технологии**:
- Docker использует системы хранения, такие как `overlay2`, `aufs` или `btrfs`, для реализации CoW.
- Если контейнер удаляется, его верхний слой (изменения) удаляется, если не сохранен через `docker commit` или тома.

**Преимущества**:
- Экономия ресурсов.
- Быстрое создание и запуск контейнеров.
- Изоляция изменений.

**Недостатки**:
- Может увеличивать сложность управления хранилищем при большом количестве контейнеров.

---
