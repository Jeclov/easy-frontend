# Фронтенд и взаимодействие с API

## Что такое фронтенд

Фронтенд — это та часть приложения, которую видит пользователь.  
Он отвечает за интерфейс, визуализацию и взаимодействие.

**Основные технологии:**
- **HTML** — структура страницы (заголовки, списки, формы).  
- **CSS** — стилизация (цвета, шрифты, расположение).  
- **JavaScript** — интерактивность (клики, динамическое обновление данных).

---

## Зачем нужны фреймворки и библиотеки

Фреймворки и библиотеки помогают:
- Создавать интерфейсы быстрее и проще.  
- Использовать компоненты для повторного использования кода.  
- Управлять состоянием приложения и обновлениями UI.  

**Примеры:**
- **React** — библиотека для создания интерактивного UI через компоненты.  
- **Vue / Angular** — альтернативные фреймворки для фронтенда.  

**Особенности React:**
- Декларативный подход: описываем, как UI должен выглядеть, а React сам обновляет DOM.  
- Компоненты: строим интерфейс из небольших, переиспользуемых блоков.  
- Управление состоянием: `useState`, `useEffect` и другие хуки помогают работать с данными.

---

## Взаимодействие с бэкендом

Фронтенд часто должен получать и отправлять данные на сервер.  
Для этого используются **API** (Application Programming Interface).

**HTTP-методы:**
- **GET** — получить данные с сервера.  
- **POST** — создать новую запись.  
- **PUT** — обновить существующую запись.  
- **DELETE** — удалить запись.

**Пример задачи: TodoList**
- Получаем список задач → GET `/todos`  
- Создаем новую задачу → POST `/todos`  
- Обновляем задачу → PUT `/todos/{id}`  
- Удаляем задачу → DELETE `/todos/{id}`

---

## OpenAPI

OpenAPI — это спецификация, которая описывает API:
- Какие эндпоинты есть.
- Какие данные нужно отправлять и что вернется в ответ.
- Помогает фронтенду правильно взаимодействовать с бэкендом.


# Создание проекта

В этом разделе мы создадим рабочий проект React с использованием Vite и подключим библиотеку компонентов shadCN.

## Создание проекта с Vite

Откройте терминал и выполните команду для создания нового проекта:

```bash
npm create vite@latest
```

Перейдите в созданную папку проекта:

```bash
cd todo-frontend
```

Установите все зависимости проекта:

```bash
npm install
```

Установите библиотеку shadCN для готовых UI-компонентов:

```bash
npm install @shadcn/ui
```

Для работы с API можно использовать стандартный fetch или установить axios:

```bash
npm install axios
```

Чтобы запустить проект в режиме разработки, выполните команду:

```bash
npm run dev
```

Откройте браузер и перейдите по адресу, который покажет терминал (обычно http://localhost:5173). Вы увидите базовую страницу React — это наш стартовый проект.


### Подключение Ant Design
antd предоставляет готовые компоненты, которые можно использовать сразу в интерфейсе, например кнопки, карточки и формы.
[Документация Ant Design](https://ant.design/docs/react/introduce)

# Реализация TodoList

## Структура проекта

Создадим три основных компонента:

- `App.tsx` — основной компонент приложения, управляет состоянием задач.
- `AddTodo.tsx` — форма для добавления новой задачи.
- `TodoList.tsx` — список задач с возможностью редактирования и удаления.

Также создадим файл `types.ts` для описания типов данных.

```ts
export interface Todo {
  id: number
  title: string
  description?: string
  completed: boolean
}
```

Также создадим файл `api.ts` для работы с API.

### api.ts — работа с бэкендом
```ts
import { Todo } from "./types"

const BASE_URL = "https://your-backend-url"

export const getTodos = async (): Promise<Todo[]> => {
  const res = await fetch(`${BASE_URL}/todos/`)
  return res.json()
}

export const createTodo = async (todo: Todo): Promise<Todo> => {
  const res = await fetch(`${BASE_URL}/todos/`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(todo),
  })
  return res.json()
}

export const updateTodo = async (todo: Todo): Promise<Todo> => {
  const res = await fetch(`${BASE_URL}/todos/${todo.id}`, {
    method: "PUT",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(todo),
  })
  return res.json()
}

export const deleteTodo = async (id: number): Promise<void> => {
  await fetch(`${BASE_URL}/todos/${id}`, { method: "DELETE" })
}

```

### Компонент AddTodo.tsx
```ts
import { useState, FormEvent } from "react"
import { Input, Button, Form } from "antd"
import { Todo } from "../types"

interface AddTodoProps {
  onAdd: (todo: Todo) => void
}

export default function AddTodo({ onAdd }: AddTodoProps) {
  const [title, setTitle] = useState("")

  const handleSubmit = (e: FormEvent) => {
    e.preventDefault()
    if (!title.trim()) return
    const newTodo: Todo = {
      id: Date.now(),
      title,
      completed: false,
    }
    onAdd(newTodo)
    setTitle("")
  }

  return (
    <Form layout="inline" onSubmitCapture={handleSubmit} style={{ marginTop: 16 }}>
      <Form.Item style={{ flex: 1 }}>
        <Input
          placeholder="Введите задачу"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
        />
      </Form.Item>
      <Form.Item>
        <Button type="primary" htmlType="submit">
          Добавить
        </Button>
      </Form.Item>
    </Form>
  )
}
```

### Компонент TodoList.tsx

```ts
import { List, Button, Checkbox, Typography, Space } from "antd"
import { DeleteOutlined } from "@ant-design/icons"
import { Todo } from "../types"

interface TodoListProps {
  todos: Todo[]
  onDelete: (id: number) => void
  onToggle: (todo: Todo) => void
}

export default function TodoList({ todos, onDelete, onToggle }: TodoListProps) {
  return (
    <List
      style={{ marginTop: 16 }}
      dataSource={todos}
      locale={{ emptyText: "Задач пока нет" }}
      renderItem={(todo) => (
        <List.Item
          actions={[
            <Button
              type="text"
              danger
              icon={<DeleteOutlined />}
              onClick={() => onDelete(todo.id)}
            />,
          ]}
        >
          <Space>
            <Checkbox
              checked={todo.completed}
              onChange={() => onToggle(todo)}
            />
            <Typography.Text
              delete={todo.completed}
              style={{ opacity: todo.completed ? 0.5 : 1 }}
            >
              {todo.title}
            </Typography.Text>
          </Space>
        </List.Item>
      )}
    />
  )
}
```

### Главный компонент App.tsx

```ts
import { useEffect, useState } from "react"
import { Card, Typography, message } from "antd"
import AddTodo from "./components/AddTodo"
import TodoList from "./components/TodoList"
import { getTodos, createTodo, deleteTodo, updateTodo } from "./api"
import { Todo } from "./types"

const { Title } = Typography

function App() {
  const [todos, setTodos] = useState<Todo[]>([])
  const [loading, setLoading] = useState(false)

  useEffect(() => {
    loadTodos()
  }, [])

  const loadTodos = async () => {
    setLoading(true)
    try {
      const data = await getTodos()
      setTodos(data)
    } catch (error) {
      message.error("Ошибка при загрузке задач")
    } finally {
      setLoading(false)
    }
  }

  const handleAdd = async (todo: Todo) => {
    try {
      const newTodo = await createTodo(todo)
      setTodos((prev) => [...prev, newTodo])
      message.success("Задача добавлена")
    } catch {
      message.error("Не удалось добавить задачу")
    }
  }

  const handleDelete = async (id: number) => {
    try {
      await deleteTodo(id)
      setTodos((prev) => prev.filter((t) => t.id !== id))
      message.success("Задача удалена")
    } catch {
      message.error("Ошибка при удалении")
    }
  }

  const handleToggle = async (todo: Todo) => {
    const updated = { ...todo, completed: !todo.completed }
    try {
      await updateTodo(updated)
      setTodos((prev) => prev.map((t) => (t.id === todo.id ? updated : t)))
    } catch {
      message.error("Ошибка при обновлении")
    }
  }

  return (
    <div style={{ minHeight: "100vh", display: "flex", justifyContent: "center", alignItems: "center", background: "#f5f5f5", padding: 24 }}>
      <Card style={{ width: 400 }}>
        <Title level={3} style={{ textAlign: "center" }}>
          Todo List
        </Title>
        <AddTodo onAdd={handleAdd} />
        <TodoList todos={todos} onDelete={handleDelete} onToggle={handleToggle} />
      </Card>
    </div>
  )
}

export default App
```

