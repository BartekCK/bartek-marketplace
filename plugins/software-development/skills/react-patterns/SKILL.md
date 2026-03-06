---
name: react-patterns
description: React component patterns, hooks, and state management best practices for TypeScript. Loaded by frontend-software-developer-agent when building UI components, managing state, or structuring React applications.
user-invocable: false
disable-model-invocation: true
---

# React Patterns Cheat Sheet

## Component Composition

### Compound Components

Share implicit state between parent and children via context.

```tsx
interface TabsContextType { active: string; setActive: (id: string) => void }
const TabsContext = createContext<TabsContextType | null>(null);

function Tabs({ children, defaultTab }: { children: ReactNode; defaultTab: string }) {
  const [active, setActive] = useState(defaultTab);
  return <TabsContext.Provider value={{ active, setActive }}>{children}</TabsContext.Provider>;
}

function TabPanel({ id, children }: { id: string; children: ReactNode }) {
  const ctx = useContext(TabsContext);
  if (!ctx) throw new Error("TabPanel must be inside Tabs");
  return ctx.active === id ? <>{children}</> : null;
}

// Usage: <Tabs defaultTab="a"><TabPanel id="a">Content</TabPanel></Tabs>
```

### Render Props

Delegate rendering to the consumer when you need flexible output.

```tsx
interface ListProps<T> { items: T[]; renderItem: (item: T, index: number) => ReactNode }
function List<T>({ items, renderItem }: ListProps<T>) {
  return <ul>{items.map((item, i) => <li key={i}>{renderItem(item, i)}</li>)}</ul>;
}
```

### HOC (use sparingly — prefer hooks)

```tsx
function withAuth<P extends object>(Component: ComponentType<P>) {
  return function AuthWrapper(props: P) {
    const { user } = useAuth();
    if (!user) return <Navigate to="/login" />;
    return <Component {...props} />;
  };
}
```

## Custom Hooks

### Data fetching hook

```tsx
function useApi<T>(url: string): { data: T | null; loading: boolean; error: string | null } {
  const [state, setState] = useState<{ data: T | null; loading: boolean; error: string | null }>({
    data: null, loading: true, error: null,
  });

  useEffect(() => {
    const controller = new AbortController();
    fetch(url, { signal: controller.signal })
      .then((res) => res.json() as Promise<T>)
      .then((data) => setState({ data, loading: false, error: null }))
      .catch((err) => { if (!controller.signal.aborted) setState({ data: null, loading: false, error: err.message }); });
    return () => controller.abort();
  }, [url]);

  return state;
}
```

### Debounced value hook

```tsx
function useDebounce<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => { const id = setTimeout(() => setDebounced(value), delay); return () => clearTimeout(id); }, [value, delay]);
  return debounced;
}
```

## State Management — When to Use What

| Approach | Use when |
|---|---|
| `useState` | Simple local state, toggles, form fields |
| `useReducer` | Complex state with multiple sub-values or transitions |
| Context | Shared state across a subtree (theme, auth, locale) |
| React Query / SWR | Server state — caching, revalidation, pagination |
| Zustand / Jotai | Global client state that many components read/write |

**Rule:** Server state belongs in React Query, not in global stores.

## Controlled vs Uncontrolled

**Controlled** — React state is the source of truth. Use when you need validation, formatting, or conditional logic on every change.

```tsx
function ControlledInput({ value, onChange }: { value: string; onChange: (v: string) => void }) {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
}
```

**Uncontrolled** — DOM is the source of truth. Use for simple forms or when integrating non-React libraries.

```tsx
function UncontrolledForm({ onSubmit }: { onSubmit: (data: FormData) => void }) {
  const ref = useRef<HTMLFormElement>(null);
  return <form ref={ref} onSubmit={(e) => { e.preventDefault(); onSubmit(new FormData(ref.current!)); }}>
    <input name="email" defaultValue="" />
    <button type="submit">Submit</button>
  </form>;
}
```

## React Query Pattern

```tsx
function useUsers() {
  return useQuery<User[]>({ queryKey: ["users"], queryFn: () => fetch("/api/users").then((r) => r.json()) });
}

function useCreateUser() {
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (data: CreateUserDto) => fetch("/api/users", { method: "POST", body: JSON.stringify(data) }),
    onSuccess: () => qc.invalidateQueries({ queryKey: ["users"] }),
  });
}
```

## Key Rules

- Keep components small — extract custom hooks for logic, child components for UI.
- Co-locate state with the component that owns it; lift only when siblings need it.
- Memoize expensive computations with `useMemo`, callback references with `useCallback` — but only when profiling shows a need.
- Always provide cleanup in `useEffect` for subscriptions, timers, and abort controllers.
