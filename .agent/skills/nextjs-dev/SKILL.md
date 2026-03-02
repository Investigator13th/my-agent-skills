---
name: Next.js 前端开发规范
description: Next.js (React) 前端项目的组织规范、组件设计、样式约定。使用 TailwindCSS + Shadcn UI 构建现代化界面。当需要开发前端、创建组件、实现页面时使用。适用于全栈项目的前端开发阶段。
---

# Next.js 前端开发规范

本规范适用于基于 **Next.js 14+ App Router** 的前端项目。

---

## 项目结构

```
frontend/
├── app/                    # 页面路由 (App Router)
│   ├── layout.tsx          # 根布局
│   ├── page.tsx            # 首页
│   ├── login/
│   │   └── page.tsx        # 登录页
│   ├── dashboard/
│   │   ├── layout.tsx      # Dashboard 专用布局
│   │   └── page.tsx        # Dashboard 页面
│   └── api/                # API Routes (可选，仅作 BFF 层)
│       └── proxy/
│           └── route.ts    # 代理后端 API
├── components/             # 可复用组件
│   ├── ui/                 # Shadcn UI 组件 (自动生成)
│   ├── layout/             # 布局组件
│   │   ├── Header.tsx
│   │   └── Sidebar.tsx
│   └── dashboard/          # 业务组件
│       ├── StatsCard.tsx
│       └── UserTable.tsx
├── lib/                    # 工具函数
│   ├── utils.ts           # 通用工具 (cn, clsx)
│   ├── api.ts             # API 调用封装
│   └── mock-data.ts       # Mock 数据 (开发阶段)
├── hooks/                  # 自定义 Hooks
│   ├── useAuth.ts
│   └── useDashboardData.ts
├── types/                  # TypeScript 类型定义
│   └── index.ts
├── public/                 # 静态资源
│   ├── images/
│   └── icons/
├── styles/                 # 全局样式
│   └── globals.css
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
└── package.json
```

---

## 命名规范

### 文件命名

- **组件文件**: PascalCase，如 `UserTable.tsx`
- **页面文件**: 使用 Next.js 约定 `page.tsx`
- **工具函数**: camelCase，如 `formatDate.ts`
- **类型文件**: PascalCase，如 `User.types.ts` 或统一 `index.ts`

### 组件命名

```tsx
// ✅ 正确：组件名与文件名一致
export default function UserTable() {
  return <div>...</div>;
}

// ❌ 错误：组件名与文件名不一致
export default function Table() {  // 文件名是 UserTable.tsx
  return <div>...</div>;
}
```

---

## 组件设计原则

### 1. 单一职责

每个组件只负责一件事。

```tsx
// ✅ 正确：职责清晰
function UserAvatar({ src, alt }: { src: string; alt: string }) {
  return <img src={src} alt={alt} className="rounded-full w-10 h-10" />;
}

// ❌ 错误：职责混乱
function UserProfile({ user }: { user: User }) {
  return (
    <div>
      <img src={user.avatar} />
      <p>{user.name}</p>
      <button onClick={handleDelete}>Delete</button>  // 应拆分到父组件
    </div>
  );
}
```

### 2. Props 优先

避免组件内部维护过多状态，通过 props 传递。

```tsx
// ✅ 正确：受控组件
function SearchInput({ value, onChange }: { value: string; onChange: (v: string) => void }) {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
}

// ❌ 错误：内部状态难以共享
function SearchInput() {
  const [value, setValue] = useState('');
  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
}
```

### 3. 组件复用

优先使用 Shadcn UI 组件，需要时进行二次封装。

```tsx
import { Button } from '@/components/ui/button';

// ✅ 正确：基于 Shadcn UI 二次封装
export function SubmitButton({ isLoading, children }: { isLoading: boolean; children: React.ReactNode }) {
  return (
    <Button disabled={isLoading}>
      {isLoading ? 'Loading...' : children}
    </Button>
  );
}
```

---

## 样式规范

### 使用 TailwindCSS

**优先级**：Tailwind > CSS Modules > Inline Styles

```tsx
// ✅ 推荐：使用 Tailwind
<div className="flex items-center gap-4 p-6 bg-white rounded-lg shadow">
  <h1 className="text-2xl font-bold">Title</h1>
</div>

// ⚠️ 可接受：复杂样式使用 CSS Modules
import styles from './Dashboard.module.css';
<div className={styles.complexGrid}>...</div>

// ❌ 避免：内联样式
<div style={{ display: 'flex', padding: '24px' }}>...</div>
```

### 使用 `cn` 工具

```tsx
import { cn } from '@/lib/utils';

function Button({ variant, className }: { variant: 'primary' | 'secondary'; className?: string }) {
  return (
    <button
      className={cn(
        'px-4 py-2 rounded',
        variant === 'primary' && 'bg-blue-500 text-white',
        variant === 'secondary' && 'bg-gray-200 text-black',
        className
      )}
    >
      Click me
    </button>
  );
}
```

---

## 数据获取

### 使用 Server Components (推荐)

```tsx
// app/dashboard/page.tsx
async function getDashboardData() {
  const res = await fetch('http://localhost:8000/api/dashboard/stats', {
    cache: 'no-store',  // 或 'force-cache' | { next: { revalidate: 60 } }
  });
  return res.json();
}

export default async function DashboardPage() {
  const data = await getDashboardData();
  return <div>{data.total_users}</div>;
}
```

### 使用 Client Components (交互时)

```tsx
'use client';

import { useEffect, useState } from 'react';

export default function DashboardClient() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('/api/stats')
      .then((res) => res.json())
      .then(setData);
  }, []);

  if (!data) return <div>Loading...</div>;
  return <div>{data.total_users}</div>;
}
```

---

## 类型定义

### API 响应类型

```tsx
// types/index.ts
export interface User {
  id: string;
  username: string;
  email: string;
  created_at: string;
}

export interface DashboardStats {
  total_users: number;
  active_users_today: number;
  revenue: {
    total: number;
    currency: string;
  };
}
```

### 组件 Props 类型

```tsx
// 简单组件：内联定义
function Greeting({ name }: { name: string }) {
  return <div>Hello, {name}!</div>;
}

// 复杂组件：单独定义
interface UserTableProps {
  users: User[];
  onDelete: (id: string) => void;
  isLoading?: boolean;
}

function UserTable({ users, onDelete, isLoading = false }: UserTableProps) {
  // ...
}
```

---

## 状态管理

对于中小型项目，优先使用 React 内置方案：

1. **URL 状态**：使用 `searchParams`（如筛选条件）
2. **本地状态**：使用 `useState`
3. **共享状态**：使用 React Context
4. **服务器状态**：使用 Server Components 或 `useEffect`

```tsx
// 1. URL 状态
export default function ProductsPage({ searchParams }: { searchParams: { category?: string } }) {
  const category = searchParams.category || 'all';
  return <div>Category: {category}</div>;
}

// 3. 共享状态 (Context)
'use client';

import { createContext, useContext, useState } from 'react';

const AuthContext = createContext<{ user: User | null }>({ user: null });

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  return <AuthContext.Provider value={{ user }}>{children}</AuthContext.Provider>;
}

export function useAuth() {
  return useContext(AuthContext);
}
```

---

## Mock 数据

在后端 API 未就绪时，使用 Mock 数据。

```tsx
// lib/mock-data.ts
import { DashboardStats } from '@/types';

export const mockDashboardStats: DashboardStats = {
  total_users: 1234,
  active_users_today: 567,
  revenue: {
    total: 98765.43,
    currency: 'USD',
  },
};
```

```tsx
// app/dashboard/page.tsx
import { mockDashboardStats } from '@/lib/mock-data';

export default function DashboardPage() {
  // const data = await fetchRealData();  // 后端就绪后替换
  const data = mockDashboardStats;
  return <div>{data.total_users}</div>;
}
```

---

## 错误处理

### 使用 Error Boundary

```tsx
// app/error.tsx (自动捕获错误)
'use client';

export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div className="flex flex-col items-center justify-center h-screen">
      <h2 className="text-2xl font-bold">出错了！</h2>
      <p className="text-gray-600">{error.message}</p>
      <button onClick={reset} className="mt-4 px-4 py-2 bg-blue-500 text-white rounded">
        重试
      </button>
    </div>
  );
}
```

---

## 性能优化

1. **使用 Next.js Image**：
   ```tsx
   import Image from 'next/image';
   <Image src="/logo.png" alt="Logo" width={100} height={100} />
   ```

2. **动态导入**：
   ```tsx
   import dynamic from 'next/dynamic';
   const HeavyComponent = dynamic(() => import('@/components/HeavyComponent'), {
     loading: () => <p>Loading...</p>,
   });
   ```

3. **避免不必要的客户端组件**：
   - 默认使用 Server Components
   - 仅在需要交互时添加 `'use client'`

---

## 最佳实践

1. **文件夹按功能分组**：`components/dashboard/` 而非 `components/DashboardCard.tsx`, `components/DashboardTable.tsx` 散乱放置。
2. **绝对路径**：使用 `@/` 别名（在 `tsconfig.json` 配置）。
3. **组件尺寸**：单个组件不超过 150 行，超过则拆分。
4. **注释规范**：复杂逻辑添加注释，简单组件无需注释。

---

## 常见错误

- ❌ **在 Server Component 中使用 `useState`**
  - ✅ 添加 `'use client'` 或改用 Server Component 模式

- ❌ **全局状态滥用**：所有数据都放 Context
  - ✅ 只在真正需要跨层级共享时使用

- ❌ **忽视 TypeScript**：大量使用 `any`
  - ✅ 严格定义类型，避免运行时错误
