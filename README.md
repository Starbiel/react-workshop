# Módulo 1 — O que é React?

> 🎯 Objetivo: Entender a mentalidade do React **antes** de escrever código.

---

## 1. O problema que React resolve!

Antes do React, construir interfaces interativas significava manipular o DOM manualmente com JavaScript puro. Isso funciona para páginas simples, mas rapidamente se torna um problema à medida que a aplicação cresce.

### Os três problemas clássicos

| Problema                      | O que acontece na prática                                                                  |
| ----------------------------- | ------------------------------------------------------------------------------------------ |
| **Manipulação manual do DOM** | Você precisa localizar cada elemento, alterar seu conteúdo e gerenciar eventos manualmente |
| **Complexidade crescente**    | Com mais funcionalidades, o código se torna difícil de manter e propensa a bugs            |
| **Reutilização de interface** | Repetir a mesma estrutura HTML em vários lugares sem uma abstração clara                   |

![Alt text for the GIF](https://media.tenor.com/yoA0Ygg9FXgAAAAM/food.gifhttps://media.tenor.com/yoA0Ygg9FXgAAAAM/food.gif)

### Comparação direta

**Sem React — JavaScript puro:**

```js
// Você precisa encontrar o elemento, criar conteúdo e inserir manualmente
const card = document.getElementById("task-card");
card.innerHTML = `<h2>${task.title}</h2><p>${task.description}</p>`;
card.addEventListener("click", () => {
  card.classList.toggle("completed");
});
```

**Com React — componente declarativo:**

```jsx
// Você descreve *o que* quer exibir, não *como* manipular o DOM
<TaskCard title={task.title} description={task.description} />
```

> 💡 **Insight-chave:** Com React, você descreve **como a UI deve parecer** para um dado estado. O React cuida de atualizar o DOM para você.  
> 📖 Referência: [React Docs — Thinking in React](https://react.dev/learn/thinking-in-react)

---

## 2. React em uma frase

> **React é uma biblioteca JavaScript para construir interfaces de usuário através de componentes reutilizáveis.**

Três palavras importantes nessa definição:

- **Biblioteca** — React cuida apenas da camada de UI. Você escolhe as outras ferramentas (roteamento, estado global, etc.)
- **Interfaces de usuário** — React vive na camada visual da sua aplicação
- **Componentes reutilizáveis** — A unidade fundamental do React

> 📖 Referência: [React Docs — Quick Start](https://react.dev/learn)

---

## 3. Conceitos fundamentais

### 3.1 Componentes

Componentes são **funções JavaScript que retornam marcação (JSX)**. Eles são os blocos de construção de qualquer aplicação React.

```jsx
// Um componente é uma função que começa com letra maiúscula
function WelcomeMessage() {
  return <h1>Olá, seja bem-vindo!</h1>;
}
```

Pense em componentes como peças de LEGO: cada um é independente, reutilizável e pode ser combinado para formar interfaces complexas.

> 📖 Referência: [React Docs — Your First Component](https://react.dev/learn/your-first-component)

---

### 3.2 JSX

JSX é uma **extensão de sintaxe do JavaScript** que permite escrever marcação parecida com HTML dentro do JS. O React não exige JSX, mas ele torna o código muito mais legível.

```jsx
// JSX parece HTML, mas é JavaScript
function Greeting() {
  const name = "Ana";
  return (
    <div className="greeting">
      <h1>Olá, {name}!</h1>
      <p>Hoje é um ótimo dia para aprender React.</p>
    </div>
  );
}
```

**Regras essenciais do JSX:**

- Use `className` em vez de `class`
- Todo componente deve retornar **um único elemento raiz**
- Expressões JavaScript ficam entre `{ }`
- Tags sem filhos devem ser auto-fechadas: `<Input />`

> 📖 Referência: [React Docs — Writing Markup with JSX](https://react.dev/learn/writing-markup-with-jsx)

---

### 3.2.1 TSX

TSX é **JSX + TypeScript**. Ou seja, a mesma sintaxe de marcação do JSX, mas dentro de arquivos `.tsx`, com **tipos estáticos** para evitar erros e melhorar o autocomplete.

```tsx
// TSX adiciona tipos ao JSX
type GreetingProps = {
  name: string;
  isMember?: boolean;
};

function Greeting({ name, isMember = false }: GreetingProps) {
  return (
    <div className="greeting">
      <h1>Ola, {name}!</h1>
      <p>
        {isMember ? "Bem-vindo de volta!" : "Crie sua conta para continuar."}
      </p>
    </div>
  );
}
```

**Quando usar TSX:**

- Ao criar componentes React com TypeScript
- Quando quiser tipar `props`, `state` e eventos de forma segura
- Para manter consistencia e reduzir bugs em projetos maiores

<img src="https://external-preview.redd.it/i-love-typescript-tho-v0-8ECpfNcBS7Ylknv6cEY5IC_VPHND3ix8003BUNGuwy8.png?width=1080&crop=smart&auto=webp&s=ede90a881072ab942c53c7acf7ab8829027bc4bc" width="300">

> 📖 Referencia: [React Docs — TypeScript](https://react.dev/learn/typescript)

---

### 3.3 Props

Props (abreviação de _properties_) são a forma de **passar dados de um componente pai para um filho**. Elas tornam os componentes configuráveis e reutilizáveis.

```jsx
// O componente recebe props como parâmetro
function Button({ label, color }) {
  return <button style={{ backgroundColor: color }}>{label}</button>;
}

// Usando o componente com diferentes props
function App() {
  return (
    <div>
      <Button label="Salvar" color="green" />
      <Button label="Cancelar" color="red" />
      <Button label="Editar" color="blue" />
    </div>
  );
}
```

**Versao TSX (com tipos):**

```tsx
type ButtonProps = {
  label: string;
  color: string;
};

function Button({ label, color }: ButtonProps) {
  return <button style={{ backgroundColor: color }}>{label}</button>;
}
```

> ⚠️ **Regra importante:** Props são **somente leitura**. Um componente nunca deve modificar suas próprias props.  
> 📖 Referência: [React Docs — Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component)

---

### 3.4 State

State (estado) é a **memória de um componente** — dados que podem mudar ao longo do tempo e que, quando mudam, fazem o componente re-renderizar.

```jsx
import { useState } from "react";

function Counter() {
  // [valorAtual, funçãoParaAtualizar] = useState(valorInicial)
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Contagem: {count}</p>
      <button onClick={() => setCount(count + 1)}>Incrementar</button>
    </div>
  );
}
```

**Versao TSX (tipando o state):**

```tsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState<number>(0);

  return (
    <div>
      <p>Contagem: {count}</p>
      <button onClick={() => setCount(count + 1)}>Incrementar</button>
    </div>
  );
}
```

**Props vs State — quando usar cada um:**

|                    | Props                   | State                     |
| ------------------ | ----------------------- | ------------------------- |
| **Quem controla**  | Componente pai          | O próprio componente      |
| **Pode mudar?**    | Não (somente leitura)   | Sim (com `setState`)      |
| **Para que serve** | Configurar o componente | Armazenar dados que mudam |

> 📖 Referência: [React Docs — State: A Component's Memory](https://react.dev/learn/state-a-components-memory)

---

### 3.5 Re-renderização

Sempre que o **state muda**, o React re-renderiza o componente automaticamente e atualiza apenas o que mudou no DOM — isso é o que torna o React eficiente.

```jsx
function Toggle() {
  const [isOn, setIsOn] = useState(false);

  // Cada vez que setIsOn é chamado, este componente re-renderiza
  return (
    <button onClick={() => setIsOn(!isOn)}>
      {isOn ? "✅ Ligado" : "❌ Desligado"}
    </button>
  );
}
```

**Versao TSX (tipando o state boolean):**

```tsx
import { useState } from "react";

function Toggle() {
  const [isOn, setIsOn] = useState<boolean>(false);

  return (
    <button onClick={() => setIsOn(!isOn)}>
      {isOn ? "✅ Ligado" : "❌ Desligado"}
    </button>
  );
}
```

> 📖 Referência: [React Docs — Render and Commit](https://react.dev/learn/render-and-commit)

---

## 4. Fluxo de renderização

Entender **quando e por que** o React re-renderiza é essencial para construir aplicações previsíveis.

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   1. State muda  →  setCount(count + 1)             │
│          ↓                                          │
│   2. React chama o componente novamente             │
│          ↓                                          │
│   3. React compara o novo resultado com o anterior  │
│          ↓                                          │
│   4. Apenas as diferenças são aplicadas no DOM      │
│          ↓                                          │
│   5. UI atualizada ✓                                │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Na prática:**

```jsx
function ScoreBoard() {
  const [score, setScore] = useState(0);

  // 1️⃣ Usuário clica → setScore é chamado
  // 2️⃣ React re-renderiza ScoreBoard com o novo valor
  // 3️⃣ Apenas o texto do <p> é atualizado no DOM
  return (
    <div>
      <p>Pontuação: {score}</p>
      <button onClick={() => setScore(score + 10)}>+ 10 pontos</button>
    </div>
  );
}
```

> 📖 Referência: [React Docs — Render and Commit](https://react.dev/learn/render-and-commit)

---

## ✏️ Exercício rápido

Crie **três componentes**

- `<Card />`
- `<UserInfo />`
- `<Button />`

Os quais:

- Recebam **props**
- Retornem **JSX**

### 🎯 Objetivo

Praticar a **composição de componentes**.

### 💻 Ambiente de Desenvolvimento

Como ainda não configuramos um projeto React localmente, utilize o compilador online:

> https://stackblitz.com/edit/react-ts-playground?file=index.tsx

> Não utilize `state` ainda.  
> O foco deste exercício é praticar a **composição de componentes**.

## Desafio Extra

- Implementar os componentes em **TSX (TypeScript + React)**.

### Componente 1 — `<Card />`

```jsx
// Exibe um cartão com título e descrição
function Card({ title, description }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <p>{description}</p>
    </div>
  );
}

// Uso:
<Card
  title="React Hooks"
  description="Funções especiais do React para gerenciar estado e efeitos."
/>;
```

---

### Componente 2 — `<UserInfo />`

```jsx
// Exibe informações de um usuário
function UserInfo({ name, role, avatarUrl }) {
  return (
    <div className="user-info">
      <img src={avatarUrl} alt={name} />
      <div>
        <strong>{name}</strong>
        <span>{role}</span>
      </div>
    </div>
  );
}

// Uso:
<UserInfo
  name="Ana Souza"
  role="Desenvolvedora Frontend"
  avatarUrl="https://i.pravatar.cc/48"
/>;
```

---

### Componente 3 — `<Button />`

```jsx
// Botão reutilizável com variantes
function Button({ label, variant }) {
  return (
    <button className={`btn btn-${variant}`}>
      {label}
    </button>
  );
}

// Uso:
<Button label="Confirmar" variant="primary" />
<Button label="Cancelar" variant="secondary" />
```

---

### Desafio: componha os três juntos

```jsx
function App() {
  return (
    <div>
      <UserInfo
        name="Ana Souza"
        role="Desenvolvedora Frontend"
        avatarUrl="https://i.pravatar.cc/48"
      />
      <Card
        title="Introdução ao React"
        description="Aprenda a construir interfaces com componentes."
      />
      <Button label="Começar agora" variant="primary" />
    </div>
  );
}
```

> ✅ **Critério de sucesso:** Os três componentes renderizam corretamente recebendo dados via props, sem hardcoded values dentro deles.

---

## 📚 Referências oficiais deste módulo

| Tema                    | Link                                                                                 |
| ----------------------- | ------------------------------------------------------------------------------------ |
| Visão geral do React    | [react.dev/learn](https://react.dev/learn)                                           |
| Pensando em React       | [Thinking in React](https://react.dev/learn/thinking-in-react)                       |
| Seu primeiro componente | [Your First Component](https://react.dev/learn/your-first-component)                 |
| Escrevendo JSX          | [Writing Markup with JSX](https://react.dev/learn/writing-markup-with-jsx)           |
| Passando props          | [Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component) |
| State e memória         | [State: A Component's Memory](https://react.dev/learn/state-a-components-memory)     |
| Renderização            | [Render and Commit](https://react.dev/learn/render-and-commit)                       |

---

> 🚀 **Próximo módulo:** Componentes na prática — estruturação, composição e boas práticas de organização de arquivos.

# Módulo 2 — Criando o Projeto

> 🎯 Objetivo: Conhecer o ecossistema e ter o projeto rodando localmente.

---

## 1. Stack utilizada

Antes de criar qualquer arquivo, entenda **por que** cada ferramenta foi escolhida.

| Ferramenta       | Função                  | Por que essa escolha                                 |
| ---------------- | ----------------------- | ---------------------------------------------------- |
| **React**        | Biblioteca de UI        | Base de componentes reutilizáveis e foco do workshop |
| **Vite**         | Bundler e dev server    | Inicialização e HMR extremamente rápidos             |
| **Tailwind CSS** | Estilização             | Utilitários inline, sem CSS separado                 |
| **React Router** | Navegação entre páginas | Roteamento declarativo, padrão do ecossistema        |
| **React Query**  | Busca e cache de dados  | Gerencia loading, erro e cache automaticamente       |
| **Axios**        | Cliente HTTP            | API mais ergonômica que o `fetch` nativo             |

<img src="https://preview.redd.it/axioscompromised-v0-e8p4v91g8bsg1.jpeg?auto=webp&s=a8167f44b7269a600e15c6524ec55ed5c74f1eec" width="300">

> 📖 Referências:
>
> - [Vite — Getting Started](https://vitejs.dev/guide/)
> - [React Router — Installation](https://reactrouter.com/start/library/installation)
> - [TanStack Query — Quick Start](https://tanstack.com/query/latest/docs/framework/react/quick-start)
> - [Tailwind CSS — Installation com Vite](https://tailwindcss.com/docs/installation/using-vite)

---

## 2. Criação do projeto

> Pré-requisito: Ter o **node** instalado

Use o Vite para gerar o projeto. Ele criará toda a estrutura inicial com configuração de desenvolvimento pronta.

```bash
npm create vite@latest
```

> Usar um "." no final indica que o projeto será criado na pasta atual. Assim, não será necessário informar um nome durante o assistente.

Durante o assistente interativo, responda:

```
✔ Project name: meu-projeto (Caso tenha utilizado . nao aparecera)
✔ Select a framework: React
✔ Select a variant: Typescript
✔ Install with npm and start now?: Yes
```

> Ao abrir `http://localhost:5173`, você verá a página padrão do Vite + React. Isso confirma que o ambiente está funcionando.

> 📖 Referência: [Vite — Scaffolding Your First Project](https://vitejs.dev/guide/#scaffolding-your-first-vite-project)

---

## 3. Instalando as dependências

Instale cada biblioteca separadamente para entender o que cada `npm install` adiciona ao projeto.

### React Router DOM — navegação entre páginas

```bash
npm install react-router-dom
```

### TanStack React Query — busca e cache de dados

```bash
npm install @tanstack/react-query
```

### Axios — cliente HTTP

```bash
npm install axios
```

### Tailwind CSS — estilização com utilitários

```bash
npm install tailwindcss @tailwindcss/vite
```

Após instalar o Tailwind, configure o plugin no Vite. Edite o arquivo `vite.config.ts`:

```js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

Adicione o import do Tailwind no seu arquivo de estilos global (ex: `src/index.css`):

```css
@import "tailwindcss";
```

> ✅ Para confirmar que o Tailwind está funcionando, adicione `className="text-blue-500"` em qualquer elemento e veja a cor no browser.

> 📖 Referências:
>
> - [Tailwind CSS — Using Vite](https://tailwindcss.com/docs/installation/using-vite)
> - [React Router — Installation](https://reactrouter.com/start/library/installation)
> - [TanStack Query — Installation](https://tanstack.com/query/latest/docs/framework/react/installation)

---

## 4. Estrutura inicial do projeto

## Por que utilizar uma arquitetura Feature-Based?

Uma boa estrutura de pastas evita o caos à medida que o projeto cresce.

### Benefícios

- Facilita o **onboarding** de novos desenvolvedores, pois cada feature representa um domínio isolado.
- Melhora a **propriedade do código**, permitindo que diferentes times sejam responsáveis por features específicas.
- Reduz o **acoplamento** entre módulos.
- Diminui **conflitos durante alterações e merges**.
- Torna **testes e deploys mais previsíveis**.
- Simplifica a **localização e correção de bugs**.
- Aumenta a **escalabilidade** do projeto.
- Melhora a **manutenção** a longo prazo.
- Favorece a **colaboração entre equipes**.

### Quando utilizar?

- **Projetos pequenos** ou **bibliotecas** podem funcionar bem com uma estrutura por camadas (`core/`, `components/`, etc.).
- **Projetos de médio e grande porte** se beneficiam significativamente da abordagem **feature-based**.

```
src/
├── core/                  # Configurações e recursos globais
│   ├── http/              # Instância configurada do Axios
│   │   └── api.js/ts
│   ├── providers/         # Providers globais (QueryClient, Router)
│   │   └── AppProviders.jsx/tsx
│   └── routes/            # Definição de todas as rotas
│       └── index.jsx/tsx
│
├── features/              # Cada feature é um módulo isolado
│   └── tasks/             # Exemplo: feature de tarefas
│       ├── components/    # Componentes específicos da feature
│       ├── hooks/         # Hooks de dados (useQuery, useMutation)
│       ├── services/      # Chamadas de API dessa feature
│       └── pages/         # Páginas da feature
│
├── App.tsx               # Componente raiz
└── main.tsx               # Ponto de entrada da aplicação
```

### Por que essa estrutura?

- **`core/`** — Tudo que é global e compartilhado entre features (configuração do Axios, providers, rotas)
- **`features/`** — Cada domínio da aplicação fica isolado; você pode entender ou deletar uma feature sem tocar nas outras
- **`App.jsx`** — Apenas monta os providers e o roteador
- **`main.jsx`** — Apenas renderiza o `<App />` no DOM

---

### Configurando o ponto de entrada

**`src/main.tsx`**

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import "./index.css";
import App from "./App";
createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

**`src/core/providers/AppProviders.tsx`**

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { BrowserRouter } from "react-router-dom";

const queryClient = new QueryClient();

interface AppProvidersProps {
  children: React.ReactNode;
}

export function AppProviders({ children }: AppProvidersProps) {
  return (
    <QueryClientProvider client={queryClient}>
      <BrowserRouter>{children}</BrowserRouter>
    </QueryClientProvider>
  );
}
```

**`src/App.tsx`**

```tsx
import { AppProviders } from "./core/providers/AppProviders";
function App() {
  return (
    <AppProviders>
      <h1 className="text-2xl font-bold text-blue-500">Projeto iniciado ✅</h1>
    </AppProviders>
  );
}
export default App;
```

> 📖 Referência: [React Docs — Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)

---

## ✏️ Checklist de conclusão

Antes de seguir para o próximo módulo, confirme cada item:

- [ ] `npm run dev` inicia o servidor sem erros
- [ ] A página abre em `http://localhost:5173`
- [ ] Tailwind está funcionando (teste com `className="text-red-500"`)
- [ ] A estrutura de pastas `core/` e `features/` foi criada
- [ ] `AppProviders` envolve a aplicação com `QueryClientProvider` e `BrowserRouter`

---

## 📚 Referências oficiais deste módulo

| Tema                        | Link                                                                                                                               |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Criando projeto com Vite    | [vitejs.dev/guide](https://vitejs.dev/guide/)                                                                                      |
| Tailwind com Vite           | [tailwindcss.com/docs/installation/using-vite](https://tailwindcss.com/docs/installation/using-vite)                               |
| React Router — instalação   | [reactrouter.com/start/library/installation](https://reactrouter.com/start/library/installation)                                   |
| TanStack Query — instalação | [tanstack.com/query/latest/docs/framework/react/installation](https://tanstack.com/query/latest/docs/framework/react/installation) |
| StrictMode                  | [react.dev/reference/react/StrictMode](https://react.dev/reference/react/StrictMode)                                               |
| createRoot                  | [react.dev/reference/react-dom/client/createRoot](https://react.dev/reference/react-dom/client/createRoot)                         |

---

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

# Módulo 4 — Construindo uma página de CRUD com Contexto

> 🎯 Objetivo: Aprender a gerenciar e compartilhar o estado de criação, edição e remoção (CRUD) de dados entre componentes de forma limpa e modular usando a React Context API.

---

## 1. O problema que o Contexto resolve (Prop Drilling)

Quando desenvolvemos páginas interativas com múltiplos componentes (uma listagem de cards, modais de criação/edição e caixas de diálogo para confirmação de exclusão), surge um problema clássico: **Prop Drilling**.

### O que é Prop Drilling?

Imagine que o botão de "Excluir" está dentro do componente `<TaskCard />`, mas o diálogo de confirmação de exclusão `<ConfirmDialog />` está na raiz da página `<TasksPage />`.
Para saber qual tarefa o usuário deseja excluir e se o diálogo está aberto, você precisaria criar estados no componente pai e passá-los como propriedades (`props`) por vários níveis de componentes, mesmo aqueles que não usam esses dados diretamente.

```
Sem Contexto (Prop Drilling):
TasksPage (Guarda estados: selectedTask, isDeleteDialogOpen)
   └── ListTasks (Apenas repassa handlers por prop)
          └── TaskCard (Chama o handler onClick)
```

Com a **React Context API**, podemos criar um "provedor de estado" em torno da nossa feature. Qualquer componente filho dentro desse provedor pode se conectar diretamente e consumir ou alterar o estado global sem intermediários.

```
Com Contexto:
[TaskCrudProvider (Guarda estados e funções)]
      ├── ListTasks
      │      └── TaskCard ────> (Consome openDeleteDialog, openEditModal)
      ├── UpsertTask ─────────> (Consome selectedTask, isFormModalOpen, closeFormModal)
      └── DeleteTask ─────────> (Consome selectedTask, isDeleteDialogOpen, closeDeleteDialog)
```

> 💡 **Insight-chave:** A Context API é perfeita para estados de UI locais compartilhados em uma mesma feature, evitando que o estado visual (se um modal está aberto ou não) polua componentes que não se importam com isso.

---

## 2. Componentes de UI Globais

Para realizar nosso CRUD, precisaremos de dois componentes de interface comuns: um modal genérico e uma caixa de diálogo para confirmação de exclusão. Estes componentes residem em nosso diretório global de UI.

### 💻 Componente 1: `<Modal />`

Este é um modal acessível e simples que escurece o fundo (backdrop) e centraliza o formulário na tela.

**`src/core/components/ui/Modal.tsx`**

```tsx
interface ModalProps {
  open: boolean;
  title?: string;
  description?: string;
  onClose: () => void;
  children: React.ReactNode;
}

export function Modal({
  open,
  title,
  description,
  onClose,
  children,
}: ModalProps) {
  if (!open) return null;

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center">
      {/* Backdrop */}
      <div
        className="absolute inset-0 bg-black/50 backdrop-blur-sm"
        onClick={onClose}
      />

      {/* Content */}
      <div className="relative z-10 w-full max-w-lg rounded-2xl border border-slate-200 bg-white shadow-xl">
        <div className="flex items-start justify-between border-b p-6">
          <div>
            {title && (
              <h2 className="text-lg font-semibold text-slate-900">{title}</h2>
            )}

            {description && (
              <p className="mt-1 text-sm text-slate-500">{description}</p>
            )}
          </div>

          <button
            onClick={onClose}
            className="rounded-md p-2 transition hover:bg-slate-100"
          >
            X
          </button>
        </div>

        <div className="p-6">{children}</div>
      </div>
    </div>
  );
}
```

---

### 💻 Componente 2: `<ConfirmDialog />`

Usado para confirmar ações destrutivas como a exclusão de registros.

**`src/core/components/ui/ConfimDialog.tsx`**

```tsx
interface ConfirmDialogProps {
  open: boolean;
  title: string;
  description: string;

  confirmLabel?: string;
  cancelLabel?: string;

  onConfirm: () => void;
  onCancel: () => void;

  loading?: boolean;
}

export function ConfirmDialog({
  open,
  title,
  description,

  confirmLabel = "Confirmar",
  cancelLabel = "Cancelar",

  onConfirm,
  onCancel,

  loading,
}: ConfirmDialogProps) {
  if (!open) return null;

  return (
    <div className="fixed inset-0 z-50 flex items-center justify-center">
      <div
        className="absolute inset-0 bg-black/50 backdrop-blur-sm"
        onClick={onCancel}
      />

      <div className="relative z-10 w-full max-w-md rounded-2xl border border-slate-200 bg-white p-6 shadow-xl">
        <div>
          <h2 className="text-lg font-semibold">{title}</h2>

          <p className="mt-2 text-sm text-slate-500">{description}</p>
        </div>

        <div className="mt-6 flex justify-end gap-3">
          <button
            onClick={onCancel}
            className="rounded-lg border border-slate-300 px-4 py-2 text-sm font-medium hover:bg-slate-50"
          >
            {cancelLabel}
          </button>

          <button
            onClick={onConfirm}
            disabled={loading}
            className="rounded-lg bg-red-600 px-4 py-2 text-sm font-medium text-white transition hover:bg-red-700 disabled:opacity-50"
          >
            {loading ? "Processando..." : confirmLabel}
          </button>
        </div>
      </div>
    </div>
  );
}
```

---

## 3. Implementação da Feature `tasks`

A seguir, construiremos nossa feature de tarefas seguindo a ordem recomendada de responsabilidade única e modularização de código.

```
Ordem de Criação:
1. Tipagem (type.ts)
2. Contexto (TaskCrudContext.tsx)
3. Componentes de UI (TaskCard.tsx, TaskForm.tsx)
4. Containers Orquestradores (ListTasks.tsx, UpsertTask.tsx, DeleteTask.tsx)
5. Página Receptora (TasksPage.tsx)
```

---

### 💻 Passo 1: Definir os Tipos e Variáveis da Feature

Definimos a estrutura básica dos dados que serão manipulados na aplicação.

**`src/features/tasks/type.ts`**

```typescript
export type TaskPriority = "LOW" | "MEDIUM" | "HIGH";

export type Task = {
  id: string;
  name: string;
  description: string | null;
  dueDate: Date | null;
  priority: TaskPriority;

  completed: boolean;

  createdAt?: string;
  updatedAt?: string;
};

export interface TaskDTO extends Omit<
  Task,
  "id" | "createdAt" | "updatedAt" | "completed"
> {}
```

---

### 💻 Passo 2: Criar o Contexto de CRUD (`TaskCrudContext`)

Este contexto servirá para orquestrar os estados de qual tarefa está selecionada e quais modais de edição ou confirmação estão abertos na tela.

**`src/features/tasks/contexts/TaskCrudContext.tsx`**

```tsx
import {
  createContext,
  useCallback,
  useContext,
  useMemo,
  useState,
  type ReactNode,
} from "react";
import type { Task } from "../type";

interface TaskCrudContextData {
  selectedTask?: Task;

  isFormModalOpen: boolean;
  isDeleteDialogOpen: boolean;

  openCreateModal: () => void;
  openEditModal: (task: Task) => void;
  closeFormModal: () => void;

  openDeleteDialog: (task: Task) => void;
  closeDeleteDialog: () => void;

  clearSelection: () => void;
}

const TaskCrudContext = createContext<TaskCrudContextData | null>(null);

interface TaskCrudProviderProps {
  children: ReactNode;
}

export function TaskCrudProvider({ children }: TaskCrudProviderProps) {
  const [selectedTask, setSelectedTask] = useState<Task | undefined>(undefined);

  const [isFormModalOpen, setIsFormModalOpen] = useState(false);
  const [isDeleteDialogOpen, setIsDeleteDialogOpen] = useState(false);

  const openCreateModal = useCallback(() => {
    setSelectedTask(undefined);
    setIsFormModalOpen(true);
  }, []);

  const openEditModal = useCallback((task: Task) => {
    setSelectedTask(task);
    setIsFormModalOpen(true);
  }, []);

  const closeFormModal = useCallback(() => {
    setIsFormModalOpen(false);
  }, []);

  const openDeleteDialog = useCallback((task: Task) => {
    setSelectedTask(task);
    setIsDeleteDialogOpen(true);
  }, []);

  const closeDeleteDialog = useCallback(() => {
    setIsDeleteDialogOpen(false);
  }, []);

  const clearSelection = useCallback(() => {
    setSelectedTask(undefined);
    setIsFormModalOpen(false);
    setIsDeleteDialogOpen(false);
  }, []);

  const value = useMemo(
    () => ({
      selectedTask,

      isFormModalOpen,
      isDeleteDialogOpen,

      openCreateModal,
      openEditModal,
      closeFormModal,

      openDeleteDialog,
      closeDeleteDialog,

      clearSelection,
    }),
    [
      selectedTask,
      isFormModalOpen,
      isDeleteDialogOpen,
      openCreateModal,
      openEditModal,
      closeFormModal,
      openDeleteDialog,
      closeDeleteDialog,
      clearSelection,
    ],
  );

  return (
    <TaskCrudContext.Provider value={value}>
      {children}
    </TaskCrudContext.Provider>
  );
}

export function useTaskCrud() {
  const context = useContext(TaskCrudContext);

  if (!context) {
    throw new Error("useTaskCrud must be used within TaskCrudProvider");
  }

  return context;
}
```

---

### 💻 Passo 3: Criar o Componente Visual `<TaskCard />`

Esse componente representa um único card visual de tarefa com prioridade e prazo. Ele expõe cliques de edição e exclusão.

**`src/features/tasks/components/TaskCard.tsx`**

```tsx
import type { Task } from "../type";

interface TaskCardProps {
  task: Task;

  onEdit: (task: Task) => void;
  onDelete: (task: Task) => void;
  onComplete: (task: Task) => void;
}

//Isto pode virar um funcao auxiliar dentro de utils ou algo do tipo
function formatDueDate(dueDate: Date | string) {
  const dateString =
    typeof dueDate === "string" ? dueDate : dueDate.toISOString();
  const localDate = new Date(dateString.replace("Z", ""));

  return localDate.toLocaleString("pt-BR", {
    day: "2-digit",
    month: "2-digit",
    year: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  });
}

export function TaskCard({
  task,
  onEdit,
  onDelete,
  onComplete,
}: TaskCardProps) {
  const isCompleted = task.completed;
  const isOverdue = task.dueDate
    ? new Date(task.dueDate) < new Date() && !isCompleted
    : false;

  const statusText = isCompleted ? "Concluída" : "Pendente";

  const statusIcon = isCompleted ? "✓" : "⏳";

  const priorityColor = {
    LOW: "bg-green-100 text-green-700",
    MEDIUM: "bg-yellow-100 text-yellow-700",
    HIGH: "bg-red-100 text-red-700",
  }[task.priority];

  return (
    <li className="rounded-lg border border-slate-200 bg-white p-4 shadow-sm">
      <div className="flex items-start justify-between">
        <div className="flex gap-3">
          <div
            className={`flex h-8 w-8 items-center justify-center rounded-full ${
              isCompleted
                ? "bg-green-100 text-green-700"
                : "bg-yellow-100 text-yellow-700"
            }`}
          >
            {statusIcon}
          </div>

          <div>
            <h3
              className={`font-semibold ${
                isCompleted ? "text-slate-400 line-through" : "text-slate-800"
              }`}
            >
              {task.name}
            </h3>

            {task.description && (
              <p className="mt-1 text-sm text-slate-500">{task.description}</p>
            )}
          </div>
        </div>

        <span
          className={`rounded-full px-3 py-1 text-xs font-medium ${
            isCompleted
              ? "bg-green-100 text-green-700"
              : "bg-yellow-100 text-yellow-700"
          }`}
        >
          {statusText}
        </span>
      </div>

      <div className="mt-4 flex flex-wrap gap-2 text-sm">
        <span className={`rounded-full px-2 py-1 font-medium ${priorityColor}`}>
          Prioridade: {task.priority}
        </span>

        {task.dueDate && (
          <span
            className={`rounded-full px-2 py-1 text-sm font-medium ${isOverdue ? "bg-red-100 text-red-700" : "bg-slate-100 text-slate-600"}`}
          >
            Prazo: {formatDueDate(task.dueDate)}
          </span>
        )}
      </div>

      <div className="mt-4 flex justify-end gap-2 border-t border-slate-100 pt-4">
        {!isCompleted && (
          <button
            type="button"
            onClick={() => onComplete(task)}
            className="rounded-md border border-green-200 bg-green-50 px-3 py-2 text-sm font-medium text-green-700 transition hover:bg-green-100 cursor-pointer"
          >
            Concluir
          </button>
        )}
        <button
          type="button"
          onClick={() => onEdit(task)}
          className="rounded-md border border-slate-300 px-3 py-2 text-sm font-medium text-slate-700 transition hover:bg-slate-50 cursor-pointer"
        >
          Editar
        </button>

        <button
          type="button"
          onClick={() => onDelete(task)}
          className="rounded-md border border-red-200 bg-red-50 px-3 py-2 text-sm font-medium text-red-700 transition hover:bg-red-100 cursor-pointer"
        >
          Excluir
        </button>
      </div>
    </li>
  );
}
```

---

### 💻 Passo 4: Criar o Formulário da Tarefa `<TaskForm />`

Um formulário puro, sem acesso a contextos globais, garantindo que ele possa ser facilmente testado ou reutilizado em diferentes fluxos da aplicação. Ele detecta se é uma tarefa existente para preencher os valores iniciais.

**`src/features/tasks/components/TaskForm.tsx`**

```tsx
import { useState } from "react";
import type { Task, TaskDTO, TaskPriority } from "../type";

interface TaskFormProps {
  task?: Task;
  onSubmit: (data: TaskDTO) => Promise<void>;
}

export function TaskForm({ task, onSubmit }: TaskFormProps) {
  const [taskData, setTaskData] = useState<TaskDTO>({
    name: task?.name ?? "",
    description: task?.description ?? null,
    dueDate: task?.dueDate ?? null,
    priority: task?.priority ?? "LOW",
  });
  const [loading, setLoading] = useState(false);

  const dueDateValue = taskData.dueDate
    ? new Date(taskData.dueDate).toISOString().slice(0, 16)
    : "";

  async function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault();

    if (loading) return;

    setLoading(true);

    try {
      await onSubmit({
        name: taskData.name,
        description: taskData.description,
        dueDate: taskData.dueDate,
        priority: taskData.priority,
      });
    } finally {
      setLoading(false);
    }
  }

  const isEditing = !!task;

  return (
    <form onSubmit={handleSubmit} className="space-y-5">
      <div>
        <label
          htmlFor="name"
          className="mb-1 block text-sm font-medium text-slate-700"
        >
          Nome da tarefa
        </label>
        <input
          id="name"
          type="text"
          value={taskData.name}
          onChange={(event) =>
            setTaskData({ ...taskData, name: event.target.value })
          }
          required
          className="w-full rounded-lg border border-slate-300 px-3 py-2 outline-none transition focus:border-slate-500"
          placeholder="Ex: Estudar React"
        />
      </div>

      <div>
        <label
          htmlFor="description"
          className="mb-1 block text-sm font-medium text-slate-700"
        >
          Descrição
        </label>
        <textarea
          id="description"
          rows={4}
          value={taskData.description ?? ""}
          onChange={(event) =>
            setTaskData({ ...taskData, description: event.target.value })
          }
          className="w-full rounded-lg border border-slate-300 px-3 py-2 outline-none transition focus:border-slate-500"
          placeholder="Descreva a tarefa..."
        />
      </div>

      <div>
        <label
          htmlFor="dueDate"
          className="mb-1 block text-sm font-medium text-slate-700"
        >
          Data e Horário Limite
        </label>
        <input
          id="dueDate"
          type="datetime-local"
          value={dueDateValue}
          onChange={(event) =>
            setTaskData({
              ...taskData,
              dueDate: event.target.value ? new Date(event.target.value) : null,
            })
          }
          className="w-full rounded-lg border border-slate-300 px-3 py-2 outline-none transition focus:border-slate-500"
        />
      </div>

      <div>
        <label
          htmlFor="priority"
          className="mb-1 block text-sm font-medium text-slate-700"
        >
          Prioridade
        </label>
        <select
          id="priority"
          value={taskData.priority}
          onChange={(event) =>
            setTaskData({
              ...taskData,
              priority: event.target.value as TaskPriority,
            })
          }
          className="w-full rounded-lg border border-slate-300 px-3 py-2 outline-none transition focus:border-slate-500"
        >
          <option value="LOW">Baixa</option>
          <option value="MEDIUM">Média</option>
          <option value="HIGH">Alta</option>
        </select>
      </div>

      <button
        type="submit"
        className="w-full rounded-lg bg-slate-900 px-4 py-2 font-medium text-white transition hover:bg-slate-800 cursor-pointer"
        disabled={loading}
      >
        {loading
          ? "Salvando..."
          : isEditing
            ? "Salvar Alterações"
            : "Criar Tarefa"}
      </button>
    </form>
  );
}
```

---

### 💻 Passo 5: Criar o Container de Listagem `<ListTasks />`

Este container consome dados fictícios e invoca as funções de abrir diálogos de edição e remoção do `useTaskCrud`.

**`src/features/tasks/containers/ListTasks.tsx`**

```tsx
import { useCallback } from "react";
import { TaskCard } from "../components/TaskCard";
import { useTaskCrud } from "../contexts/TaskCrudContext";
import type { Task } from "../type";

const mookTask: Task[] = [
  {
    id: "1",
    name: "Task 1",
    description: "Descrição da Task 1",
    dueDate: new Date("2026-06-12T00:00:00.000Z"),
    priority: "MEDIUM",
    completed: false,
  },
  {
    id: "2",
    name: "Task 2",
    description: "Descrição da Task 2",
    dueDate: new Date("2026-06-18T20:45:00.000Z"),
    priority: "HIGH",
    completed: true,
  },
];

export function ListTasks() {
  const { openDeleteDialog, openEditModal } = useTaskCrud();

  const handleToggleComplete = useCallback((task: Task) => {
    // Adicionaremos em breve
  }, []);

  return (
    <div className="grid grid-cols-1 gap-4 sm:grid-cols-1 lg:grid-cols-1">
      {mookTask.map((task) => (
        <TaskCard
          key={task.id}
          task={task}
          onEdit={() => openEditModal(task)}
          onDelete={() => openDeleteDialog(task)}
          onComplete={() => handleToggleComplete(task)}
        />
      ))}
    </div>
  );
}
```

---

### 💻 Passo 6: Criar o Container de Formulário `<UpsertTask />`

Gerencia o modal de criação/edição conectando-se diretamente ao contexto para carregar as informações do formulário e fechar a modal quando necessário.

**`src/features/tasks/containers/UpsertTask.tsx`**

```tsx
import { useCallback } from "react";
import { TaskForm } from "../components/TaskForm";
import { useTaskCrud } from "../contexts/TaskCrudContext";
import { Modal } from "../../../core/components/ui";
import type { TaskDTO } from "../type";

export function UpsertTask() {
  const { selectedTask, isFormModalOpen, clearSelection } = useTaskCrud();

  const handleSubmit = useCallback(async (data: TaskDTO): Promise<void> => {
    console.log("Enviando dados:", data);

    await new Promise<void>((resolve) => {
      setTimeout(() => {
        resolve();
      }, 1000);
    });

    console.log("Promise resolvida com sucesso!");
  }, []);

  if (!isFormModalOpen) {
    return null;
  }

  return (
    <Modal
      open={isFormModalOpen}
      title={selectedTask ? "Editar Task" : "Criar Task"}
      description={
        selectedTask
          ? "Edite os detalhes da task abaixo."
          : "Preencha os detalhes da nova task."
      }
      onClose={clearSelection}
    >
      <TaskForm onSubmit={handleSubmit} task={selectedTask} />
    </Modal>
  );
}
```

---

### 💻 Passo 7: Criar o Container de Exclusão `<DeleteTask />`

Exibe o diálogo de confirmação consumindo o estado do `TaskCrudContext`.

**`src/features/tasks/containers/DeleteTask.tsx`**

```tsx
import { useState } from "react";
import { ConfirmDialog } from "../../../core/components/ui";
import { useTaskCrud } from "../contexts/TaskCrudContext";

export function DeleteTask() {
  const { isDeleteDialogOpen, selectedTask, clearSelection } = useTaskCrud();
  const [isLoading, setIsLoading] = useState(false);

  if (!isDeleteDialogOpen && !selectedTask) {
    return null;
  }

  const handleDelete = () => {
    alert(`Task "${selectedTask?.name}" excluída!`);
  };

  return (
    <ConfirmDialog
      open={isDeleteDialogOpen}
      title="Excluir Task"
      description={`Tem certeza que deseja excluir a task "${selectedTask?.name}"? Esta ação não pode ser desfeita.`}
      confirmLabel="Excluir"
      cancelLabel="Cancelar"
      onConfirm={handleDelete}
      onCancel={clearSelection}
      loading={isLoading}
    />
  );
}
```

---

### 💻 Passo 8: Integrar Tudo na Página `<TasksPage />`

Agora a página de tarefas simplesmente posiciona os containers orquestradores na tela e fornece um botão para abrir o modal de criação inicial.

**`src/features/tasks/pages/TasksPage.tsx`**

```tsx
import { DeleteTask } from "../containers/DeleteTask";
import { ListTasks } from "../containers/ListTasks";
import { UpsertTask } from "../containers/UpsertTask";
import { useTaskCrud } from "../contexts/TaskCrudContext";

export function TasksPage() {
  const { openCreateModal } = useTaskCrud();
  return (
    <div className="max-w-4xl mx-auto">
      <header className="flex justify-between items-center mb-8">
        <div>
          <h1 className="text-3xl font-bold text-slate-950 tracking-tight mb-2">
            Gerenciar Tarefas
          </h1>
        </div>
        <div>
          <button
            onClick={openCreateModal}
            className="bg-blue-500 hover:bg-blue-600 text-white font-medium py-2 px-4 rounded-md cursor-pointer"
          >
            Criar Tarefa
          </button>
        </div>
      </header>
      <ListTasks />

      <UpsertTask />

      <DeleteTask />
    </div>
  );
}
```

---

### 💻 Passo 9: Acoplar o Provider nas Rotas

No Módulo 3 nós apenas injetamos a página `<TasksPage />`. Agora nós precisamos envolver essa página com o `<TaskCrudProvider />` para que os contextos estejam disponíveis para os componentes filhos dessa rota.

**`src/core/routes/index.tsx`**

```tsx
import { createBrowserRouter } from "react-router-dom";

import { RootLayout } from "../layouts/RootLayout";
import { AuthLayout } from "../layouts/AuthLayout";

import { DashboardPage } from "../../features/dashboard";
import { TasksPage, TaskCrudProvider } from "../../features/tasks";
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
        element: (
          <TaskCrudProvider>
            <TasksPage />
          </TaskCrudProvider>
        ),
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

---

### 💻 Passo 10: Atualizar o Barrel Export

Certifique-se de que o provedor de contexto esteja devidamente exportado da feature para ser consumido externamente.

**`src/features/tasks/index.ts`**

```typescript
export * from "./pages/TasksPage";
export * from "./contexts/TaskCrudContext";
```

---

## ✏️ Exercício Prático

Implemente o fluxo completo do CRUD local em sua aplicação. O seu desafio será:

1. Modificar o container `<ListTasks />` para utilizar um estado local (`useState`) inicializado com a lista fictícia (`mookTask`).
2. Passar a lista e a função para atualizá-la (ou criar funções de adição/edição/deleção dedicadas) de forma que quando você preencher o formulário do modal ou clicar em Confirmar no diálogo de exclusão, as alterações sejam refletidas de verdade na tela.
3. Garanta que o formulário do card exiba os dados corretos ao clicar em "Editar".

---

## ✅ Checklist de conclusão

- [ ] Entendeu o conceito de Prop Drilling e por que o Context API ajuda a solucioná-lo.
- [ ] Criou o `TaskCrudContext` com o gerenciamento dos modais/dialogs e item selecionado.
- [ ] Construiu componentes visuais desacoplados (`TaskCard`, `TaskForm`, `Modal` e `ConfirmDialog`).
- [ ] Centralizou a lógica do CRUD nos containers orquestradores.
- [ ] Injetou o `TaskCrudProvider` em torno do `<TasksPage />` na definição de rotas do roteador.

---

## 📚 Referências oficiais deste módulo

| Tema                   | Link                                                                                                                                     |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| React Context API      | [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)                                             |
| useMemo & useCallback  | [useMemo Reference](https://react.dev/reference/react/useMemo) \| [useCallback Reference](https://react.dev/reference/react/useCallback) |
| custom Hooks           | [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)                                               |
| Componentes Puros e UI | [Keeping Components Pure](https://react.dev/learn/keeping-components-pure)                                                               |

---

> 🚀 **Próximo módulo:** Integração com React Query — gerenciamento de estado assíncrono (server state), cache e mutações.

---

# Módulo 5 — Integração com React Query

> 🎯 Objetivo: Aprender a gerenciar estado de dados assíncronos (server state) de forma limpa, segura e performática utilizando o React Query, eliminando as complexidades e os problemas clássicos de `useEffect` + `useState`.

---

## 1. O problema com `useEffect` + `useState` para busca de dados

Na abordagem tradicional, para buscar dados de uma API externa, costumamos criar múltiplos estados locais (`useState`) no componente e disparar a requisição assíncrona dentro de um efeito (`useEffect`).

### O Exemplo Tradicional

```tsx
import { useEffect, useState } from "react";
import axios from "axios";
import type { Task } from "../type";

export function TraditionalListTasks() {
  const [tasks, setTasks] = useState<Task[]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    let isMounted = true; // Necessário para evitar race conditions

    axios
      .get("https://task-manager-vuxy.onrender.com/task")
      .then((response) => {
        if (isMounted) {
          setTasks(response.data);
          setIsLoading(false);
        }
      })
      .catch((err) => {
        if (isMounted) {
          setError(err.message);
          setIsLoading(false);
        }
      });

    return () => {
      isMounted = false;
    };
  }, []);

  if (isLoading) return <p>Carregando tasks...</p>;
  if (error) return <p>Erro ao carregar tasks: {error}</p>;

  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>{task.name}</li>
      ))}
    </ul>
  );
}
```

### Por que essa abordagem é problemática no mundo real?

1. **Boilerplate Excessivo (Verbose):** É preciso criar pelo menos três estados (`data`, `isLoading`, `error`) e tratar blocos `try/catch` ou `.then/.catch` em toda busca de dados.
2. **Race Conditions (Condições de Corrida):** Se o componente for montado, desmontado e montado novamente de forma rápida, requisições antigas que finalizarem por último podem sobrescrever os dados corretos no estado local, a menos que um cleanup extra seja implementado (como o `isMounted` acima).
3. **Ausência de Cache:** Toda vez que o componente for montado na tela (ex: navegando de volta para a rota `/tasks`), uma nova chamada de rede será disparada e o usuário verá um layout em branco (loading spinner) em vez de dados instantâneos.
4. **Acoplamento e Revalidação:** Se você adicionar ou excluir uma tarefa em outro modal/componente, você precisa disparar manualmente uma re-busca de dados ou sincronizar o estado local passando funções via props, o que desfaz o propósito do encapsulamento dos containers.
5. **Falta de funcionalidades avançadas:** Implementar retentativas em caso de falha (`retry`), re-busca automática quando a janela ganha foco (`refetchOnWindowFocus`), ou paginação/scroll infinito com essa abordagem requer centenas de linhas de código customizado.

---

## 2. O que é o React Query e como ele funciona?

O **TanStack Query (React Query)** é uma biblioteca de gerenciamento de estado assíncrono para React. Ele assume o controle do **Server State** (estado do servidor), deixando para o React apenas o **Client State** (estado visual interno como modais abertos, abas ativas, tema, etc.).

### Client State vs Server State

| Característica  | Client State (Ex: Context API, useState)              | Server State (React Query)                                        |
| :-------------- | :---------------------------------------------------- | :---------------------------------------------------------------- |
| **Propriedade** | Pertence 100% à aplicação React e é síncrono.         | Pertence ao banco de dados/servidor remoto e é assíncrono.        |
| **Duração**     | Dura enquanto a aba do navegador estiver aberta.      | Persistente, mas pode ser atualizado por outros usuários ou APIs. |
| **Atualização** | Atualiza instantaneamente ao disparar uma ação na UI. | Requer requisições de rede (HTTP GET/POST/PATCH/DELETE).          |
| **Problemas**   | Compartilhamento entre componentes (prop drilling).   | Cache, concorrência, retentativas, sincronização e invalidação.   |

### Exemplo Básico de Consumo

O React Query funciona principalmente através de duas ferramentas base:

- **`useQuery`**: Usado para buscar dados (operações de leitura, HTTP GET).
- **`useMutation`**: Usado para criar, atualizar ou deletar dados (operações de escrita, HTTP POST/PUT/PATCH/DELETE).

#### O papel do `queryClient`

O **`queryClient`** é a instância central que gerencia o cache e o estado de todas as requisições na nossa aplicação. Ele é configurado na raiz da aplicação e pode ser recuperado dentro de qualquer componente ou custom hook por meio do hook `useQueryClient()`. Com ele, conseguimos acessar o cache, revalidar dados manualmente e atualizar estados remotamente.

#### 1. Buscar Dados (`useQuery`)

```tsx
const { data, isLoading, error } = useQuery({
  queryKey: ["tasks"],
  queryFn: fetchTasks, // Função que retorna uma Promise
});
```

#### 2. Alterar Dados e Invalidar o Cache (`useMutation`)

```tsx
// Recupera a instância global do queryClient para podermos gerenciar o cache
const queryClient = useQueryClient();

const mutation = useMutation({
  mutationFn: createTask,
  onSuccess: () => {
    // Diz para o React Query que os dados da chave "tasks" estão desatualizados.
    // Isso força uma re-busca automática em segundo plano para atualizar a UI de forma reativa!
    queryClient.invalidateQueries({ queryKey: ["tasks"] });
  },
});
```

---

## ✏️ Mexendo no codigo

### 3. Configurando o React Query na aplicação

Primeiro, configuramos o cliente central e envolvemos nossa árvore de componentes com o provedor global do React Query. No nosso projeto, isso está em `AppProviders.tsx`.

**`src/core/providers/AppProviders.tsx`**

```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import type { ReactNode } from "react";

// 1. Instancia o cliente do React Query que gerencia o cache
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      refetchOnWindowFocus: false, // Evita re-buscar dados ao clicar fora/dentro da janela do browser
      retry: 1, // Se a chamada falhar, tenta apenas mais uma vez antes de exibir o erro
    },
  },
});

interface AppProvidersProps {
  children: ReactNode;
}

export function AppProviders({ children }: AppProvidersProps) {
  return (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
  );
}
```

---

### 4. Estrutura de Rotas e Cliente de API (Axios)

Para se comunicar com o backend, configuramos um cliente HTTP usando Axios com a URL base da nossa API hospedada.

**`src/core/consts/apiRoutes.ts`**

```typescript
import axios from "axios";

export const apiClient = axios.create({
  baseURL: "https://task-manager-vuxy.onrender.com",
  timeout: 10000,
});

export const apiRoutes = {
  TASKS: {
    BASE: `/task`,
    BY_ID: (id: string) => `/task/${id}`,
  },
};
```

---

### 5. Criando o Serviço de Tarefas (`TaskService`)

Centralizamos as chamadas HTTP em uma classe de serviço, desacoplando o código de rede do React.

**`src/features/tasks/services/taskService.ts`**

```typescript
import type { AxiosInstance } from "axios";
import type { Task, TaskDTO } from "../type";
import { apiRoutes } from "../../../core/consts/apiRoutes";

export class TaskService {
  constructor(private readonly http: AxiosInstance) {}

  getTasks = async () => {
    const { data } = await this.http.get<Task[]>(apiRoutes.TASKS.BASE);
    return data;
  };

  createTask = async (task: TaskDTO) => {
    const { data } = await this.http.post<Task>(apiRoutes.TASKS.BASE, task);
    return data;
  };

  updateTask = async (id: string, task: TaskDTO) => {
    const { data } = await this.http.patch<Task>(
      apiRoutes.TASKS.BY_ID(id),
      task,
    );
    return data;
  };

  deleteTask = async (id: string) => {
    await this.http.delete(apiRoutes.TASKS.BY_ID(id));
  };

  completeTask = async (id: string) => {
    const { data } = await this.http.patch<Task>(apiRoutes.TASKS.BY_ID(id), {
      completed: true,
    });
    return data;
  };
}
```

E expomos a instância configurada da classe através do barrel de serviços:

**`src/features/tasks/services/index.ts`**

```typescript
import { apiClient } from "../../../core/consts/apiRoutes";
import { TaskService } from "./taskService";

export const taskService = new TaskService(apiClient);
```

---

### 6. Criando Custom Hooks com React Query

Criamos custom hooks específicos para encapsular a lógica de queries e mutations da nossa feature de tarefas. Isso mantém os componentes visuais e os containers limpos e livres de detalhes de implementação do React Query.

### 💻 Buscar Tarefas: `useFetchTasks`

**`src/features/tasks/hooks/useFetchTasks.ts`**

```typescript
import { useQuery } from "@tanstack/react-query";
import type { Task } from "../type";
import { taskService } from "../services";

export function useFetchTasks() {
  return useQuery<Task[]>({
    queryKey: ["tasks"],
    queryFn: taskService.getTasks,
  });
}
```

#### 💻 Criar Tarefa: `useCreateTask`

**`src/features/tasks/hooks/useCreateTask.ts`**

```typescript
import { useMutation, useQueryClient } from "@tanstack/react-query";
import type { TaskDTO } from "../type";
import { taskService } from "../services";

export function useCreateTask() {
  const queryClient = useQueryClient();
  const queryKey = ["tasks"];

  return useMutation({
    mutationFn: (task: TaskDTO) => taskService.createTask(task),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey });
    },
  });
}
```

#### 💻 Editar Tarefa: `useUpdateTask`

**`src/features/tasks/hooks/useUpdateTask.ts`**

```typescript
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { taskService } from "../services";
import type { TaskDTO } from "../type";

export function useUpdateTask() {
  const queryClient = useQueryClient();
  const queryKey = ["tasks"];

  return useMutation({
    mutationFn: ({ id, task }: { id: string; task: TaskDTO }) =>
      taskService.updateTask(id, task),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey });
    },
  });
}
```

#### 💻 Excluir Tarefa: `useDeleteTask`

**`src/features/tasks/hooks/useDeleteTask.ts`**

```typescript
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { taskService } from "../services";

export function useDeleteTask() {
  const queryClient = useQueryClient();
  const queryKey = ["tasks"];

  return useMutation({
    mutationFn: (id: string) => taskService.deleteTask(id),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey });
    },
  });
}
```

#### 💻 Concluir Tarefa: `useCompleteTask`

**`src/features/tasks/hooks/useCompleteTask.ts`**

```typescript
import { useMutation, useQueryClient } from "@tanstack/react-query";
import { taskService } from "../services";

export function useCompleteTask() {
  const queryClient = useQueryClient();
  const queryKey = ["tasks"];

  return useMutation({
    mutationFn: (id: string) => {
      return taskService.completeTask(id);
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey });
    },
  });
}
```

---

### 7. Refatorando os Containers para usar React Query

Com os hooks prontos, substituímos o estado local e os mocks estáticos nos containers orquestradores.

#### 💻 Container de Listagem: `<ListTasks />`

Consome a API de consulta (query) e também a mutação de conclusão, renderizando os estados visuais corretos de carregamento ou erro.

**`src/features/tasks/containers/ListTasks.tsx`**

```tsx
import { useCallback } from "react";
import { TaskCard } from "../components/TaskCard";
import { useTaskCrud } from "../contexts/TaskCrudContext";
import { useFetchTasks } from "../hooks/useFetchTasks";
import { useCompleteTask } from "../hooks/useCompleteTask"; // <- Importa o hook de conclusão
import type { Task } from "../type";

export function ListTasks() {
  const { openDeleteDialog, openEditModal } = useTaskCrud();

  // Consome a Query do React Query para listar as tarefas
  const { data, isLoading, error } = useFetchTasks();

  // Consome a Mutation para concluir a tarefa
  const completeTask = useCompleteTask();

  const handleToggleComplete = useCallback((task: Task) => {
    completeTask.mutate(task.id); // <- Dispara a chamada da mutação de conclusão
  }, []);

  if (isLoading) {
    return <p>Carregando tasks...</p>;
  }

  if (error) {
    return <p>Erro ao carregar tasks: {error.message}</p>;
  }

  return (
    <div className="grid grid-cols-1 gap-4 sm:grid-cols-1 lg:grid-cols-1">
      {data?.map((task) => (
        <TaskCard
          key={task.id}
          task={task}
          onEdit={() => openEditModal(task)}
          onDelete={() => openDeleteDialog(task)}
          onComplete={() => handleToggleComplete(task)}
        />
      ))}
    </div>
  );
}
```

#### 💻 Container de Criação/Edição: `<UpsertTask />`

Consome os hooks de mutação correspondentes e fecha o modal quando a promise da mutação for resolvida.

**`src/features/tasks/containers/UpsertTask.tsx`**

```tsx
import { useCallback } from "react";
import { TaskForm } from "../components/TaskForm";
import { useTaskCrud } from "../contexts/TaskCrudContext";
import { Modal } from "../../../core/components/ui";
import type { TaskDTO } from "../type";
import { useCreateTask } from "../hooks/useCreateTask";
import { useUpdateTask } from "../hooks/useUpdateTask";

export function UpsertTask() {
  const { selectedTask, isFormModalOpen, clearSelection } = useTaskCrud();

  // Consome Mutations do React Query
  const createMutation = useCreateTask();
  const updateMutation = useUpdateTask();

  const handleSubmit = useCallback(
    async (data: TaskDTO) => {
      try {
        if (selectedTask) {
          await updateMutation.mutateAsync({ id: selectedTask.id, task: data });
        } else {
          await createMutation.mutateAsync(data);
        }

        clearSelection();
      } catch {
        alert(
          selectedTask
            ? "Erro ao atualizar a task. Por favor, tente novamente."
            : "Erro ao criar a task. Por favor, tente novamente.",
        );
      }
    },
    [createMutation, clearSelection, selectedTask, updateMutation],
  );

  if (!isFormModalOpen) {
    return null;
  }

  return (
    <Modal
      open={isFormModalOpen}
      title={selectedTask ? "Editar Task" : "Criar Task"}
      description={
        selectedTask
          ? "Edite os detalhes da task abaixo."
          : "Preencha os detalhes da nova task."
      }
      onClose={clearSelection}
    >
      <TaskForm onSubmit={handleSubmit} task={selectedTask} />
    </Modal>
  );
}
```

#### 💻 Container de Remoção: `<DeleteTask />`

Consome a mutação de delete e utiliza a propriedade `isPending` para gerenciar o botão de carregamento do diálogo.

**`src/features/tasks/containers/DeleteTask.tsx`**

```tsx
import { ConfirmDialog } from "../../../core/components/ui";
import { useTaskCrud } from "../contexts/TaskCrudContext";
import { useDeleteTask } from "../hooks/useDeleteTask";

export function DeleteTask() {
  const { isDeleteDialogOpen, selectedTask, clearSelection } = useTaskCrud();

  const deleteMutation = useDeleteTask();

  if (!isDeleteDialogOpen && !selectedTask) {
    return null;
  }

  const handleDelete = async () => {
    if (selectedTask) {
      try {
        await deleteMutation.mutateAsync(selectedTask.id);
      } catch (error) {
        alert("Erro ao excluir a task. Por favor, tente novamente.");
      } finally {
        clearSelection();
      }
    }
  };

  return (
    <ConfirmDialog
      open={isDeleteDialogOpen}
      title="Excluir Task"
      description={`Tem certeza que deseja excluir a task "${selectedTask?.name}"? Esta ação não pode ser desfeita.`}
      confirmLabel="Excluir"
      cancelLabel="Cancelar"
      onConfirm={handleDelete}
      onCancel={clearSelection}
      loading={deleteMutation.isPending}
    />
  );
}
```

---

## ✏️ Melhorias Praticas

Agora que integramos as consultas e as mutações na nossa listagem de tarefas, seu desafio é aprimorar a experiência do usuário adicionando feedbacks de carregamento e controle de erros.

Em seu projeto local, no arquivo **`src/features/tasks/containers/ListTasks.tsx`**:

1. **Desativar botões durante o salvamento:** Utilize a propriedade `completeTask.isPending` para desativar o botão de conclusão do card enquanto o request estiver em andamento.
2. **Adicionar tratamento de erros:** Atualmente se a API de conclusão de tarefas falhar por timeout ou queda de conexão, nada acontece na tela. Ajuste a função `handleToggleComplete` utilizando um bloco `try/catch` (lembre de usar `mutateAsync` em vez de `mutate` para retornar a Promise do request) ou utilize o callback `onError` no hook `useCompleteTask` para disparar um alerta amigável de erro.

---

## ✅ Checklist de conclusão

- [ ] Entendeu a diferença teórica entre Client State e Server State.
- [ ] Compreendeu por que usar `useEffect` para buscar dados traz problemas como race conditions e falta de cache.
- [ ] Configurou o `QueryClient` e `QueryClientProvider` no arquivo global de providers.
- [ ] Entendeu o papel do `queryClient` e opções de revalidação de query (`refetchOnWindowFocus`, `retry`, `invalidateQueries`).
- [ ] Criou e organizou a estrutura de hooks assíncronos (`useFetchTasks`, `useCreateTask`, `useUpdateTask`, `useDeleteTask`, `useCompleteTask`).
- [ ] Substituiu a lista estática simulada por dados reais em tempo de execução vindos da API nos containers de tarefas.
- [ ] Integrou o hook `useCompleteTask` dentro do container `<ListTasks />` e disparou a mutação ao concluir tarefas.

---

## 📚 Referências oficiais deste módulo

| Tema                      | Link                                                                                                                                                 |
| :------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------- |
| TanStack Query Docs       | [tanstack.com/query/latest/docs/framework/react/overview](https://tanstack.com/query/latest/docs/framework/react/overview)                           |
| Caching in React Query    | [tanstack.com/query/latest/docs/framework/react/guides/caching](https://tanstack.com/query/latest/docs/framework/react/guides/caching)               |
| useQuery API Reference    | [tanstack.com/query/latest/docs/framework/react/reference/useQuery](https://tanstack.com/query/latest/docs/framework/react/reference/useQuery)       |
| useMutation API Reference | [tanstack.com/query/latest/docs/framework/react/reference/useMutation](https://tanstack.com/query/latest/docs/framework/react/reference/useMutation) |
| Axios HTTP Client Docs    | [axios-http.com/docs/intro](https://axios-http.com/docs/intro)                                                                                       |
