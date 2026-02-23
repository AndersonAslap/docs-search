# Configurando projeto em react 

# 🚀 Stack escolhida

* **React** (Vite)
* **TypeScript**
* **React Router DOM**
* **Tailwind CSS**
* **shadcn/ui**
* **ESLint + Prettier**
* **Alias de paths**
* **Estrutura escalável de pastas**

---

## 1️⃣ Criando o projeto React + TypeScript (Vite)

Vamos usar **Vite**, que hoje é o padrão para projetos React modernos.

```bash
npm create vite@latest my-app
```

Escolha:

```text
✔ Project name: my-app
✔ Select a framework: React
✔ Select a variant: TypeScript
```

Entre no projeto:

```bash
cd my-app
npm install
npm run dev
```

👉 Neste ponto, o React + TypeScript já está funcionando.

---

## 2️⃣ Limpando o projeto inicial

Remova arquivos desnecessários:

```bash
src/App.css
src/assets/
```

Edite `main.tsx`:

```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

Edite `App.tsx`:

```tsx
export default function App() {
  return <h1 className="text-2xl font-bold">Hello React</h1>;
}
```

---

## 3️⃣ Instalando e configurando Tailwind CSS

### Instalação

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### Configurando `tailwind.config.ts`

```ts
import type { Config } from "tailwindcss";

const config: Config = {
  content: ["./index.html", "./src/**/*.{ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};

export default config;
```

### Ajustando `index.css`

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Teste no `App.tsx`:

```tsx
<h1 className="text-3xl font-bold text-blue-500">
  Tailwind funcionando 🚀
</h1>
```

---

## 4️⃣ Instalando React Router DOM

```bash
npm install react-router-dom
```

### Estrutura inicial de rotas

Crie a pasta:

```bash
src/routes
```

`src/routes/index.tsx`:

```tsx
import { createBrowserRouter } from "react-router-dom";
import Home from "@/pages/Home";

export const router = createBrowserRouter([
  {
    path: "/",
    element: <Home />,
  },
]);
```

Crie a página:

```bash
src/pages/Home.tsx
```

```tsx
export default function Home() {
  return <h1 className="text-xl">Home Page</h1>;
}
```

Atualize `main.tsx`:

```tsx
import { RouterProvider } from "react-router-dom";
import { router } from "./routes";

<RouterProvider router={router} />
```

---

## 5️⃣ Configurando alias de paths (`@/`)

### Atualize `tsconfig.json`

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

### Atualize `vite.config.ts`

```ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```

Agora você pode importar assim:

```ts
import Home from "@/pages/Home";
```

---

## 6️⃣ Instalando e configurando shadcn/ui

### Instale dependências base

```bash
npm install -D tailwindcss-animate
npm install clsx class-variance-authority
npm install lucide-react
```

### Inicializando o shadcn

```bash
npx shadcn-ui@latest init
```

Escolha:

```text
✔ TypeScript: yes
✔ Tailwind CSS: yes
✔ src directory: yes
✔ Alias: @
✔ CSS file: src/index.css
✔ Theme: default
```

### Adicionando um componente

```bash
npx shadcn-ui@latest add button
```

Uso:

```tsx
import { Button } from "@/components/ui/button";

<Button>Clique aqui</Button>
```

---

## 7️⃣ Estrutura de pastas recomendada

```text
src/
├── components/
│   └── ui/           # shadcn
├── pages/
├── routes/
├── hooks/
├── services/
├── layouts/
├── lib/
│   └── utils.ts
├── styles/
├── App.tsx
├── main.tsx
```

👉 Essa estrutura escala bem para projetos médios e grandes.

---

## 8️⃣ ESLint + Prettier (opcional, mas recomendado)

```bash
npm install -D eslint prettier eslint-config-prettier eslint-plugin-react eslint-plugin-react-hooks
```

Configuração garante:

* Código padronizado
* Menos bugs
* Melhor colaboração em times

(Se quiser, posso montar a config completa depois.)

---

## 9️⃣ Boas práticas iniciais

✔ Use **componentes pequenos**
✔ Centralize chamadas HTTP em `services/`
✔ Crie **layouts** para páginas autenticadas
✔ Evite lógica de negócio em componentes
✔ Use Tailwind + shadcn como **design system**, não misture libs aleatórias
