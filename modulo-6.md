# Módulo 6 — Autenticação e Segurança

> 🎯 Objetivo: Compreender como gerenciar sessões de usuários utilizando tokens JWT, proteger rotas privadas com Guards, injetar credenciais automaticamente via interceptadores do Axios e tratar expirações de sessão com React Query.

---

## 1. O Fluxo de Autenticação (JWT)

A autenticação em aplicações web modernas (especialmente SPAs) geralmente utiliza **JSON Web Tokens (JWT)**. Ao contrário do modelo clássico de cookies de sessão mantidos pelo servidor, o JWT é stateless (sem estado no servidor).

O fluxo funciona da seguinte forma:

```
Usuário digita credenciais e clica em Entrar
                    ↓
Browser envia POST /auth/login para o Servidor
                    ↓
Servidor valida e retorna:  { accessToken: "eyJhbG..." }
                    ↓
Browser decodifica o payload do JWT para extrair os dados do usuário (id, email)
                    ↓
Browser armazena o token e os dados derivados no localStorage
                    ↓
Em cada nova chamada à API, o Axios insere o token no Header:
Authorization: Bearer <token>
                    ↓
Se a API retornar erro 401 (Não Autorizado), a sessão expirou:
O app apaga o localStorage e envia o usuário de volta para /login
```

### Anatomia de um JWT

Um JWT é composto por três partes separadas por pontos:

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxMjMiLCJlbWFpbCI6ImFkbWluQGFkbWluLmNvbSJ9.assinatura
|______ Header ______|.|_______________ Payload _______________|.| Signature |
```

- **Header:** Algoritmo de assinatura (ex: HS256) — codificado em Base64.
- **Payload:** Dados do usuário (`sub`, `email`, `iat`, `exp`) — codificado em Base64.
- **Signature:** Assinatura criptográfica gerada pelo servidor — validada apenas no backend.

> 💡 **Insight-chave:** O frontend **não valida** a assinatura do JWT. Ele apenas decodifica o payload (a segunda parte) para extrair informações como o `id` e o `email` do usuário. Quem garante a integridade é o servidor, ao verificar a assinatura em cada requisição protegida.

---

## 2. O Contexto de Autenticação

Para compartilhar os dados do usuário conectado, o token e o status de autenticação por toda a nossa árvore de componentes (Sidebar, páginas e guards), criamos o **`AuthContext`**.

Além disso, para lidar didaticamente com momentos em que a API do backend estiver temporariamente offline, incluímos uma **porta de escape** no serviço de login para simular sucesso com credenciais locais.

### 💻 Passo 1: Tipagem de Autenticação
A API retorna apenas `{ accessToken: "..." }`. Os dados do usuário estão **dentro** do token, e para tipá-los criamos o `JwtPayload`.

**`src/features/auth/type.ts`**
```typescript
export type User = {
  id: string;
  name: string;
  email: string;
};

export type AuthResponse = {
  accessToken: string;
};

export type JwtPayload = {
  sub: string;
  email: string;
  iat?: number;
  exp?: number;
};
```

### 💻 Passo 2: Decodificador de Token JWT
Para extrair o `sub` (id do usuário) e o `email` de dentro do JWT, criamos o utilitário `decodeToken`. Ele faz o seguinte:

1. Separa as 3 partes do token pelo caractere `.`
2. Pega a segunda parte (payload)
3. Converte de Base64 para string JSON usando `atob()`
4. Faz o parse do JSON

**`src/features/auth/utils/decodeToken.ts`**
```typescript
import type { JwtPayload } from "../type";

/**
 * Decodifica o payload de um JSON Web Token (JWT) sem validar a assinatura.
 *
 * Um JWT possui o formato: header.payload.signature
 * Extraímos a segunda parte (payload), decodificamos de Base64 e parseamos o JSON.
 *
 * ⚠️ Importante: essa função NÃO valida a assinatura do token.
 * A validação de integridade é responsabilidade do servidor.
 */
export function decodeToken(token: string): JwtPayload {
  const base64Payload = token.split(".")[1];

  // Corrige caracteres Base64URL (usado em JWTs) para Base64 padrão
  const normalizedPayload = base64Payload
    .replace(/-/g, "+")
    .replace(/_/g, "/");

  const jsonPayload = atob(normalizedPayload);
  return JSON.parse(jsonPayload);
}
```

> 💡 **Por que Base64URL?** O padrão JWT usa uma variante chamada Base64URL que substitui `+` por `-` e `/` por `_` para ser seguro em URLs. Antes de usar `atob()` (que espera Base64 padrão), precisamos reverter essa substituição.

### 💻 Passo 3: O Serviço de Autenticação com Fallback (Porta de Escape)
**`src/features/auth/services/authService.ts`**
```typescript
import type { AxiosInstance } from "axios";
import type { AuthResponse } from "../type";
import { apiRoutes } from "../../../core/consts/apiRoutes";

export class AuthService {
  private readonly http: AxiosInstance;

  constructor(http: AxiosInstance) {
    this.http = http;
  }

  login = async (email: string, password: string): Promise<AuthResponse> => {
    try {
      const { data } = await this.http.post<AuthResponse>(apiRoutes.AUTH.LOGIN, {
        email,
        password,
      });
      return data;
    } catch (error: any) {
      // Porta de escape: Fallback se a API estiver offline ou indisponível (404 / Rede)
      const isApiOffline = !error.response || error.response.status === 404;
      
      if (isApiOffline && email === "admin@admin.com" && password === "admin123") {
        console.warn("API de autenticação offline. Utilizando porta de escape (Mock local).");

        // Monta um JWT mock com payload decodificável (header.payload.signature)
        const header = btoa(JSON.stringify({ alg: "HS256", typ: "JWT" }));
        const payload = btoa(
          JSON.stringify({
            sub: "mock-id-1",
            email: "admin@admin.com",
            iat: Math.floor(Date.now() / 1000),
          })
        );
        const mockToken = `${header}.${payload}.mock-signature`;

        return { accessToken: mockToken };
      }
      throw error;
    }
  };
}
```

> ⚠️ **Nota sobre o Mock:** O fallback agora gera um JWT real decodificável usando `btoa()` para codificar o header e o payload em Base64. Isso garante que o `decodeToken()` funcione de forma idêntica tanto com a API real quanto com a porta de escape.

E a exportação centralizada no barrel de serviços:
**`src/features/auth/services/index.ts`**
```typescript
import { apiClient } from "../../../core/consts/apiRoutes";
import { AuthService } from "./authService";

export const authService = new AuthService(apiClient);
```

### 💻 Passo 4: O Contexto Global do Usuário (`AuthContext`)
O `AuthContext` agora utiliza o `decodeToken` para extrair os dados do usuário diretamente do JWT recebido.

**`src/features/auth/contexts/AuthContext.tsx`**
```tsx
import { createContext, useContext, useState, useEffect, type ReactNode } from "react";
import type { User, AuthResponse } from "../type";
import { decodeToken } from "../utils/decodeToken";

interface AuthContextType {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  login: (data: AuthResponse) => void;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [token, setToken] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const storedToken = localStorage.getItem("@TaskManager:token");
    const storedUser = localStorage.getItem("@TaskManager:user");

    if (storedToken && storedUser) {
      setToken(storedToken);
      setUser(JSON.parse(storedUser));
    }
    setIsLoading(false);
  }, []);

  const login = (data: AuthResponse) => {
    // Decodifica o payload do JWT para extrair os dados do usuário
    const payload = decodeToken(data.accessToken);

    const decodedUser: User = {
      id: payload.sub,
      email: payload.email,
      name: payload.email.split("@")[0], // Deriva o nome a partir do e-mail
    };

    setToken(data.accessToken);
    setUser(decodedUser);
    localStorage.setItem("@TaskManager:token", data.accessToken);
    localStorage.setItem("@TaskManager:user", JSON.stringify(decodedUser));
  };

  const logout = () => {
    setToken(null);
    setUser(null);
    localStorage.removeItem("@TaskManager:token");
    localStorage.removeItem("@TaskManager:user");
  };

  return (
    <AuthContext.Provider
      value={{
        user,
        token,
        isAuthenticated: !!token,
        isLoading,
        login,
        logout,
      }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error("useAuth deve ser utilizado dentro de um AuthProvider");
  }
  return context;
}
```

---

## 3. Protegendo a Aplicação (Guards)

Com as rotas declarativas do React Router, podemos interceptar o fluxo de acesso a determinadas páginas usando um **Auth Guard**.

### 💻 Passo 1: O Componente `<AuthGuard />`
Esse componente atua como uma barreira na renderização das rotas filhas. Se o usuário estiver deslogado, ele renderiza o componente `<Navigate />` que o redireciona imediatamente.

**`src/features/auth/components/AuthGuard.tsx`**
```tsx
import { Navigate, Outlet } from "react-router-dom";
import { useAuth } from "../contexts/AuthContext";

export function AuthGuard() {
  const { isAuthenticated, isLoading } = useAuth();

  // Exibe um loading simples enquanto verifica o localStorage
  if (isLoading) {
    return (
      <div className="flex h-screen items-center justify-center bg-slate-50">
        <p className="text-slate-500 font-medium">Carregando sessão...</p>
      </div>
    );
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  // <Outlet /> renderiza a rota filha correspondente ativa
  return <Outlet />;
}
```

### 💻 Passo 2: Configurando o Roteador
Em nosso roteador, empacotamos o layout principal (`RootLayout`) dentro do `AuthGuard`, protegendo todas as sub-rotas como o `/` e o `/tasks` simultaneamente.

**`src/core/routes/index.tsx`**
```tsx
import { createBrowserRouter } from "react-router-dom";

import { RootLayout } from "../layouts/RootLayout";
import { AuthLayout } from "../layouts/AuthLayout";

import { DashboardPage } from "../../features/dashboard";
import { TasksPage, TaskCrudProvider } from "../../features/tasks";
import { LoginPage, AuthGuard } from "../../features/auth";

export const router = createBrowserRouter([
  {
    // Área Protegida por Login
    element: <AuthGuard />,
    children: [
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
    ],
  },
  {
    // Área Pública de Login
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

## 4. Interceptors (O Middleware do Axios)

Para evitar ter que buscar e inserir manualmente o token JWT no cabeçalho HTTP de cada request Axios, configuramos **interceptadores**.

### Interceptor de Requisição (Request)
Lê o token salvo no `localStorage` e o insere automaticamente na chave `Authorization` em todas as chamadas.

### Interceptor de Resposta (Response)
Observa todas as respostas que voltam do servidor. Se retornar status `401 Unauthorized`, apaga o cache local (token expirou ou foi invalidado) e direciona para a página `/login`.

**`src/core/consts/apiRoutes.ts`**
```typescript
import axios from "axios";

export const apiClient = axios.create({
  baseURL: "https://task-manager-vuxy.onrender.com",
  timeout: 10000,
});

export const apiRoutes = {
  AUTH: {
    LOGIN: "/auth/login",
  },
  TASKS: {
    BASE: `/task`,
    BY_ID: (id: string) => `/task/${id}`,
  },
};

// Interceptor de Requisição (Injeta o Token JWT)
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem("@TaskManager:token");
  if (token && config.headers) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
}, (error) => {
  return Promise.reject(error);
});

// Interceptor de Resposta (Intercepta 401 e desloga)
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    const isUnauthorized = error.response && error.response.status === 401;
    if (isUnauthorized) {
      // Limpa os dados locais e força recarregamento da página para bater no AuthGuard
      localStorage.removeItem("@TaskManager:token");
      localStorage.removeItem("@TaskManager:user");
      window.location.href = "/login";
    }
    return Promise.reject(error);
  }
);
```

---

## 5. React Query + Autenticação

Para realizar a mutação de login de forma didática com tratamento de loading, criamos o custom hook `useLogin`.

**`src/features/auth/hooks/useLogin.ts`**
```typescript
import { useMutation } from "@tanstack/react-query";
import { authService } from "../services";
import { useAuth } from "../contexts/AuthContext";

export function useLogin() {
  const { login } = useAuth();

  return useMutation({
    mutationFn: ({ email, password }: { email: string; password: string }) =>
      authService.login(email, password),
    onSuccess: (data) => {
      login(data);
    },
  });
}
```

### Limpando o cache no Logout
Quando o usuário sai da aplicação (`logout`), é de vital importância apagar os dados armazenados em memória pelo React Query. Se não fizermos isso, outro usuário ao logar no mesmo navegador pode ver as tarefas do usuário anterior salvas em cache por alguns instantes!

Adicionamos a limpeza no botão da **Sidebar**:

**`src/core/components/Sidebar.tsx`**
```tsx
import { NavLink } from "react-router-dom";
import { useAuth } from "../../features/auth";
import { useQueryClient } from "@tanstack/react-query";

function navClass({ isActive }: { isActive: boolean }): string {
  return isActive
    ? "bg-blue-600 rounded px-3 py-2 text-left"
    : "px-3 py-2 hover:bg-gray-700 rounded text-left";
}

export function Sidebar() {
  const { logout, user } = useAuth();
  const queryClient = useQueryClient();

  function handleLogout() {
    logout();
    queryClient.clear(); // Limpa completamente as consultas e dados salvos no cache do React Query!
  }

  return (
    <aside className="w-56 bg-gray-900 text-white flex flex-col p-4 justify-between h-full">
      <div className="flex flex-col gap-2">
        <h1 className="text-lg font-bold mb-4">MyApp</h1>
        <NavLink to="/" end className={navClass}>Dashboard</NavLink>
        <NavLink to="/tasks" className={navClass}>Tasks</NavLink>
      </div>

      <div className="border-t border-gray-800 pt-4 flex flex-col gap-3">
        {user && (
          <div className="px-2">
            <p className="text-xs text-gray-400">Logado como:</p>
            <p className="text-sm font-semibold truncate">{user.name}</p>
          </div>
        )}
        <button
          onClick={handleLogout}
          className="w-full bg-red-600 hover:bg-red-700 text-white text-sm font-medium py-2 px-3 rounded-lg transition cursor-pointer text-left"
        >
          Sair
        </button>
      </div>
    </aside>
  );
}
```

---

## ✏️ Exercício Prático

Agora que a estrutura de autenticação e os interceptadores de segurança estão rodando em sua aplicação, implemente as seguintes melhorias:

1. **Feedback Visual:** No formulário de login (`LoginPage.tsx`), utilize o estado `isPending` fornecido pelo hook `useLogin` para desabilitar o botão de envio e mudar o texto para "Entrando...", evitando cliques duplicados. (Dica: Essa melhoria já foi pré-implementada na estrutura inicial; analise como os inputs se comportam!).
2. **Erro Customizado:** Trate cenários de erro do Axios exibindo alertas visuais na interface caso o usuário insira uma senha inválida ou a conexão caia.
3. **Persistência Segura:** Atualmente estamos decodificando e salvando a informação do usuário mock no `localStorage`. Como desafio extra, adicione um validador de expiração de token no `AuthGuard` para que, caso o token no storage seja inválido ou expirado, a sessão seja invalidada imediatamente na montagem do componente. (Dica: utilize o campo `exp` do `JwtPayload` e compare com `Date.now() / 1000`).

---

## ✅ Checklist de conclusão

- [ ] Entendeu o conceito de autenticação sem estado usando tokens JWT.
- [ ] Compreendeu a anatomia de um JWT (header.payload.signature) e por que apenas o payload é decodificado no frontend.
- [ ] Criou o utilitário `decodeToken` para extrair `sub` e `email` do token.
- [ ] Criou o `AuthContext` e envolveu os provedores da aplicação com o `AuthProvider`.
- [ ] Implementou a rota pública `/login` e a protegeu com redirecionamento automático caso já logado.
- [ ] Construiu o componente `<AuthGuard />` e envelopou as rotas privadas no roteador.
- [ ] Criou interceptores de request no Axios para envio automático do cabeçalho `Authorization: Bearer <token>`.
- [ ] Criou interceptor de response no Axios para remover dados e deslogar o usuário em caso de erro `401`.
- [ ] Limpou o cache do React Query utilizando `queryClient.clear()` no momento do logout.
- [ ] Testou a porta de escape local (Mock local) com as credenciais `admin@admin.com` e `admin123`.

---

## 📚 Referências oficiais deste módulo

| Tema | Link |
| :--- | :--- |
| JWT (JSON Web Token) Introduction | [jwt.io/introduction](https://jwt.io/introduction) |
| Axios Interceptors | [axios-http.com/docs/interceptors](https://axios-http.com/docs/interceptors) |
| React Router Guards / Navigate | [reactrouter.com/en/main/components/navigate](https://reactrouter.com/en/main/components/navigate) |
| React Query Invalidation / Cleansing | [tanstack.com/query/latest/docs/framework/react/reference/QueryClient#queryclientclear](https://tanstack.com/query/latest/docs/framework/react/reference/QueryClient#queryclientclear) |
