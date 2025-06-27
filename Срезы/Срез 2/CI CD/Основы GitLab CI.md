### Первый блок
1. Что такое GitLab Runner ? Какие виды раннеров бывают?  
**GitLab Runner** — это приложение, которое выполняет задания (jobs) из CI/CD-пайплайнов GitLab. Он работает как агент, который запускает код (например, сборку, тесты, деплой) на указанной машине (локальной, виртуальной, в Docker или Kubernetes).

GitLab Runner можно разделить по **типу исполнения** и **способу регистрации**:

**1. По типу исполнения (executor):**

- **Shell** – запускает команды напрямую в shell (Bash, PowerShell).
-  **Docker** – выполняет задания в Docker-контейнерах.
-  **Docker Machine** – автоматически создает виртуальные машины с Docker (для масштабирования).
-  **Kubernetes** – запускает задания в Kubernetes-кластере.
- **SSH** – выполняет команды на удаленной машине по SSH.
- **Parallels / VirtualBox** – запускает задачи в виртуальных машинах.

**2. По области видимости (scope):**

- **Shared Runners** – общие раннеры, доступные для всех проектов в GitLab.
- **Group Runners** – раннеры, доступные для всех проектов в группе.
- **Project Runners** – приватные раннеры, привязанные к конкретному проекту.

2. Какие есть триггеры запуска пайплайнов в GitLab CI?

Пайплайны могут запускаться автоматически или вручную. Основные триггеры:

1.    **Push / Merge в репозиторий** (по умолчанию):

o    git push в ветку (запускает пайплайн для этой ветки).

o    Создание/обновление Merge Request (MR).

2.    **Webhook-события**:

o    Внешние события (например, обновление репозитория, API-вызов).

3.    **Расписание (Scheduled Pipelines)**:

o    Запуск по расписанию (например, ночные тесты).

4.    **Ручной запуск (Manual)**:

o    Кнопка **"Run Pipeline"** в GitLab.

o    Запуск конкретного этапа (when: manual в .gitlab-ci.yml).

5.    **API-запрос**:

o    Запуск через GitLab API (POST /projects/:id/pipeline).

6.    **Изменение файлов (**changes**)**:

o    Пайплайн запускается только если изменились определенные файлы:

rules:

  - changes: [ "*.js" ]

7.    **Зависимости от других пайплайнов (Pipeline triggers)**:

o    Запуск пайплайна после завершения другого.

---
### Второй блок блок
1. Как подключить новый раннер?
1.**Установите GitLab Runner** на нужную машину:
Для Linux (пример для Debian/Ubuntu)
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
sudo apt-get install gitlab-runner
2.**Зарегистрируйте раннер**:
sudo gitlab-runner register
o    Укажите URL вашего GitLab instance (например, https://gitlab.com)
o    Введите токен регистрации (находится в Settings → CI/CD → Runners)
o    Выберите исполнитель (shell, docker, kubernetes и т.д.)
o    Добавьте теги (если нужно) и описание.
3.**Запустите раннер**:
sudo gitlab-runner start

2. Как сделать так, чтобы пайплайн запускался только при открытии MR в ветку master?
Используйте правило rules или only/except в .gitlab-ci.yml:
Вариант 1 (современный, с rules):
job_name:
  script: echo "This runs only on MR to master"
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event" && $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"'
Вариант 2 (с only):
job_name:
  script: echo "This runs only on MR to master"
  only:
    - merge_requests
  variables:
    - $CI_MERGE_REQUEST_TARGET_BRANCH_NAME == "master"

3. Как работать с секретами в GitLab CI?
**Основные методы:**

1.**CI/CD Variables** (рекомендуется)
- Добавьте переменные в Settings → CI/CD → Variables.
- Отметьте Mask variable, чтобы скрыть значение в логах.
- Используйте в .gitlab-ci.yml:
job_name:
  script:
    - echo $MY_SECRET
    
2.**Файлы с секретами** (например, для сертификатов)
- Используйте artifacts или cache для временных файлов.
- Пример с переменной:
job_name:
  script:
    - echo "$SSL_CERT" > cert.pem
    - use_cert.sh cert.pem

3.**HashiCorp Vault** (для Enterprise)
- Интеграция через CI_JOB_JWT:
job_name:
  script:
    - export SECRET=$(curl -H "X-Vault-Token: $VAULT_TOKEN" ...)

4.**Защищенные переменные** (для protected branches/tags)
- Включите Protected для переменной в настройках.

Важно:
-  Никогда не храните секреты прямо в репозитории!
-  Используйте Mask variable для строк и File для бинарных данных.

---
##### Дополнительно:
-   Для проверки MR используйте $CI_MERGE_REQUEST_* переменные.
-   Для Docker-раннеров настройте config.toml для безопасного хранения секретов.
---
### Третий блок
1. Как можно переиспользовать код пайплайнов в GitLab CI? 
Есть несколько способов избежать дублирования кода:
**🔹** **1.1. Использование** include
Позволяет подключать внешние YAML-файлы (локальные или удалённые).
include:
  - local: '/templates/.gitlab-ci-backend.yml'
  - remote: 'https://example.com/ci-templates/frontend.yml'
  - template: 'Auto-DevOps.gitlab-ci.yml'  # встроенные шаблоны GitLab

**🔹** **1.2. Якоря (**anchors**) и алиасы (**aliases**)**
Позволяют определять повторяющиеся блоки:
.job_template: **&job_config**
  script:
    - echo "Running script"
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

job1:
  <<: ***job_config**
  variables:
    ENV: "prod"

job2:
  <<: ***job_config**
  variables:
    ENV: "test"

**🔹** **1.3.** extends **для наследования**
Аналогично ООП, можно наследовать конфигурацию:
.tests:
  script: rake test
  rules:
    - if: $CI_PIPELINE_SOURCE == "push"

rspec:
  extends: .tests
  script: rake rspec

**🔹** **1.4. Использование** !reference
Позволяет ссылаться на части других джобов:
.setup:
  script:
    - echo "Setting up..."

.build:
  script:
    - !reference [.setup, script]
    - echo "Building..."

2. Как запустить стадии пайплайна параллельно? В каких ситуациях может потребоваться параллельный запуск стадий пайплайна?
Параллелизм ускоряет выполнение пайплайна, особенно при:
- **Тестировании** (разные виды тестов можно запускать одновременно).
-  **Сборке мультиплатформенных артефактов** (Docker-образы под разные ОС).
- **Развёртывании в несколько сред** (dev, staging, prod).

**🔹** **2.1. Параллельные джобы внутри стадии**
Если несколько джобов находятся в одной стадии, они выполняются **параллельно**:
stages:
  - test

test:unit:
  stage: test
  script: echo "Running unit tests"

test:integration:
  stage: test
  script: echo "Running integration tests"

**🔹** **2.2. Параллелизм через** parallel
Можно запускать **один джоб в нескольких параллельных экземплярах**:

test:
  stage: test
  script: ./run-tests.sh $CI_NODE_INDEX $CI_NODE_TOTAL
  parallel: 5  # запустит 5 копий джоба

**🔹** **2.3. Динамическое разбиение (**matrix**)**
Запуск джоба с разными переменными (GitLab 13.5+):

deploy:
  stage: deploy
  script: ./deploy.sh $ENV
  parallel:
    matrix:
      - ENV: ["dev", "staging"]
      - REGION: ["eu", "us"]