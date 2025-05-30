# Hasura GraphQL API для управления задачами

Этот проект представляет собой backend-приложение на базе Hasura, которое реализует GraphQL API для работы с базой данных, содержащей таблицы задач, пользователей и меток.

## Структура базы данных

Система включает в себя три основные таблицы:
- **users** (пользователи): id, firstName (имя), lastName (фамилия), bio (биография)
- **labels** (метки): id, caption (название), color (цвет)
- **tasks** (задачи): id, title (заголовок), description (описание), assignee (исполнитель из таблицы `users`), связь с метками (many-to-many с таблицей `labels`)

## Требования

Для запуска проекта необходимо установить:
- Docker
- Docker Compose

## Запуск проекта

### Автоматический запуск (Windows)

В системе Windows вы можете использовать скрипт `start.bat` для быстрого запуска:

```
start.bat
```

### Ручной запуск

1. Клонируйте репозиторий:
```
git clone <url_репозитория>
cd <имя_директории>
```

2. Запустите проект с помощью Docker Compose:
```
docker-compose up -d
```

3. Откройте консоль Hasura по адресу:
```
http://localhost:8080/console
```

### Примечание об отслеживании таблиц и отношений

При первом запуске система автоматически настраивает отслеживание таблиц и отношений через скрипт инициализации. Если вы видите запрос на отслеживание таблиц или отношений в консоли Hasura, выполните следующие действия:

1. Перейдите на вкладку "Data" в консоли Hasura
2. Для каждой таблицы (users, labels, tasks, task_labels) нажмите кнопку "Track" в разделе "Untracked tables or views"
3. После отслеживания всех таблиц, перейдите на вкладку "Data → users" и нажмите "Track" для отношения с tasks
4. Повторите аналогичные действия для остальных таблиц:
   - В таблице tasks отследите связь с assignee и task_labels
   - В таблице labels отследите связь с task_labels
   - В таблице task_labels отследите связи с task и label

Это необходимо сделать только при первом запуске, в случае если автоматическая настройка не сработала.

## Примеры GraphQL запросов

После запуска вы можете использовать GraphQL API через консоль Hasura или инструменты типа Postman, отправляя запросы на эндпоинт:
```
http://localhost:8080/v1/graphql
```

### Получение всех пользователей

```graphql
query GetAllUsers {
  users {
    id
    first_name
    last_name
    bio
  }
}
```

### Получение всех задач с информацией об исполнителе и метках

```graphql
query GetAllTasks {
  tasks {
    id
    title
    description
    assignee {
      id
      first_name
      last_name
    }
    task_labels {
      label {
        id
        caption
        color
      }
    }
  }
}
```

### Создание нового пользователя

```graphql
mutation CreateUser {
  insert_users_one(object: {
    first_name: "Александр",
    last_name: "Смирнов",
    bio: "Инженер по тестированию"
  }) {
    id
    first_name
    last_name
  }
}
```

### Создание новой метки

```graphql
mutation CreateLabel {
  insert_labels_one(object: {
    caption: "Критично",
    color: "#FF0000"
  }) {
    id
    caption
  }
}
```

### Создание новой задачи с метками

```graphql
mutation CreateTask {
  insert_tasks_one(object: {
    title: "Разработать новую функцию",
    description: "Необходимо добавить функцию экспорта данных",
    assignee_id: 1,
    task_labels: {
      data: [
        { label_id: 1 },
        { label_id: 3 }
      ]
    }
  }) {
    id
    title
  }
}
```

### Обновление задачи

```graphql
mutation UpdateTask {
  update_tasks_by_pk(
    pk_columns: { id: 1 },
    _set: {
      title: "Исправить критическую ошибку",
      description: "Срочно исправить баг в модуле авторизации",
      assignee_id: 2
    }
  ) {
    id
    title
    assignee {
      first_name
      last_name
    }
  }
}
```

### Удаление метки у задачи

```graphql
mutation RemoveTaskLabel {
  delete_task_labels(where: {
    task_id: { _eq: 1 },
    label_id: { _eq: 2 }
  }) {
    affected_rows
  }
}
```

### Добавление метки к задаче

```graphql
mutation AddTaskLabel {
  insert_task_labels_one(object: {
    task_id: 1,
    label_id: 4
  }) {
    task {
      title
    }
    label {
      caption
    }
  }
}
```

### Удаление задачи

```graphql
mutation DeleteTask {
  delete_tasks_by_pk(id: 6) {
    id
    title
  }
}
```

### Фильтрация задач по меткам

```graphql
query GetTasksByLabel {
  tasks(where: {
    task_labels: {
      label_id: { _eq: 1 }
    }
  }) {
    id
    title
    description
    assignee {
      first_name
      last_name
    }
  }
}
```

### Поиск задач по названию

```graphql
query SearchTasks {
  tasks(where: {
    title: { _ilike: "%ошибк%" }
  }) {
    id
    title
    description
  }
}
```

## Как остановить проект

Для остановки работы проекта выполните команду:
```
docker-compose down
```

Для удаления всех данных и полного сброса проекта выполните:
```
docker-compose down -v
``` 