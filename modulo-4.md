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
  id: number;
  name: string;
  description?: string;
  dueDate?: string;
  priority: TaskPriority;

  completed: boolean;

  createdAt: string;
  updatedAt: string;
};
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
}

export function TaskCard({ task, onEdit, onDelete }: TaskCardProps) {
  const isCompleted = task.completed;

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
          <span className="rounded-full bg-slate-100 px-2 py-1 text-slate-600">
            Prazo: {task.dueDate}
          </span>
        )}
      </div>

      <div className="mt-4 flex justify-end gap-2 border-t border-slate-100 pt-4">
        <button
          type="button"
          onClick={() => onEdit(task)}
          className="rounded-md border border-slate-300 px-3 py-2 text-sm font-medium text-slate-700 transition hover:bg-slate-50"
        >
          Editar
        </button>

        <button
          type="button"
          onClick={() => onDelete(task)}
          className="rounded-md border border-red-200 bg-red-50 px-3 py-2 text-sm font-medium text-red-700 transition hover:bg-red-100"
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
import type { Task, TaskPriority } from "../type";

interface TaskFormProps {
  task?: Task;
  onSubmit: (data: TaskFormData) => void;
}

export type TaskFormData = {
  name: string;
  description: string;
  dueDate: string;
  priority: TaskPriority;
};

export function TaskForm({ task, onSubmit }: TaskFormProps) {
  const [name, setName] = useState(task?.name ?? "");
  const [description, setDescription] = useState(task?.description ?? "");
  const [dueDate, setDueDate] = useState(task?.dueDate ?? "");
  const [priority, setPriority] = useState<TaskPriority>(
    task?.priority ?? "MEDIUM",
  );

  function handleSubmit(event: React.FormEvent<HTMLFormElement>) {
    event.preventDefault();

    onSubmit({
      name,
      description,
      dueDate,
      priority,
    });
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
          value={name}
          onChange={(event) => setName(event.target.value)}
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
          value={description}
          onChange={(event) => setDescription(event.target.value)}
          className="w-full rounded-lg border border-slate-300 px-3 py-2 outline-none transition focus:border-slate-500"
          placeholder="Descreva a tarefa..."
        />
      </div>

      <div>
        <label
          htmlFor="dueDate"
          className="mb-1 block text-sm font-medium text-slate-700"
        >
          Data limite
        </label>

        <input
          id="dueDate"
          type="date"
          value={dueDate}
          onChange={(event) => setDueDate(event.target.value)}
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
          value={priority}
          onChange={(event) => setPriority(event.target.value as TaskPriority)}
          className="w-full rounded-lg border border-slate-300 px-3 py-2 outline-none transition focus:border-slate-500"
        >
          <option value="LOW">Baixa</option>

          <option value="MEDIUM">Média</option>

          <option value="HIGH">Alta</option>
        </select>
      </div>

      <button
        type="submit"
        className="w-full rounded-lg bg-slate-900 px-4 py-2 font-medium text-white transition hover:bg-slate-800"
      >
        {isEditing ? "Atualizar tarefa" : "Criar tarefa"}
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
import { TaskCard } from "../components/TaskCard";
import { useTaskCrud } from "../contexts/TaskCrudContext";
import type { Task } from "../type";

const mookTask: Task[] = [
  {
    id: 1,
    name: "Task 1",
    description: "Descrição da Task 1",
    dueDate: "2024-12-31",
    priority: "MEDIUM",
    completed: false,
    createdAt: "2024-01-01T00:00:00Z",
    updatedAt: "2024-01-01T00:00:00Z",
  },
  {
    id: 2,
    name: "Task 2",
    description: "Descrição da Task 2",
    dueDate: "2024-12-31",
    priority: "HIGH",
    completed: true,
    createdAt: "2024-01-01T00:00:00Z",
    updatedAt: "2024-01-01T00:00:00Z",
  },
];

export function ListTasks() {
  const { openDeleteDialog, openEditModal } = useTaskCrud();

  return (
    <div className="grid grid-cols-1 gap-4 sm:grid-cols-1 lg:grid-cols-1">
      {mookTask.map((task) => (
        <TaskCard
          key={task.id}
          task={task}
          onEdit={() => openEditModal(task)}
          onDelete={() => openDeleteDialog(task)}
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
import { TaskForm, type TaskFormData } from "../components/TaskForm";
import { useTaskCrud } from "../contexts/TaskCrudContext";
import { Modal } from "../../../core/components/ui";

export function UpsertTask() {
  const { selectedTask, isFormModalOpen, clearSelection } = useTaskCrud();

  const handleSubmit = useCallback((data: TaskFormData) => {
    console.log(data);
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
import { ConfirmDialog } from "../../../core/components/ui";
import { useTaskCrud } from "../contexts/TaskCrudContext";

export function DeleteTask() {
  const { isDeleteDialogOpen, selectedTask, clearSelection } = useTaskCrud();

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
            Tasks
          </h1>
          <p className="text-slate-500 text-sm">
            Aplicações escaláveis separam infraestrutura de rede, estado
            assíncrono e interface visual.
          </p>
        </div>
        <div>
          <button
            onClick={openCreateModal}
            className="bg-blue-500 hover:bg-blue-600 text-white font-medium py-2 px-4 rounded-md"
          >
            Create Task
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

| Tema | Link |
| --- | --- |
| React Context API | [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context) |
| useMemo & useCallback | [useMemo Reference](https://react.dev/reference/react/useMemo) \| [useCallback Reference](https://react.dev/reference/react/useCallback) |
| custom Hooks | [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) |
| Componentes Puros e UI | [Keeping Components Pure](https://react.dev/learn/keeping-components-pure) |



