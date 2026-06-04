# Módulo 3 — React Router

> 🎯 Objetivo: Criar navegação entre páginas sem recarregar o browser.

---

## 1. O que é SPA?

**Single Page Application (SPA)** é uma aplicação que carrega **uma única página HTML** e troca o conteúdo dinamicamente, sem fazer um novo request ao servidor a cada navegação.

### Comparação: navegação tradicional vs SPA

**Navegação tradicional (Multi Page Application):**

```
Usuário clica em "Dashboard"
    ↓
Browser faz GET /dashboard
    ↓
Servidor retorna um HTML novo
    ↓
Página recarrega completamente ← problema: lento, perde estado
```

**SPA com React Router:**

```
Usuário clica em "Dashboard"
    ↓
React Router intercepta o clique
    ↓
Troca apenas o componente renderizado
    ↓
URL atualizada, sem reload ← rápido, estado preservado
```

> O React Router é quem faz esse trabalho: ele observa a URL e decide qual componente renderizar, sem sair da página.

> 📖 Referência: [React Router — Main Concepts](https://reactrouter.com/start/library/routing)

---

## 2. Configurando rotas com `createBrowserRouter`

O `createBrowserRouter` é a forma moderna e recomendada de definir rotas no React Router v6+. Você declara todas as rotas em um único objeto, separado da UI.

**`src/core/routes/index.tsx`**

```tsx
import { createBrowserRouter } from "react-router-dom";

export const router = createBrowserRouter([
  {
    path: "/",
    element: (
      <h1 className="text-2xl font-bold">Rota para a página de Tasks!</h1>
    ),
  },
  {
    path: "/login",
    element: <h1 className="text-2xl font-bold">Rota para o login!</h1>,
  },
]);
```

> 📖 Referência: [React Router — createBrowserRouter](https://reactrouter.com/6.30.1/routers/create-browser-router)

---

Conecte o router ao `main.tsx`:

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { RouterProvider } from "react-router-dom";

import "./index.css";

import { AppProviders } from "./core/providers/AppProviders";
import { router } from "./core/routes";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <AppProviders>
      <RouterProvider router={router} />
    </AppProviders>
  </StrictMode>,
);
```

> ⚠️ O `!` após `getElementById("root")` é uma **non-null assertion** do TypeScript — informa ao compilador que esse elemento existe no `index.html` e nunca será `null`. Quando usa `createBrowserRouter` com `RouterProvider`, o `<BrowserRouter>` não é mais necessário.

---

## 3. Estrutura de rotas da aplicação

```
URL              Layout Utilizado       Componente Filho (Outlet)
───────────────────────────────────────────────────────────────────
/login          →  AuthLayout         →  LoginPage (Formulário)
/               →  RootLayout         →  DashboardPage (Entrada)
/tasks          →  RootLayout         →  TasksPage (Lista de tarefas + Form de criacao e edicao)
/tasks/:id      →  RootLayout         →  TaskDetailPage
```

---

## 4. Layout compartilhado com `<Outlet />`

**Layout** é um componente que define a estrutura visual permanente da aplicação — o que aparece em todas as páginas — e reserva um espaço para o conteúdo variável.

<img src="https://preview.redd.it/learning-css-layouts-v0-xx3ossy3y9b51.jpg?auto=webp&s=24468f5bc567a2c220e2a2a013bc929719db2d4c" width="400">

O `<Outlet />` é esse espaço reservado: o React Router substitui ele pelo componente da rota filha ativa.

**`src/core/layouts/AuthLayout.tsx`**

```tsx
import { Outlet } from "react-router-dom";

export function AuthLayout() {
  return (
    <div className="min-h-screen flex flex-col items-center justify-center bg-gray-50 px-4 py-12 sm:px-6 lg:px-8">
      <div className="max-w-md w-full space-y-8 bg-white p-8 rounded-xl shadow-md border border-gray-100">
        <div className="flex flex-col items-center">
          <div className="h-12 w-12 rounded-lg bg-blue-600 flex items-center justify-center text-white text-2xl font-bold shadow-sm mb-2">
            T
          </div>
          <span className="text-2xl font-extrabold text-gray-900 tracking-tight">
            Task Manager
          </span>
        </div>

        <main>
          <Outlet />
        </main>
      </div>
    </div>
  );
}
```

**`src/core/layouts/RootLayout.tsx`**

```tsx
import { Outlet } from "react-router-dom";

export function RootLayout() {
  return (
    <div className="flex h-screen">
      <main className="flex-1 overflow-y-auto p-6">
        <Outlet />
      </main>
    </div>
  );
}
```

Então agora faremos a parte inicial das nossas rotas, como foi definido acima

**`src/core/routes/index.tsx`**

```tsx
import { createBrowserRouter } from "react-router-dom";

import { RootLayout } from "../layouts/RootLayout"; // <- Importaremos os Layouts que criamos
import { AuthLayout } from "../layouts/AuthLayout"; // <- Importaremos os Layouts que criamos

export const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />, //Primeiro Uso
    children: [
      {
        index: true,
        element: <div>Dashboard</div>,
      },
      {
        path: "/tasks",
        element: <div>Tasks</div>,
      },
      {
        path: "/tasks/:id",
        element: <div>Task Details</div>,
      },
    ],
  },
  {
    path: "/login",
    element: <AuthLayout />, //Segundo Uso
    children: [
      {
        index: true,
        element: <div>Login Page</div>,
      },
    ],
  },
]);
```

> A propriedade `index: true` define qual rota filha renderiza quando nenhum path específico é correspondido — neste caso, o Dashboard é a página inicial.

### Como o fluxo funciona

```
Usuário acessa "/tasks"
        ↓
React Router encontra a rota com path "tasks"
        ↓
Renderiza RootLayout
        ↓
Dentro de RootLayout, substitui <Outlet /> por <div>Tasks</div>
        ↓
Resultado: Sidebar + TasksPage na tela
```

> 📖 Referência: [React Router — Outlet](https://reactrouter.com/6.30.1/components/outlet)

---

### Criando a Sidebar com navegação declarativa

Use `<NavLink>` em vez de `<a>` — ele expõe `{ isActive }` para estilizar o item de menu da rota atual, sem reload de página.

**`src/core/components/Sidebar.tsx`**

```tsx
import { NavLink } from "react-router-dom";

// Extrai a lógica de classe para evitar repetição
function navClass({ isActive }: { isActive: boolean }): string {
  return isActive
    ? "bg-blue-600 rounded px-3 py-2"
    : "px-3 py-2 hover:bg-gray-700 rounded";
}

export function Sidebar() {
  return (
    <aside className="w-56 bg-gray-900 text-white flex flex-col p-4 gap-2">
      <h1 className="text-lg font-bold mb-4">MyApp</h1>

      <NavLink to="/" end className={navClass}>
        Dashboard
      </NavLink>

      <NavLink to="/tasks" className={navClass}>
        Tasks
      </NavLink>
    </aside>
  );
}
```

E agora adicionamos nossa SideBar no RootLayout

**`src/core/layouts/RootLayout.tsx`**

```tsx
import { Outlet } from "react-router-dom";
import { Sidebar } from "../components/Sidebar";

export function RootLayout() {
  return (
    <div className="flex h-screen">
      <Sidebar />

      <main className="flex-1 overflow-y-auto p-6">
        <Outlet />
      </main>
    </div>
  );
}
```

> Em TypeScript, a função `navClass` precisa tipar explicitamente o parâmetro `{ isActive: boolean }` — o React Router fornece esse tipo via `NavLinkRenderProps`, mas tipando inline já é suficiente para este caso. O prop `end` no link `/` evita que ele fique marcado como ativo em todas as rotas.

> 📖 Referência: [React Router — NavLink](https://reactrouter.com/6.30.1/components/nav-link)

---

## ✏️ Exercício

Crie as Duas páginas e navegue entre elas usando a sidebar.

- Usando Sidebar tera que mover entre Dashboard e Tasks

E uma terceira pagina, a nevegação devera ser URL

- Usando a URL navegue para Login

Como fazer

- Devera ser feitas dentro de uma feature
- Deverão ser expostas por um barrel
- Deverão ser colocadas nas rotas

### `<DashboardPage />`

**`src/features/dashboard/pages/DashboardPage.tsx`**

```tsx
export function DashboardPage() {
  return (
    <div>
      <h1 className="text-2xl font-bold mb-2">Dashboard</h1>
      <p className="text-gray-500">Visão geral da aplicação.</p>
    </div>
  );
}
```

### `<TasksPage />`

**`src/features/tasks/pages/TasksPage.tsx`**

```tsx
export function TasksPage() {
  return (
    <div>
      <h1 className="text-2xl font-bold mb-2">Tasks</h1>
      <p className="text-gray-500">Lista de tarefas.</p>
    </div>
  );
}
```

### `<LoginPage />`

**`src/features/auth/pages/LoginPage.tsx`**

```tsx
export function LoginPage() {
  return (
    <div>
      <h1 className="text-2xl font-bold mb-2">Login</h1>
      <p className="text-gray-500">Faça login para continuar.</p>
    </div>
  );
}
```

> Para manter a arquitetura Feature-Based limpa e isolada, nenhuma página ou componente externo deve navegar pelas subpastas internas de uma feature. Em vez disso, expomos apenas o que é estritamente necessário através de um arquivo de fechamento centralizado. 💡 Conceito: O que é um Barrel? > Barrel (barril) é um padrão de design em software que consiste em consolidar as exportações de vários módulos em um único arquivo simplificado (geralmente chamado de index.ts). Ele funciona como uma "fachada" ou ponto de distribuição pública da feature.

src/features/tasks/index.ts

```ts
export * from "./pages/TasksPage";
```

src/features/dashboard/index.ts

```ts
export * from "./pages/DashboardPage";
```

src/features/auth/index.ts

```ts
export * from "./pages/LoginPage";
```

> Por fim adicionamos os dois novos componentes nas rotas

src/core/routes/index.tsx

```tsx
import { createBrowserRouter } from "react-router-dom";

import { RootLayout } from "../layouts/RootLayout";
import { AuthLayout } from "../layouts/AuthLayout";

import { DashboardPage } from "../../features/dashboard";
import { TasksPage } from "../../features/tasks";
import { LoginPage } from "../../features/auth";

export const router = createBrowserRouter([
  {
    path: "/",
    element: <RootLayout />,
    children: [
      {
        index: true,
        element: <DashboardPage />,
      },
      {
        path: "/tasks",
        element: <TasksPage />,
      },
      {
        path: "/tasks/:id",
        element: <div>Task Details</div>,
      },
    ],
  },
  {
    path: "/login",
    element: <AuthLayout />,
    children: [
      {
        index: true,
        element: <LoginPage />,
      },
    ],
  },
]);
```

### Resultado esperado

- Ir para rota `/tasks`
- Clicar em "Dashboard" → `DashboardPage` aparece à direita, URL vai para `/`
- Clicar em "Tasks" → `TasksPage` aparece à direita, URL vai para `/tasks`
- A Sidebar permanece visível nas duas navegações
- Ir para rota `/login`
- Sumira a Sidebar

---

## ✅ Checklist de conclusão

- [ ] `createBrowserRouter` define as rotas com `children`
- [ ] `RouterProvider` conecta o router ao React em `main.tsx`
- [ ] Non-null assertion `!` usada no `getElementById("root")`
- [ ] `RootLayout` renderiza `<Sidebar />` e `<Outlet />`
- [ ] `navClass` tipada com `{ isActive: boolean }`
- [ ] Navegar entre `/` e `/tasks` não recarrega a página

---

## 📚 Referências oficiais deste módulo

| Tema                | Link                                                                                                  |
| ------------------- | ----------------------------------------------------------------------------------------------------- |
| createBrowserRouter | [reactrouter.com — createBrowserRouter](https://reactrouter.com/6.30.1/routers/create-browser-router) |
| RouterProvider      | [reactrouter.com — RouterProvider](https://reactrouter.com/6.30.1/routers/router-provider)            |
| Outlet              | [reactrouter.com — Outlet](https://reactrouter.com/6.30.1/components/outlet)                          |
| NavLink             | [reactrouter.com — NavLink](https://reactrouter.com/6.30.1/components/nav-link)                       |
| Link                | [reactrouter.com — Link](https://reactrouter.com/6.30.1/components/link)                              |

---

> 🚀 **Próximo módulo:** Construindo uma página de CRUD com Contexto — gerenciamento de estado local compartilhado e criação/edição/remoção de dados.

