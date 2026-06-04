# Módulo 2 — Criando o Projeto

> ⏱ Tempo estimado: 20 minutos  
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

Durante o assistente interativo, responda:

```
✔ Project name: meu-projeto
✔ Select a framework: React
✔ Select a variant: Typescript
```

Em seguida, acesse a pasta e instale as dependências base:

```bash
cd meu-projeto
npm install
npm run dev
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

Após instalar o Tailwind, configure o plugin no Vite. Edite o arquivo `vite.config.js`:

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
import type { ReactNode } from "react";

const queryClient = new QueryClient();

interface AppProvidersProps {
  children: ReactNode;
}

export function AppProviders({ children }: AppProvidersProps) {
  return (
    <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>
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

> 🚀 **Próximo módulo:** Componentes na prática — construindo as primeiras telas da aplicação com JSX e props.
