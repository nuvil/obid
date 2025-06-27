## Первый блок

**Что такое Helm?**
**Helm** — это менеджер пакетов (package manager) для Kubernetes, который упрощает развертывание, управление и обновление приложений в кластере. Он работает как аналог apt/yum (для Linux) или npm/pip (для разработчиков), но специализируется на Kubernetes-приложениях.

**Какую проблему решает Helm?**
Без Helm развертывание приложений в Kubernetes требует ручного создания множества YAML-файлов (Deployments, Services, ConfigMaps, Secrets и т. д.), что приводит к:
1.    **Сложности управления** — десятки файлов для одного приложения.
2.    **Повторяемости** — трудно тиражировать конфигурации между средами (dev/stage/prod).
3.    **Отсутствию версионирования** — сложно отслеживать изменения конфигураций.
4.    **Зависимостям** — приложения могут зависеть от других компонентов (например, базы данных или очереди сообщений).

**Helm решает эти проблемы**, предлагая:
    Шаблонизацию конфигов (подстановка переменных).
    Управление зависимостями (как в requirements.txt Python).
    Версионирование релизов (возможность отката).
    Репозитории готовых чартов (как Helm Hub или Artifact Hub).

**Что такое Helm-чарт?**
**Helm-чарт (Helm Chart)** — это упакованный набор файлов, описывающих ресурсы Kubernetes для развертывания приложения. Это аналог "пакета" в других менеджерах.

> [!NOTE]
> Структура чарта:
> 
> my-app-chart/ 
> ├── Chart.yaml          # Метаданные (название, версия, зависимости) 
> ├── values.yaml         # Дефолтные параметры конфигурации 
> ├── templates/          # Шаблоны Kubernetes-манифестов 
> │   ├── deployment.yaml 
> │   ├── service.yaml 
> │   └── ... 
> └── charts/             # Подчиненные чарты (зависимости) 
> 
> **Пример использования:**
> 
> 1.    Установка чарта из репозитория:
> 
> helm install my-release bitnami/nginx
> 
> 2.    Свой чарт можно создать командой:
> 
> helm create my-chart

---

> [!NOTE]
> **Ключевые термины**
> 
> ·         **Release** — экземпляр развернутого чарта в кластере (например, my-app-dev).
> 
> ·         **Repository** — хранилище чартов (как Docker Hub, но для Helm).
> 
> ·         **Templating** — использование Go-шаблонов для генерации YAML-файлов.
> 
> Helm особенно полезен для CI/CD, многокомпонентных приложений (например, микросервисов) и работы с разными окружениями.
## Второй блок

1. **Что такое values-файл в Helm?**
**Values-файл** в Helm — это YAML-файл (обычно values.yaml), который содержит конфигурационные параметры для чарта. Он позволяет:
- Определять настраиваемые переменные (например, количество реплик, образы контейнеров, ресурсы).
- Переопределять значения по умолчанию из шаблонов чарта (templates/).
- Хранить environment-specific настройки (dev/stage/prod).

Пример values.yaml:

replicaCount: 1
image:
  repository: nginx
  tag: "latest"

2. **Как использовать переменные в Helm-чартах?**
Переменные из values.yaml используются в шаблонах (templates/*.yaml) через синтаксис {{ .Values.<параметр> }}.

Пример (в deployment.yaml):

apiVersion: apps/v1
kind: Deployment
spec:
  replicas: {{ .Values.replicaCount }}
  containers:
    - image: {{ .Values.image.repository }}:{{ .Values.image.tag }}

3. **Как использовать несколько values-файлов при деплое?**
Helm позволяет передавать несколько values-файлов в CLI. **Приоритет** определяется порядком: последний файл переопределяет предыдущие.
helm install myapp . -f values.yaml -f override-values.yaml
или:
helm upgrade myapp . --values=values.yaml --values=prod-values.yaml

**Пример**:
- values.yaml (базовые настройки):
replicaCount: 1
- prod-values.yaml (переопределение для prod):
replicaCount: 3

4. **Как переопределить переменную из values-файла при запуске деплоя?**

Через флаг --set можно переопределить отдельные переменные прямо в командной строке.

helm install myapp . --set replicaCount=5

или для вложенных параметров:

helm upgrade myapp . --set image.tag="v1.2.0"

**Приоритеты** (от высшего к низшему):

1.    --set (в командной строке).

2.    -f/--values (указанные файлы, последний файл важнее).

3.    values.yaml в чарте.

4.    Значения по умолчанию в Chart.yaml (если есть).

---

Примеры комбинированного использования

```
# Базовый values.yaml + переопределения из prod-values.yaml + ручная установка replicaCount

helm install myapp . -f values.yaml -f prod-values.yaml --set replicaCount=10
```

**Важно**:

- Используйте helm template . --debug для проверки итоговых значений перед деплоем.

- Для сложных структур в --set используйте точки:

helm upgrade --set "database.enabled=true"

## Третий блок

Как посмотреть сгенерированные манифесты перед запуском деплоя с помощью Helm?

Чтобы увидеть сгенерированные манифесты **перед** фактическим деплоем, можно использовать команду:

```
helm template <RELEASE_NAME> <CHART> [FLAGS]
```

Примеры:

1.    **Базовый рендеринг манифестов**:

helm template my-release ./my-chart

или для чарта из репозитория:

helm template my-release bitnami/nginx

2.    **С указанием namespace**:

helm template my-release ./my-chart --namespace=my-namespace

3.    **С подстановкой values-файла**:

helm template my-release ./my-chart -f values.yaml

4.    **С выводом в файл**:

helm template my-release ./my-chart > manifests.yaml

Альтернатива (если релиз уже установлен):

Если релиз уже установлен, можно посмотреть манифесты с помощью:

helm get manifest <RELEASE_NAME>

---

Как откатиться на предыдущий релиз с помощью Helm?

Helm хранит историю релизов, и откат выполняется командой:
```
helm rollback <RELEASE_NAME> <REVISION>
```
Примеры:
1.    **Откат на предыдущую версию** (например, на ревизию 1):
helm rollback my-release 1
2.    **Просмотр истории релиза** (чтобы узнать номер ревизии):
helm history my-release

Вывод будет выглядеть примерно так:
REVISION  STATUS      DESCRIPTION
1         superseded  Install complete
2         deployed    Upgrade complete

Здесь можно выбрать нужную ревизию для отката.
3.    **Автоматический откат при неудачном деплое** (если использовался --atomic):
helm upgrade --install my-release ./my-chart --atomic
Если деплой провалится, Helm автоматически откатится на предыдущую версию.

Дополнительные флаги:
·         --wait – ждать завершения отката;
·         --cleanup-on-fail – удалять ресурсы при неудачном деплое.

Итог:
- **Для просмотра манифестов** → helm template (перед деплоем) или helm get manifest (после деплоя).
```
Для отката → helm rollback <RELEASE_NAME> <REVISION>, предварительно проверив историю через helm history.
```