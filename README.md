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


### Подключение shadcn
shadCN предоставляет готовые компоненты, которые можно использовать сразу в интерфейсе, например кнопки, карточки и формы.

[подключение shadcn](https://ui.shadcn.com/docs/installation/vite)

Пример использования кнопки:

```bash
npx shadcn@latest add button
```

```jsx
import { ArrowUpIcon } from "lucide-react"
import { Button } from "@/components/ui/button"
export function ButtonDemo() {
  return (
    <div className="flex flex-wrap items-center gap-2 md:flex-row">
      <Button variant="outline">Button</Button>
      <Button variant="outline" size="icon" aria-label="Submit">
        <ArrowUpIcon />
      </Button>
    </div>
  )
}
```

# Реализация TodoList

## Структура проекта

Создадим три основных компонента:

- `App.tsx` — основной компонент приложения, управляет состоянием задач.
- `AddTodo.tsx` — форма для добавления новой задачи.
- `TodoList.tsx` — список задач с возможностью редактирования и удаления.

Также создадим файл `api.ts` для работы с API.

### api.ts — работа с бэкендом
```ts
import { Todo } from "./types"

const BASE_URL = "https://your-backend-url" // замените на адрес вашего бэкенда

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
import { Input, Button } from "@shadcn/ui"
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
    <form onSubmit={handleSubmit} className="flex gap-2 mt-4">
      <Input
        placeholder="Введите задачу..."
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        className="flex-1"
      />
      <Button type="submit">Добавить</Button>
    </form>
  )
}
```

### Компонент TodoList.tsx

```ts
import { Button, Checkbox } from "@shadcn/ui"
import { Todo } from "../types"

interface TodoListProps {
  todos: Todo[]
  onDelete: (id: number) => void
  onToggle: (todo: Todo) => void
}

export default function TodoList({ todos, onDelete, onToggle }: TodoListProps) {
  if (todos.length === 0) {
    return <p className="text-gray-500 mt-4">Задач пока нет</p>
  }

  return (
    <ul className="mt-4 space-y-2">
      {todos.map((todo) => (
        <li
          key={todo.id}
          className="flex items-center justify-between bg-white rounded-lg p-3 shadow"
        >
          <div className="flex items-center gap-2">
            <Checkbox
              checked={todo.completed}
              onCheckedChange={() => onToggle(todo)}
            />
            <span
              className={`${
                todo.completed ? "line-through text-gray-400" : ""
              }`}
            >
              {todo.title}
            </span>
          </div>
          <Button variant="destructive" size="sm" onClick={() => onDelete(todo.id)}>
            Удалить
          </Button>
        </li>
      ))}
    </ul>
  )
}
```

### Главный компонент App.tsx

```ts
import { useEffect, useState } from "react"
import AddTodo from "./components/AddTodo"
import TodoList from "./components/TodoList"
import { getTodos, createTodo, deleteTodo, updateTodo } from "./api"
import { Todo } from "./types"
import { Card, CardHeader, CardContent } from "@shadcn/ui"

function App() {
  const [todos, setTodos] = useState<Todo[]>([])

  useEffect(() => {
    loadTodos()
  }, [])

  const loadTodos = async () => {
    const data = await getTodos()
    setTodos(data)
  }

  const handleAdd = async (todo: Todo) => {
    const newTodo = await createTodo(todo)
    setTodos((prev) => [...prev, newTodo])
  }

  const handleDelete = async (id: number) => {
    await deleteTodo(id)
    setTodos((prev) => prev.filter((t) => t.id !== id))
  }

  const handleToggle = async (todo: Todo) => {
    const updated = { ...todo, completed: !todo.completed }
    await updateTodo(updated)
    setTodos((prev) =>
      prev.map((t) => (t.id === todo.id ? updated : t))
    )
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100 p-6">
      <Card className="w-full max-w-md p-4">
        <CardHeader>
          <h1 className="text-2xl font-semibold text-center">Todo List</h1>
        </CardHeader>
        <CardContent>
          <AddTodo onAdd={handleAdd} />
          <TodoList todos={todos} onDelete={handleDelete} onToggle={handleToggle} />
        </CardContent>
      </Card>
    </div>
  )
}

export default App

```

