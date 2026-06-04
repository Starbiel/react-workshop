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
