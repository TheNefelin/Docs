# Astro 5 - Skill Completo para Proyectos Nuevos

> Guía completa para desarrollo moderno con Astro 5.
> Transversal, sin dependencias de proyectos específicos.

> **⚠️ Importante:** Usar **pnpm** en lugar de npm/yarn. npm tiene vulnerabilidades graves de seguridad y no es seguro para producción.

---

## ¿Por qué pnpm?

| Aspecto | npm | pnpm |
|---------|-----|------|
| **Seguridad** | Vulnerabilidades | Protegido contra dependency confusion |
| **Espacio** | Copias múltiples | Disco virtual compartido |
| **Velocidad** | Lento | 3x más rápido |
| **Estricto** | Permite conflictos | Bloquea conflictos de versiones |

---

## Tabla de Contenidos

1. [Setup y Configuración](#1-setup-y-configuración)
2. [Estructura del Proyecto](#2-estructura-del-proyecto)
3. [Componentes Astro](#3-componentes-astro)
4. [Pages y Routing](#4-pages-y-routing)
5. [Integraciones UI](#5-integraciones-ui)
6. [Estado Compartido](#6-estado-compartido)
7. [API Routes y SSR](#7-api-routes-y-ssr)
8. [Styling](#8-styling)
9. [Data Fetching](#9-data-fetching)
10. [Islands Architecture](#10-islands-architecture)
11. [Markdown y Content Collections](#11-markdown-y-content-collections)
12. [Testing](#12-testing)
13. [Deployment](#13-deployment)
14. [Checklist de Código](#14-checklist-de-código)

---

## 1. Setup y Configuración

### 1.1 Crear Proyecto Astro

```bash
# Instalar pnpm si no lo tienes
npm install -g pnpm

# Crear proyecto Astro
pnpm create astro@latest my-project

# O con opciones automáticas
pnpm create astro@latest my-project --template minimal --install --no-git --typescript strict

# Agregar pnpm a proyecto existente
pnpm add astro

# Scripts en package.json
{
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview",
    "check": "astro check",
    "sync": "astro sync"
  }
}
```

### 1.2 Configuración Principal

```typescript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import tailwind from '@astrojs/tailwind';
import react from '@astrojs/react';
import node from '@astrojs/node';

export default defineConfig({
  output: 'static', // 'static', 'server', 'hybrid'
  integrations: [
    tailwind(),
    react()
  ],
  output: 'hybrid',
  adapter: node({
    mode: 'standalone'
  }),
  vite: {
    optimizeDeps: {
      exclude: ['@libsql/client']
    }
  }
});
```

### 1.3 tsconfig.json

```json
{
  "extends": "astro/tsconfigs/strict",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@layouts/*": ["src/layouts/*"],
      "@lib/*": ["src/lib/*"]
    },
    "strictNullChecks": true
  }
}
```

---

## 2. Estructura del Proyecto

```
my-project/
├── src/
│   ├── components/
│   │   ├── BaseHead.astro
│   │   ├── Header.astro
│   │   ├── Footer.astro
│   │   ├── Card.astro
│   │   └── react/           # Componentes React
│   │       ├── Counter.tsx
│   │       └── Modal.tsx
│   ├── layouts/
│   │   ├── BaseLayout.astro
│   │   └── BlogPost.astro
│   ├── pages/
│   │   ├── index.astro
│   │   ├── about.astro
│   │   ├── blog/
│   │   │   ├── index.astro
│   │   │   └── [slug].astro
│   │   └── api/
│   │       └── users.ts     # API routes
│   ├── lib/
│   │   ├── db.ts           # Database client
│   │   └── auth.ts         # Auth utilities
│   ├── content/
│   │   ├── config.ts       # Content collections config
│   │   └── blog/           # Markdown/MDX files
│   │       ├── post-1.md
│   │       └── post-2.mdx
│   ├── styles/
│   │   └── global.css
│   └── env.d.ts            # Type definitions
├── public/
│   ├── favicon.svg
│   └── images/
├── astro.config.mjs
├── tsconfig.json
└── package.json
```

---

## 3. Componentes Astro

### 3.1 Componente Básico

```astro
---
// src/components/Button.astro
interface Props {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  class?: string;
}

const {
  variant = 'primary',
  size = 'md',
  disabled = false,
  class: className = ''
} = Astro.props;

const variants = {
  primary: 'bg-blue-600 hover:bg-blue-700 text-white',
  secondary: 'bg-gray-200 hover:bg-gray-300 text-gray-900'
};

const sizes = {
  sm: 'px-3 py-1.5 text-sm',
  md: 'px-4 py-2',
  lg: 'px-6 py-3 text-lg'
};
---

<button
  class:list={[
    'rounded-md font-medium transition-colors disabled:opacity-50',
    variants[variant],
    sizes[size],
    className
  ]}
  disabled={disabled}
>
  <slot />
</button>
```

### 3.2 Componente con Slots

```astro
---
// src/components/Card.astro
interface Props {
  title?: string;
  footer?: boolean;
}

const { title, footer = false } = Astro.props;
---

<article class="card">
  {title && <h3 class="card-title">{title}</h3>}
  <div class="card-body">
    <slot />
  </div>
  {footer && (
    <div class="card-footer">
      <slot name="footer" />
    </div>
  )}
</article>

<style>
  .card {
    border: 1px solid #e5e7eb;
    border-radius: 0.5rem;
    overflow: hidden;
  }
  .card-title {
    padding: 1rem;
    font-weight: 600;
    border-bottom: 1px solid #e5e7eb;
  }
  .card-body {
    padding: 1rem;
  }
  .card-footer {
    padding: 1rem;
    background: #f9fafb;
    border-top: 1px solid #e5e7eb;
  }
</style>
```

### 3.3 Props con Tipos

```astro
---
// src/components/UserCard.astro
import type { User } from '@/types';

interface Props {
  user: User;
  showAvatar?: boolean;
}

const { user, showAvatar = true } = Astro.props;
---

<div class="user-card">
  {showAvatar && (
    <img
      src={user.avatarUrl}
      alt={user.name}
      class="avatar"
      loading="lazy"
    />
  )}
  <h3>{user.name}</h3>
  <p>{user.email}</p>
</div>
```

---

## 4. Pages y Routing

### 4.1 Páginas Básicas

```astro
---
// src/pages/index.astro
import BaseLayout from '@/layouts/BaseLayout.astro';
import { getFeaturedPosts } from '@/lib/posts';

const posts = await getFeaturedPosts();
const title = 'Mi Blog';
---

<BaseLayout title={title}>
  <h1>Bienvenido</h1>
  <ul>
    {posts.map(post => (
      <li>
        <a href={`/blog/${post.slug}`}>{post.title}</a>
      </li>
    ))}
  </ul>
</BaseLayout>
```

### 4.2 Rutas Dinámicas

```astro
---
// src/pages/blog/[slug].astro
import { getCollection } from 'astro:content';
import BlogPostLayout from '@/layouts/BlogPost.astro';
import type { StaticPaths } from 'astro';

export async function getStaticPaths(): Promise<StaticPaths> {
  const posts = await getCollection('blog');
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { post }
  }));
}

const { post } = Astro.props;
const { Content } = await post.render();
---

<BlogPostLayout title={post.data.title} description={post.data.description}>
  <article>
    <h1>{post.data.title}</h1>
    <time datetime={post.data.pubDate.toISOString()}>
      {post.data.pubDate.toLocaleDateString()}
    </time>
    <Content />
  </article>
</BlogPostLayout>
```

### 4.3 Routing con API

```typescript
// src/pages/api/users.ts
import type { APIRoute } from 'astro';
import { getUsers } from '@/lib/db';

export const GET: APIRoute = async () => {
  const users = await getUsers();
  return new Response(JSON.stringify(users), {
    status: 200,
    headers: {
      'Content-Type': 'application/json'
    }
  });
};

export const POST: APIRoute = async ({ request }) => {
  const data = await request.json();
  const user = await createUser(data);
  return new Response(JSON.stringify(user), { status: 201 });
};
```

---

## 5. Integraciones UI

### 5.1 Agregar Integraciones

```bash
# React
pnpm add @astrojs/react react react-dom

# Vue
pnpm add @astrojs/vue vue

# Svelte
pnpm add @astrojs/svelte svelte

# Preact
pnpm add @astrojs/preact preact

# Solid
pnpm add @astrojs/solid-js solid-js
```

### 5.2 Componentes en Astro

```astro
---
// src/pages/index.astro
import ReactCounter from '@/components/react/Counter.vue';
import VueModal from '@/components/vue/Modal.vue';
import SvelteComponent from '@/components/svelte/Component.svelte';
---

<!-- Client:only - solo cliente -->
<ReactCounter client:load />

<!-- Client:visible - cuando sea visible -->
<ReactCounter client:visible />

<!-- Client:idle - cuando el navegador esté idle -->
<ReactCounter client:idle />

<!-- Client:media - cuando coincida media query -->
<ReactCounter client:media="(min-width: 768px)" />
```

### 5.3 Configuración Multiple

```typescript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import vue from '@astrojs/vue';
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  integrations: [
    react(),
    vue(),
    tailwind()
  ]
});
```

---

## 6. Estado Compartido

### 6.1 Nano Stores

```bash
pnpm add nanostores @nanostores/react @nanores/react
```

```typescript
// src/stores/cartStore.ts
import { atom, map } from 'nanostores';

export type CartItem = {
  id: string;
  name: string;
  price: number;
  quantity: number;
};

export const $cart = map<Record<string, CartItem>>({});
export const $cartCount = atom(0);

export function addToCart(item: CartItem) {
  const existing = $cart.get()[item.id];
  if (existing) {
    $cart.setKey(item.id, {
      ...existing,
      quantity: existing.quantity + 1
    });
  } else {
    $cart.setKey(item.id, item);
  }
  updateCount();
}

export function removeFromCart(id: string) {
  const current = $cart.get();
  const { [id]: _, ...rest } = current;
  $cart.set(rest);
  updateCount();
}

function updateCount() {
  const items = Object.values($cart.get());
  $cartCount.set(items.reduce((sum, item) => sum + item.quantity, 0));
}
```

### 6.2 Uso en Componente

```tsx
// src/components/CartButton.tsx
import { useStore } from '@nanostores/react';
import { $cartCount } from '@/stores/cartStore';

export function CartButton() {
  const count = useStore($cartCount);
  return <button>Cart ({count})</button>;
}
```

---

## 7. API Routes y SSR

### 7.1 Configuración SSR

```typescript
// astro.config.mjs
import node from '@astrojs/node';

export default defineConfig({
  output: 'server',
  adapter: node({
    mode: 'standalone'
  })
});
```

### 7.2 endpoints

```typescript
// src/pages/api/auth/login.ts
import type { APIRoute } from 'astro';
import { verifyPassword } from '@/lib/auth';
import { createSession } from '@/lib/session';

export const POST: APIRoute = async ({ request, cookies }) => {
  const body = await request.json();
  const { email, password } = body;

  const user = await findUserByEmail(email);
  if (!user || !(await verifyPassword(password, user.password))) {
    return new Response(JSON.stringify({ error: 'Invalid credentials' }), {
      status: 401
    });
  }

  const session = await createSession(user.id);
  cookies.set('session', session.token, {
    path: '/',
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 60 * 60 * 24 * 7
  });

  return new Response(JSON.stringify({ user: { id: user.id, email: user.email } }));
};
```

### 7.3 Server Islands

```astro
---
// src/pages/dashboard.astro
import UserGreeting from '@/components/UserGreeting.astro';
---

<html>
  <body>
    <h1>Dashboard</h1>
    <!-- Server component - renders on server -->
    <UserGreeting />
  </body>
</html>
```

---

## 8. Styling

### 8.1 Tailwind CSS

```bash
pnpm add @astrojs/tailwind tailwindcss
```

```typescript
// astro.config.mjs
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  integrations: [tailwind()]
});
```

```astro
---
// src/pages/index.astro
---

<div class="container mx-auto px-4 py-8">
  <h1 class="text-3xl font-bold text-gray-900">
    Hello World
  </h1>
</div>
```

### 8.2 CSS Modules

```astro
---
// src/components/StyledComponent.astro
import styles from './StyledComponent.module.css';
---

<div class={styles.container}>
  <p class={styles.text}>Styled content</p>
</div>
```

### 8.3 Estilos Globales

```css
/* src/styles/global.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --color-primary: #3b82f6;
}

html {
  scroll-behavior: smooth;
}

body {
  @apply antialiased;
}
```

---

## 9. Data Fetching

### 9.1 Fetch en Server

```astro
---
// src/pages/index.astro
const response = await fetch('https://api.example.com/data');
const data = await response.json();
---

<pre>{JSON.stringify(data, null, 2)}</pre>
```

### 9.2 Fetch en Componente

```tsx
// src/components/DataFetcher.tsx
import { useEffect, useState } from 'react';

export function DataFetcher({ url }: { url: string }) {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch(url).then(res => res.json()).then(setData);
  }, [url]);

  return <pre>{JSON.stringify(data, null, 2)}</pre>;
}
```

### 9.3 Fetch con Loading State

```tsx
// src/components/UserList.tsx
import { useState, useEffect } from 'react';

export function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetch('/api/users')
      .then(res => {
        if (!res.ok) throw new Error('Failed to fetch');
        return res.json();
      })
      .then(setUsers)
      .catch(err => setError(err.message))
      .finally(() => setLoading(false));
  }, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

---

## 10. Islands Architecture

### 10.1 Concepto

Islands = componentes interactivos aislados. El resto de la página es HTML estático.

```astro
---
// src/pages/index.astro
import StaticHeader from '@/components/StaticHeader.astro';
import InteractiveCounter from '@/components/react/Counter.tsx';
---

<StaticHeader />
<!-- Solo este componente envía JS al cliente -->
<InteractiveCounter client:load />
```

### 10.2 Tipos de Hydration

| Directive | Cuándo carga |
|-----------|--------------|
| `client:load` | Inmediatamente |
| `client:idle` | Cuando CPU libre |
| `client:visible` | Cuando visible viewport |
| `client:media` | Cuando media query coincide |
| `client:only` | Solo cliente (no SSR) |

### 10.3 Patterns

```astro
---
// Evitar prop drilling - pasar solo lo necesario
const { user } = Astro.props;
---

<InteractiveComponent
  client:visible
  userId={user.id}
  userName={user.name}
/>
```

---

## 11. Markdown y Content Collections

### 11.1 Configurar Content Collections

```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const blog = defineCollection({
  type: 'content', // 'content' (markdown) o 'data' (JSON/YAML)
  schema: z.object({
    title: z.string(),
    description: z.string(),
    pubDate: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    heroImage: z.string().optional(),
    tags: z.array(z.string()).default([]),
    draft: z.boolean().default(false)
  })
});

const products = defineCollection({
  type: 'data',
  schema: z.object({
    name: z.string(),
    price: z.number(),
    category: z.enum(['electronics', 'clothing', 'books']),
    inStock: z.boolean()
  })
});

export const collections = { blog, products };
```

### 11.2 Usar Content Collections

```astro
---
// src/pages/blog/index.astro
import { getCollection } from 'astro:content';

const posts = await getCollection('blog', ({ data }) => {
  return import.meta.env.PROD ? data.draft !== true : true;
});

const sortedPosts = posts.sort(
  (a, b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf()
);
---

<ul>
  {sortedPosts.map(post => (
    <li>
      <a href={`/blog/${post.slug}`}>
        {post.data.title}
      </a>
    </li>
  ))}
</ul>
```

### 11.3 MDX

```bash
pnpm add @astrojs/mdx
```

```astro
---
// src/pages/mdx-page.astro
import { Content } from '../content/blog/my-post.mdx';
---

<Content />
```

---

## 12. Testing

### 12.1 Vitest + Testing Library

```bash
pnpm add -D vitest @testing-library/dom @testing-library/user-event jsdom
```

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import astro from 'astro/vitest';

export default defineConfig({
  plugins: [astro()],
  test: {
    environment: 'jsdom',
    globals: true
  }
});
```

### 12.2 Tests de Componentes

```tsx
// src/components/__tests__/Counter.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { Counter } from '../Counter';

describe('Counter', () => {
  it('renders initial count', () => {
    render(<Counter initialValue={0} />);
    expect(screen.getByText('0')).toBeInTheDocument();
  });

  it('increments count', () => {
    render(<Counter initialValue={0} />);
    fireEvent.click(screen.getByRole('button', { name: 'Increment' }));
    expect(screen.getByText('1')).toBeInTheDocument();
  });
});
```

### 12.3 Tests de Integración

```astro
---
// src/pages/__tests__/index.test.ts
import { describe, it, expect } from 'vitest';
import { getStaticPaths, getStaticProps } from '../index.astro';

describe('Home Page', () => {
  it('generates correct paths', async () => {
    const result = await getStaticPaths();
    expect(result).toBeDefined();
  });
});
```

---

## 13. Deployment

### 13.1 Build

```bash
# Build para producción
pnpm build

# Preview local
pnpm preview
```

### 13.2 Adaptadores

```bash
# Node
pnpm add @astrojs/node

# Vercel
pnpm add @astrojs/vercel

# Netlify
pnpm add @astrojs/netlify

# Cloudflare
pnpm add @astrojs/cloudflare
```

### 13.3 Vercel

```typescript
// astro.config.mjs
import vercel from '@astrojs/vercel/serverless';

export default defineConfig({
  output: 'server',
  adapter: vercel()
});
```

### 13.4 Docker

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder
RUN corepack enable
WORKDIR /app
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile --prod
COPY . .
RUN pnpm build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package.json ./
RUN pnpm add pm2
EXPOSE 3000
CMD ["node", "dist/server/entry.mjs"]
```

---

## 14. Checklist de Código

### 14.1 Performance

- [ ] Usar `client:visible` o `client:idle` en componentes no críticos
- [ ] Optimizar imágenes con `<Image />` de Astro
- [ ] Minimizar islands - solo lo necesario
- [ ] Lazy loading de componentes pesados

### 14.2 SEO

- [ ] Meta tags en `<BaseHead>`
- [ ] Semantic HTML (`<main>`, `<article>`, `<nav>`)
- [ ] Sitemap configurado
- [ ] Robots.txt configurado

### 14.3 Accesibilidad

- [ ] Labels en formularios
- [ ] ARIA labels donde sea necesario
- [ ] Keyboard navigation
- [ ] Alt text en imágenes

### 14.4 Seguridad

- [ ] Headers de seguridad (CSP, etc.)
- [ ] Sanitizar inputs
- [ ] No exponer secrets en cliente
- [ ] Use `Astro.cookies` con `httpOnly`

### 14.5 TypeScript

- [ ] Strict mode enabled
- [ ] Tipos para props de componentes
- [ ] Interfaces para datos de API
- [ ] Evitar `any`

---

## Recursos

- [Astro Docs](https://docs.astro.build)
- [Astro Integrations](https://astro.build/integrations/)
- [Astro Blog Tutorial](https://docs.astro.build/en/tutorial/)
- [Nano Stores](https://github.com/nanostores/nanostores)