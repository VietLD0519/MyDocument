# Hướng Dẫn Tạo Project Frontend Next.js 15 Sử Dụng Vertical Slice Architecture

## 1. Giới thiệu

**Vertical Slice Architecture** là mô hình tổ chức mã nguồn chia nhỏ theo từng feature/business use-case, thay vì chia theo technical layers (components, services, utils, ...). Mỗi slice chứa toàn bộ logic, UI, API call,... liên quan đến 1 tính năng/feature.

Ưu điểm:
- Dễ mở rộng, maintain, tách biệt code của từng feature.
- Dễ onboarding team mới.
- Giảm phụ thuộc giữa các module.

## 2. Khởi tạo dự án Next.js 15

### Bước 1: Tạo project mới

```bash
npx create-next-app@latest my-vertical-slice-app
cd my-vertical-slice-app
```

Chọn các option phù hợp với dự án của bạn (TypeScript, App Router, v.v.).

### Bước 2: Cài đặt các dependency cần thiết

Ví dụ:
```bash
npm install @tanstack/react-query axios
```
(Tùy vào các slice của bạn mà thêm các thư viện như Zustand, Redux Toolkit, Tailwind, ...)

## 3. Cấu trúc thư mục Vertical Slice

Đề xuất cấu trúc như sau:

```
src/
  slices/
    user/
      components/
        UserProfile.tsx
        UserAvatar.tsx
      api/
        user.api.ts
      hooks/
        useUser.ts
      types.ts
      index.ts
    product/
      components/
        ProductList.tsx
        ProductDetail.tsx
      api/
        product.api.ts
      hooks/
        useProduct.ts
      types.ts
      index.ts
  shared/
    components/
      Button.tsx
      Modal.tsx
    utils/
      fetcher.ts
    types/
      ...
  app/
    ...
  pages/ (nếu dùng Pages Router)
```

**Giải thích:**
- `slices/`: Mỗi folder con là một vertical slice (feature) riêng (user, product, order, ...).
- Trong mỗi slice: có components, api, hooks, types,... dành riêng cho feature đó.
- `shared/`: Chứa các thành phần dùng chung toàn ứng dụng.
- `app/` hoặc `pages/`: Routing của Next.js.

## 4. Ví dụ tạo 1 slice "user"

### Tạo thư mục slice
```
src/slices/user/
```

### 1. API: `src/slices/user/api/user.api.ts`
```typescript
import axios from 'axios';

export const fetchUser = (id: string) => axios.get(`/api/users/${id}`);
```

### 2. Hook: `src/slices/user/hooks/useUser.ts`
```typescript
import { useQuery } from '@tanstack/react-query';
import { fetchUser } from '../api/user.api';

export function useUser(userId: string) {
  return useQuery(['user', userId], () => fetchUser(userId).then(res => res.data));
}
```

### 3. Component: `src/slices/user/components/UserProfile.tsx`
```tsx
import { useUser } from '../hooks/useUser';

export default function UserProfile({ userId }: { userId: string }) {
  const { data, isLoading, error } = useUser(userId);

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading user</div>;

  return (
    <div>
      <h2>{data.name}</h2>
      <p>Email: {data.email}</p>
    </div>
  );
}
```

### 4. Types: `src/slices/user/types.ts`
```typescript
export interface User {
  id: string;
  name: string;
  email: string;
}
```

### 5. Index file: `src/slices/user/index.ts`
```typescript
export * from './components/UserProfile';
export * from './hooks/useUser';
export * from './api/user.api';
export * from './types';
```

## 5. Sử dụng slice trong trang

```tsx
// src/app/page.tsx hoặc src/pages/index.tsx

import UserProfile from '@/slices/user/components/UserProfile';

export default function HomePage() {
  return (
    <main>
      <UserProfile userId="1" />
    </main>
  );
}
```

## 6. Lưu ý & Best Practices

- Luôn tổ chức code theo từng feature, tránh import chéo giữa các slice.
- Nếu cần shared component, đưa vào `shared/` và import từ đó.
- Đặt test file cùng slice (`components/UserProfile.test.tsx`).
- Đặt các logic đặc trưng của slice trong slice, tránh để ở level global.
- Khi mở rộng, chỉ cần tạo thêm slice mới.

---

**Tài liệu tham khảo:**
- [Vertical Slice Architecture Example](https://khalilstemmler.com/articles/vertical-slice-architecture/)
- [Next.js Documentation](https://nextjs.org/docs)