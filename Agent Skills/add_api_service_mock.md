---
name: API Service & Mock Implementation
description: Implements a full vertical slice of data flow: Type -> Mock -> Service -> Hook.
---

# API Service & Mock Implementation Skill

This skill guides you through adding a new API endpoint, from type definition to UI hook, ensuring comprehensive coverage and consistency.

## Usage

When the user asks to "add an API", "create a service", or "connect to backend", follow these steps.

## Steps

1.  **Define Types**:
    - Open `types/index.ts`.
    - Define the entity `interface` (e.g., `Product`).
    - Define `DTO` types for Create/Update operations (e.g., `CreateProductDTO`).
2.  **Implement Mock Handler**:
    - Open `mocks/handlers.ts`.
    - Add a new `http.[method]` handler (GET, POST, PUT, DELETE).
    - Ensure the handler returns realistic mock data matching the defined Type.
    - Simulate network delay if appropriate (`delay(500)`).
3.  **Create Service**:
    - Create/Update `services/[feature].service.ts` or `app/(main)/[feature]/services/[feature].service.ts`.
    - Import `apiClient` from `@/lib/api-client`.
    - Create a class `[Feature]Service` with async methods.
    - Use `apiClient.get<T>()`, `apiClient.post<T>()`, etc.
    - Export a singleton instance: `export const [feature]Service = new [Feature]Service();`.
4.  **Create TanStack Query Hook**:
    - Create/Update `hooks/use[Feature].ts` or `app/(main)/[feature]/hooks/use[Feature].ts`.
    - Define Query Keys constant: `export const [feature]Keys = { ... }`.
    - Implement `useQuery` hook for fetching data.
    - Implement `useMutation` hook for modifying data, ensuring `onSuccess` invalidates relevant query keys.

## Example Code

### Service

```typescript
import apiClient from "@/lib/api-client";
import { Product } from "@/types";

class ProductService {
  async getProducts() {
    const response = await apiClient.get<Product[]>("/products");
    return response.data;
  }
}
export const productService = new ProductService();
```

### Hook

```typescript
import { useQuery } from "@tanstack/react-query";
import { productService } from "../services/product.service";

export const productKeys = {
  all: ["products"] as const,
};

export function useProducts() {
  return useQuery({
    queryKey: productKeys.all,
    queryFn: () => productService.getProducts(),
  });
}
```
