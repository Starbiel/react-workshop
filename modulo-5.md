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
