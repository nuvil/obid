### Первый блок
Что такое Jenkins-агент? Как добавить новый агент?
**Jenkins-агент** (или _агентный узел_) — это удалённая машина (физическая или виртуальная), которая выполняет задачи (джобы, пайплайны) от имени Jenkins-мастера. Агенты позволяют распределять нагрузку и запускать задачи в разных окружениях.

1.**Как добавить новый агент?**
1.    **Перейти в управление узлами**:  
Jenkins → Управление Jenkins → Управление узлами и облаками
2.    **Создать новый узел**:
o    Нажать **"Новый узел"**
o    Указать имя и выбрать **"Постоянный агент"**
3.    **Настроить агент**:
o    **Количество исполнителей** (executors) – сколько задач может выполняться параллельно.
o    **Корневая директория** – рабочая папка агента.
o    **Метод запуска**:
§  **Launch agent via Java Web Start** (JNLP) – агент подключается к мастеру.
§  **Launch agent via SSH** – мастер сам подключается к агенту.
§  **Команда из файла** (для Docker/Kubernetes).
o    Указать метки (labels), чтобы назначать задачи конкретным агентам.
4.    **Сохранить и запустить агент**.

2.**Почему не рекомендуется запускать пайплайны на мастере?**  
- **Безопасность**: Мастер хранит конфиденциальные данные (ключи, логины), выполнение задач на нём повышает риск утечки.
- **Производительность**: Мастер управляет всей инфраструктурой, и его перегрузка может привести к сбоям.
- **Стабильность**: Долгие или "тяжёлые" задачи могут замедлить работу Jenkins.
- **Изоляция**: Агенты позволяют запускать задачи в разных окружениях (ОС, версии ПО).
- **Рекомендация**: Мастер должен только координировать задачи, а выполняться они должны на агентах.

Что такое Job, Build и pipeline в Jenkins ? Какая между ними взаимосвязь? 
- **Job (Задача)** – единица работы в Jenkins (например, сборка, тестирование).
o    _Примеры_: Freestyle Job, Pipeline Job.
- **Build (Сборка)** – конкретный запуск джобы с результатами (успех/неудача, логи).
-  **Pipeline** – цепочка этапов (stages), объединяющая несколько джоб в единый процесс (например: сборка → тесты → деплой).
**Взаимосвязь**:
-  **Pipeline** состоит из **stages** (этапов), каждый из которых может запускать **jobs**.
- Каждый запуск пайплайна или джобы создаёт **build**.

Какие виды синтаксиса пайплайнов поддерживает Jenkins ? В чем отличия между ними? 
Jenkins поддерживает два основных синтаксиса:

1.**Declarative Pipeline** (Декларативный):
o    Проще, структурирован.
o    Использует предопределённые блоки (pipeline, stages, stage, steps).
o    Подходит для большинства сценариев.
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make'
            }
        }
    }
}

2.**Scripted Pipeline** (Скриптовый):
o    Гибче, но сложнее.
o    Основан на Groovy-скриптах.
o    Позволяет использовать циклы, условия, сложную логику.
node {
    stage('Build') {
        sh 'make'
    }
Отличия:

| **Declarative**     | **Scripted**                             |
| ------------------- | ---------------------------------------- |
| Жёсткая структура   | Полная свобода                           |
| Проще для новичков  | Требует знания Groovy                    |
| Встроенные проверки | Нужно самостоятельно обрабатывать ошибки |

Какие есть тригеры запуска пайплайнов в Jenkins?
Триггеры определяют, когда пайплайн должен запускаться автоматически:
1.**SCM-триггер (Poll SCM)** – проверка изменений в репозитории (Git, SVN).
triggers { pollSCM('* * * * *') }  // Каждую минуту
2.**Webhook (GitHub/GitLab Hook)** – запуск при пуше в репозиторий.
3.**Таймер (Cron)** – запуск по расписанию.
triggers { cron('0 12 * * *') }  // В 12:00 ежедневно
4.**Ручной запуск** – через UI или API.
5.**Запуск после другого джоба** (например, через build job: 'test').
6.**Триггер по событию** (например, завершение сборки артефакта в Nexus).

### Второй блок  
Как запускать пайплайн по вебхуку?  
Как запускать отдельные стадии пайплайна параллельно друг с другом?




1.Как работать с секретами в Jenkins?
Для работы с секретами в Jenkins можно использовать:
-  **Credentials Plugin** – хранит пароли, токены, SSH-ключи в зашифрованном виде.
- **Secret Text / Username & Password / SSH Key** – типы секретов.
-  **Использование в пайплайне**:
withCredentials([usernamePassword(credentialsId: 'my-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
    sh 'echo $USER: $PASS'
}
- **HashiCorp Vault** – для интеграции с внешним хранилищем секретов.

2.Как запустить определенную стадию пайплайна в Docker-контейнере?
Используйте директиву agent внутри стадии:
pipeline {
    agent none
    stages {
        stage('Build') {
            agent { docker 'maven:3.8.6' }
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
Или с дополнительными параметрами:
agent {
    docker {
        image 'node:18'
        args '-v /tmp:/tmp'
    }
}
3.Как гарантировать, что шаги выполнятся в любом случае?

Используйте блок post с условиями:
post {
    always {
        echo "Этот шаг выполнится всегда"
        cleanWs() // Очистка workspace
    }
    success {
        echo "Только при успехе"
    }
    failure {
        echo "Только при ошибке"
    }
}

4.Что означает статус UNSTABLE?
-  **UNSTABLE** – билд завершился, но с проблемами (например, неудачные тесты).
-  Устанавливается через currentBuild.result = 'UNSTABLE' или плагинами (например, JUnit при проваленных тестах).

5.Как запускать пайплайн по вебхуку?

1.    Настройте **GitHub/GitLab/Bitbucket Webhook** на URL Jenkins:
http://<JENKINS_URL>/github-webhook/
2.    В Jenkins:
o    Установите плагин **GitHub Plugin**.
o    В настройках пайплайна выберите **"GitHub hook trigger for GITScm polling"**.
3.    Альтернативно – используйте Generic Webhook Trigger Plugin.

6.Как запускать стадии параллельно?
Используйте директиву parallel:
stages {
    stage('Parallel Stage') {
        parallel {
            stage('Unit Tests') {
                steps { sh './run-unit-tests.sh' }
            }
            stage('Integration Tests') {
                steps { sh './run-integration-tests.sh' }
            }
        }
    }
}

Или для динамического параллелизма:
def stages = ['Test', 'Lint', 'Build'].collect { name ->
    stage(name) {
        steps { sh "./run-${name.toLowerCase()}.sh" }
    }
}
parallel stages
### Третий блок 
**1. Запуск Jenkins Pipeline из консоли**
**Способ 1: Через Jenkins CLI (Command Line Interface)**
Jenkins предоставляет CLI-утилиту (jenkins-cli.jar), которая позволяет управлять заданиями.
**Шаги:**
1.    **Скачайте** jenkins-cli.jar
o    Обычно доступен по адресу: http://<JENKINS_URL>/cli/
2.    **Запустите Pipeline**
```
java -jar jenkins-cli.jar -s http://<JENKINS_URL>/ -auth <USER>:<API_TOKEN> build <JOB_NAME> -p PARAM1=value1 -p PARAM2=value2
```

o    <JENKINS_URL> — адрес Jenkins (например, http://localhost:8080).
```
o    <USER>:<API_TOKEN> — логин и API-токен (можно получить в Manage Jenkins → Security → Manage Users → API Token).
```
o    <JOB_NAME> — название Pipeline-задания.
o    -p — передача параметров (если Pipeline parameterized).

---

**Способ 2: Через REST API (cURL)**

Jenkins предоставляет REST API для запуска заданий.
**Шаги:**
1.**Получите** crumb **(CSRF-токен, если включена защита)**
```
CRUMB=$(curl -s "http://<USER>:<API_TOKEN>@<JENKINS_URL>/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,\":\",//crumb)")
```
2.  **Запустите Pipeline**
```
curl -X POST -H "$CRUMB" "http://<USER>:<API_TOKEN>@<JENKINS_URL>/job/<JOB_NAME>/build" --data-urlencode json='{"parameter": [{"name":"PARAM1", "value":"value1"}, {"name":"PARAM2", "value":"value2"}]}'
```

o    Если Pipeline не parameterized, можно просто:
```
curl -X POST "http://<USER>:<API_TOKEN>@<JENKINS_URL>/job/<JOB_NAME>/build"
```

**2. Запуск Jenkins Pipeline из другого Pipeline**

Для запуска одного Pipeline из другого можно использовать:
-  build (шаг из jenkins-build-step плагина)
-  parallel (если нужно запустить несколько Pipeline параллельно)
-  **REST API** (через httpRequest или curl)

**Способ 1: Через** build **(рекомендуется)**
pipeline {
    agent any
    stages {
        stage('Run Another Pipeline') {
            steps {
                // Запуск Pipeline с параметрами
                build job: 'Other-Pipeline-Name',
                      parameters: [
                          string(name: 'PARAM1', value: 'value1'),
                          string(name: 'PARAM2', value: 'value2')
                      ],
                      wait: false // если не нужно ждать завершения
            }
        }
    }
}
-   wait: false — запуск асинхронно (не блокирует текущий Pipeline).
-   Можно передавать booleanParam, choice, text и другие типы параметров.

---

**Способ 2: Через** parallel **(параллельный запуск)**
pipeline {
    agent any
    stages {
        stage('Run Pipelines in Parallel') {
            steps {
                script {
                    parallel(
                        "Pipeline1": { build job: 'Pipeline1' },
                        "Pipeline2": { build job: 'Pipeline2', parameters: [string(name: 'PARAM', value: 'test')] }
                    )
                }
            }
        }
    }
}

---

**Способ 3: Через REST API (из Pipeline)**
Если нужно больше контроля, можно использовать httpRequest (требуется установленный плагин HTTP Request):
```
pipeline {
    agent any
    stages {
        stage('Trigger via API') {
            steps {
                script {
                    def response = httpRequest(
                        url: "http://<USER>:<API_TOKEN>@<JENKINS_URL>/job/<JOB_NAME>/build",
                        httpMode: 'POST',
                        contentType: 'APPLICATION_JSON',
                        validResponseCodes: '200,201'
                    )
                    echo "Triggered build: ${response.content}"
                }
            }
        }
    }
}
```

---

| Способ                | Когда использовать                      |
| --------------------- | --------------------------------------- |
| Jenkins CLI           | Для скриптов вне Jenkins (bash, Python) |
| REST API (cURL)       | Для интеграции с внешними системами     |
| build в Pipeline      | Лучший вариант для Jenkins-to-Jenkins   |
| parallel + build      | Параллельный запуск                     |
| httpRequest в Pipline | Если нужны тонкие настройки API         |
