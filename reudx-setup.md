Drafting Redux folder structure documentation:


[2 tools called]


Redux folder structure documentation:

```markdown
# Redux Architecture Documentation

## Overview

This project uses **Redux Toolkit** with **RTK Query** for state management and API calls. The architecture follows a feature-based folder structure that scales well as the application grows.

## Folder Structure

```
redux/
├── features/              # Feature-based Redux modules
│   ├── api/
│   │   └── baseApi.ts      # Base RTK Query API configuration
│   ├── auth/
│   │   ├── authApi.ts      # Auth API endpoints
│   │   └── authSlice.ts    # Auth state slice
│   ├── stores/             # Store management (to be implemented)
│   ├── customers/          # Customer management (to be implemented)
│   ├── invoices/           # Invoice management (to be implemented)
│   └── parcel/             # Parcel management (to be implemented)
├── store/
│   └── index.ts            # Redux store configuration
├── hooks.ts                # Typed Redux hooks
└── ReduxProvider.tsx       # Redux Provider component
```

## Core Components

### 1. Base API (`features/api/baseApi.ts`)

The foundation for all RTK Query API calls. It includes:

- **Base URL Configuration**: Centralized API endpoint
- **Authentication Headers**: Automatically adds Bearer token to requests
- **Token Refresh Logic**: Handles 401 errors and refreshes tokens automatically
- **Re-authentication**: Redirects to login on auth failure

**Key Features:**
- Automatic token injection from Redux state
- Token refresh on 401 errors
- Automatic logout and redirect on refresh failure

### 2. Feature Structure Pattern

Each feature should follow this pattern:

```
feature-name/
├── featureApi.ts    # RTK Query endpoints (API calls)
└── featureSlice.ts  # Redux slice (local state, optional)
```

#### Example: Auth Feature

**`authSlice.ts`** - Manages local auth state:
- Access/refresh tokens
- User information
- Cookie management
- Actions: `setCredentials`, `setTokens`, `setUser`, `logOut`

**`authApi.ts`** - Handles API calls:
- `login` mutation
- `signup` mutation
- `logout` mutation
- `getCurrentUser` query
- `refreshToken` mutation

### 3. Store Configuration (`store/index.ts`)

Central Redux store setup:

```typescript
{
  reducer: {
    auth: authReducer,              // Auth slice reducer
    [baseApi.reducerPath]: baseApi.reducer  // RTK Query reducer
  },
  middleware: [
    ...defaultMiddleware,
    baseApi.middleware               // RTK Query middleware
  ]
}
```

**To add a new feature:**
1. Import the reducer/slice
2. Add it to the `reducer` object
3. If using RTK Query, it's automatically handled via `baseApi`

### 4. Typed Hooks (`hooks.ts`)

Provides TypeScript-typed Redux hooks:

- `useAppDispatch()` - Typed dispatch function
- `useAppSelector()` - Typed selector with RootState

**Usage:**
```typescript
import { useAppDispatch, useAppSelector } from '@/redux/hooks';

const dispatch = useAppDispatch();
const user = useAppSelector((state) => state.auth.user);
```

### 5. Redux Provider (`ReduxProvider.tsx`)

Wraps the application with Redux Provider. Should be added at the root layout level.

## Adding a New Feature

### Step 1: Create Feature API

Create `features/stores/storesApi.ts`:

```typescript
import { baseApi } from "../api/baseApi";

export interface Store {
  id: string;
  name: string;
  address: string;
  // ... other fields
}

export const storesApi = baseApi.injectEndpoints({
  endpoints: (builder) => ({
    getStores: builder.query<Store[], void>({
      query: () => "/stores",
      providesTags: ["Stores"],
    }),
    getStore: builder.query<Store, string>({
      query: (id) => `/stores/${id}`,
      providesTags: (result, error, id) => [{ type: "Stores", id }],
    }),
    createStore: builder.mutation<Store, Partial<Store>>({
      query: (body) => ({
        url: "/stores",
        method: "POST",
        body,
      }),
      invalidatesTags: ["Stores"],
    }),
    updateStore: builder.mutation<Store, { id: string; data: Partial<Store> }>({
      query: ({ id, data }) => ({
        url: `/stores/${id}`,
        method: "PUT",
        body: data,
      }),
      invalidatesTags: (result, error, { id }) => [
        { type: "Stores", id },
        "Stores",
      ],
    }),
    deleteStore: builder.mutation<void, string>({
      query: (id) => ({
        url: `/stores/${id}`,
        method: "DELETE",
      }),
      invalidatesTags: ["Stores"],
    }),
  }),
});

export const {
  useGetStoresQuery,
  useGetStoreQuery,
  useCreateStoreMutation,
  useUpdateStoreMutation,
  useDeleteStoreMutation,
} = storesApi;
```

### Step 2: Add Tags to baseApi (if needed)

Update `features/api/baseApi.ts`:

```typescript
tagTypes: ["User", "Auth", "Stores"], // Add "Stores"
```

### Step 3: Create Feature Slice (if needed)

If you need local state management, create `features/stores/storesSlice.ts`:

```typescript
import { createSlice, PayloadAction } from "@reduxjs/toolkit";

interface StoresState {
  selectedStore: string | null;
  filters: {
    status?: string;
    search?: string;
  };
}

const initialState: StoresState = {
  selectedStore: null,
  filters: {},
};

const storesSlice = createSlice({
  name: "stores",
  initialState,
  reducers: {
    setSelectedStore: (state, action: PayloadAction<string | null>) => {
      state.selectedStore = action.payload;
    },
    setFilters: (state, action: PayloadAction<StoresState["filters"]>) => {
      state.filters = action.payload;
    },
    clearFilters: (state) => {
      state.filters = {};
    },
  },
});

export const { setSelectedStore, setFilters, clearFilters } = storesSlice.actions;
export default storesSlice.reducer;
```

### Step 4: Register Reducer in Store

Update `store/index.ts`:

```typescript
import storesReducer from "../features/stores/storesSlice";

export const store = configureStore({
  reducer: {
    auth: authReducer,
    stores: storesReducer,  // Add this
    [baseApi.reducerPath]: baseApi.reducer,
  },
  // ... rest of config
});
```

### Step 5: Use in Components

```typescript
import { useGetStoresQuery, useCreateStoreMutation } from '@/redux/features/stores/storesApi';
import { useAppSelector, useAppDispatch } from '@/redux/hooks';
import { setSelectedStore } from '@/redux/features/stores/storesSlice';

function MyComponent() {
  const { data: stores, isLoading } = useGetStoresQuery();
  const [createStore] = useCreateStoreMutation();
  const dispatch = useAppDispatch();
  const selectedStore = useAppSelector((state) => state.stores.selectedStore);

  // Use the hooks...
}
```

## Best Practices

### 1. API Endpoints
- ✅ Use RTK Query for all API calls
- ✅ Inject endpoints into `baseApi` using `injectEndpoints`
- ✅ Use `providesTags` and `invalidatesTags` for cache management
- ✅ Define TypeScript interfaces for request/response types

### 2. State Management
- ✅ Use RTK Query for server state (API data)
- ✅ Use Redux slices only for client-side state (UI state, filters, etc.)
- ✅ Keep slices minimal - prefer RTK Query when possible

### 3. File Naming
- ✅ Use camelCase: `storesApi.ts`, `authSlice.ts`
- ✅ Be descriptive: `customerApi.ts` not `api.ts`

### 4. Code Organization
- ✅ One feature per folder
- ✅ Keep related code together (API + slice in same folder)
- ✅ Export hooks from API files for easy imports

### 5. Type Safety
- ✅ Always define TypeScript interfaces for API types
- ✅ Use typed hooks (`useAppDispatch`, `useAppSelector`)
- ✅ Export types for reuse across the app

## RTK Query Cache Management

### Tags System

Tags are used to manage cache invalidation:

```typescript
providesTags: ["Stores"]  // This query provides "Stores" tag
invalidatesTags: ["Stores"]  // This mutation invalidates "Stores" tag
```

**Benefits:**
- Automatic cache updates
- No manual refetching needed
- Efficient data synchronization

### Example Flow:

1. `getStores` provides `["Stores"]` tag
2. `createStore` invalidates `["Stores"]` tag
3. All queries with `["Stores"]` tag automatically refetch

## Authentication Flow

1. **Login**: `authApi.login` → Updates tokens in `authSlice` → Cookies set
2. **API Calls**: `baseApi` automatically adds Bearer token from Redux state
3. **Token Refresh**: On 401 error → Refresh token → Retry original request
4. **Logout**: `authSlice.logOut` → Clears state and cookies → Redirect

## Common Patterns

### Pattern 1: Simple CRUD Feature

```typescript
// Only API, no slice needed
export const itemsApi = baseApi.injectEndpoints({
  endpoints: (builder) => ({
    // CRUD operations
  }),
});
```

### Pattern 2: Feature with UI State

```typescript
// API + Slice for local state
// storesApi.ts - Server state
// storesSlice.ts - UI state (filters, selections, etc.)
```

### Pattern 3: Feature with Complex State

```typescript
// Multiple slices if needed
// storesSlice.ts - Store-related state
// storeFiltersSlice.ts - Filter state (if complex)
```

## Troubleshooting

### Issue: Cache not updating
- Check `invalidatesTags` in mutations
- Verify tags match between queries and mutations

### Issue: Token not being sent
- Check `baseApi.ts` `prepareHeaders` function
- Verify token exists in Redux state: `state.auth.access_token`

### Issue: Type errors
- Ensure `RootState` type is imported correctly
- Use typed hooks: `useAppDispatch`, `useAppSelector`

## Migration Guide

When migrating from existing code:

1. **Identify API calls** → Move to RTK Query endpoints
2. **Identify local state** → Move to Redux slices (if needed)
3. **Update components** → Use RTK Query hooks
4. **Update store** → Register new reducers
5. **Test thoroughly** → Verify cache invalidation works

## Resources

- [Redux Toolkit Documentation](https://redux-toolkit.js.org/)
- [RTK Query Documentation](https://redux-toolkit.js.org/rtk-query/overview)
- [TypeScript with Redux](https://redux-toolkit.js.org/usage/usage-with-typescript)
```

