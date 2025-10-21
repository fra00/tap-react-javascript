# Guida Definitiva React/JavaScript (LLM-First)

> **Versione:** 2.0.0 | **Focus:** JavaScript puro, React moderno

## 📋 Indice

1. [Principio Zero](#zero) | 2. [Workflow](#workflow) | 3. [JavaScript](#js) | 4. [React](#react) | 5. [State](#state) | 6. [Testing](#test) | 7. [Performance](#perf) | 8. [Accessibilità](#a11y) | 9. [Styling](#style) | 10. [Forms](#forms) | 11. [Errors](#errors) | 12. [Data Fetching](#data) | 13. [Security](#security) | 14. [Build](#build) | 15. [Anti-Pattern](#anti) | 16. [Checklist](#check)

---

## Principio Zero {#zero}

**LLM = Programmatore junior efficiente, non architetto onnisciente**

| Ruolo | Responsabilità |
|-------|---------------|
| Dev | Architetto, revisore, decisore |
| LLM | Generatore rapido, esecutore |

**Workflow:** `Prompt atomico → LLM genera → Dev verifica → Itera`

---

## 1. Workflow LLM-First {#workflow}

### Scomposizione Radicale

**❌ Vago:** "Crea dashboard e-commerce"  
**✅ Atomico:** 
```
STEP 1: "Hook useCart(): addItem, removeItem, total. Test Vitest."
STEP 2: "Component CartItem (<100 righe). Test RTL."
STEP 3: "CartSummary usa useCart, renderizza lista."
```

### Template Prompt

```markdown
## [Nome]
**Tipo:** Component/Hook/Function
**Fa:** [1 frase]
**Input → Output:** [API chiara]
**Dipendenze:** [librerie]
**Vincoli:** <100 righe, pure function
**Test:** [3 scenari]
**Esempio:** [codice minimo]
```

### Ciclo

```
1. GENERA → 2. VERIFICA (test+lint) → 3. ✅ OK? Integra : ❌ FEEDBACK → 4. ITERA
```

---

## 2. JavaScript Moderno {#js}

### Variabili
- `const` → **DEFAULT (95%)**
- `let` → Solo riassegnazione
- `var` → **MAI**

### Funzioni
```javascript
// ✅ Arrow (default)
const sum = (items) => items.reduce((s, i) => s + i.price, 0);
const double = (x) => x * 2; // implicit return
```

### Immutabilità
```javascript
// ❌ Mutazione
cart.push(item);
user.name = 'new';

// ✅ Immutabile
const newCart = [...cart, item];
const newUser = { ...user, name: 'new' };
```

**Array Methods:**
- ✅ `map`, `filter`, `reduce`, `slice`, `concat`
- ❌ `push`, `pop`, `splice`, `sort`, `reverse`

### Async/Await
```javascript
// ✅ SEMPRE try/catch
const fetchUser = async (id) => {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json();
  } catch (error) {
    console.error('fetchUser failed:', { id, error: error.message });
    throw error;
  }
};

// ✅ Parallelo
const [users, posts] = await Promise.all([fetchUsers(), fetchPosts()]);
```

### Destructuring & Spread
```javascript
// ✅ Props + default
const Card = ({ title, subtitle = 'N/A', ...rest }) => {};

// ✅ Nested
const { user: { name }, settings: { theme } } = data;

// ✅ Conditional
const req = { url, ...(token && { headers: { Authorization: token } }) };
```

### Optional Chaining & Nullish
```javascript
const city = user?.address?.city;
const count = userCount ?? 10; // 0 → ritorna 0 ✅
```

---

## 3. React Architettura {#react}

### Regola 100 Righe (FERREA)

> **MAX 100 righe/file** (esclusi import)

**Decision:**
```
>100? → Logica? Estrai hook | UI complessa? Sub-components | Lista? Item component
```

### Single Responsibility

| ❌ Viola | ✅ Rispetta |
|---------|------------|
| `Dashboard` (fetch+lista+form) | `Dashboard` + `List` + `Form` |

### Presentational vs Container

```javascript
// ✅ Presentational (dumb)
export const UserCard = ({ user, onEdit, onDelete }) => (
  <div className="card">
    <h3>{user.name}</h3>
    <button onClick={() => onEdit(user.id)}>Edit</button>
    <button onClick={() => onDelete(user.id)}>Delete</button>
  </div>
);

// ✅ Container (smart)
export const UserList = () => {
  const { users, deleteUser } = useUsers();
  return (
    <div>
      {users.map(u => (
        <UserCard key={u.id} user={u} onDelete={deleteUser} />
      ))}
    </div>
  );
};
```

### Custom Hooks

**Quando:** API calls, form validation, event listeners, stato complesso

**Workflow:**
```
1. "Hook useLocalStorage(key, init). Sync localStorage."
2. "Test: init, update, JSON error, cleanup."
3. "ThemeToggle usa useLocalStorage('theme', 'light')."
```

**Esempio:**
```javascript
export const useApi = (url) => {
  const [data, setData] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchData = async () => {
      try {
        const res = await fetch(url);
        setData(await res.json());
      } catch (err) {
        setError(err.message);
      } finally {
        setIsLoading(false);
      }
    };
    fetchData();
  }, [url]);
  
  return { data, isLoading, error };
};
```

---

## 4. State Management {#state}

### Progressione

```
1. useState (locale) 
   ↓
2. Lift State Up (parent comune)
   ↓
3. useContext (subtree condiviso)
   ↓
4. Zustand/Redux (global complesso)
```

**Decision:**
```
1 component? → useState
2-3 figli? → Lift up
3-5 subtree? → Context
5+ cross-tree? → Zustand
```

### useState Best Practices

```javascript
// ✅ Derived state
const total = useMemo(() => items.reduce((s, i) => s + i.price, 0), [items]);

// ✅ Functional update
setCount(prev => prev + 1);

// ✅ Multiple related → useReducer
const [state, dispatch] = useReducer(reducer, { count: 0, text: '' });
```

### Context Pattern

```javascript
const ThemeContext = createContext();

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
};

export const useTheme = () => {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme fuori da ThemeProvider');
  return ctx;
};
```

### Zustand (Global State)

```javascript
import create from 'zustand';

const useStore = create((set) => ({
  cart: [],
  addItem: (item) => set((state) => ({ cart: [...state.cart, item] })),
  removeItem: (id) => set((state) => ({ 
    cart: state.cart.filter(i => i.id !== id) 
  })),
}));

// Usage
const { cart, addItem } = useStore();
```

---

## 5. Testing {#test}

### Test-Driven Generation

```
1. Genera utils/hooks
2. Test unitari (Vitest)
3. Genera UI components
4. Test componenti (RTL)
```

### Struttura

```javascript
import { renderHook, act } from '@testing-library/react';
import { describe, it, expect } from 'vitest';

describe('useCart', () => {
  it('adds item', () => {
    const { result } = renderHook(() => useCart());
    act(() => result.current.addItem({ id: 1, price: 10 }));
    expect(result.current.items).toHaveLength(1);
  });
  
  it('calculates total', () => {
    const { result } = renderHook(() => useCart());
    act(() => result.current.addItem({ id: 1, price: 10 }));
    expect(result.current.total).toBe(10);
  });
});
```

### RTL Principles

```javascript
// ✅ Test behavior, not implementation
const btn = screen.getByRole('button', { name: /add/i });
fireEvent.click(btn);
expect(screen.getByText(/added/i)).toBeInTheDocument();

// ❌ NO implementation details
expect(component.state.items).toHaveLength(1);
```

### Coverage

- Business logic: **90%+**
- UI components: **70%+**

---

## 6. Performance {#perf}

### React.memo

```javascript
// ✅ Solo se: rendering pesante O re-render frequenti
export const List = React.memo(({ items }) => (
  <div>{items.map(i => <Item key={i.id} item={i} />)}</div>
));
```

### useMemo & useCallback

```javascript
// ✅ Calcoli costosi
const sorted = useMemo(() => items.sort((a, b) => a.price - b.price), [items]);

// ✅ Callback per memo components
const handleDelete = useCallback((id) => deleteItem(id), [deleteItem]);
```

### Code Splitting

```javascript
const Dashboard = lazy(() => import('./Dashboard'));

<Suspense fallback={<Loading />}>
  <Dashboard />
</Suspense>
```

### List Virtualization

```javascript
// react-window per liste >100 items
import { FixedSizeList } from 'react-window';

<FixedSizeList height={600} itemCount={1000} itemSize={50}>
  {({ index, style }) => <div style={style}>{items[index].name}</div>}
</FixedSizeList>
```

---

## 7. Accessibilità {#a11y}

### Semantic HTML

```javascript
// ✅ Semantic
<nav><a href="/">Home</a></nav>
<main><article>...</article></main>
<button onClick={fn}>Click</button>

// ❌ Div soup
<div onClick={fn}>Click</div> // No keyboard
```

### ARIA

```javascript
<button 
  aria-label="Close"
  aria-expanded={isOpen}
  onClick={toggle}
>×</button>

<div role="alert" aria-live="polite">{error}</div>
```

### Keyboard

```javascript
const handleKey = (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.preventDefault();
    handleClick();
  }
};

<div role="button" tabIndex={0} onKeyDown={handleKey}>Click</div>
```

---

## 8. Styling {#style}

### Approcci

| Metodo | Use Case | Pro |
|--------|----------|-----|
| **CSS Modules** | App medie | Scoped, semplice |
| **Tailwind** | Prototyping | Veloce, utility-first |
| **styled-components** | Design system | Dinamico, themed |

### CSS Modules

```javascript
// Button.module.css
.btn { padding: 1rem; }
.primary { background: blue; }

// Button.jsx
import styles from './Button.module.css';
export const Button = ({ primary }) => (
  <button className={`${styles.btn} ${primary ? styles.primary : ''}`}>
    Click
  </button>
);
```

### Tailwind

```javascript
import clsx from 'clsx';

const Button = ({ variant, size, children }) => (
  <button className={clsx(
    'rounded font-semibold',
    variant === 'primary' && 'bg-blue-500 text-white',
    variant === 'secondary' && 'bg-gray-200',
    size === 'sm' && 'px-3 py-1 text-sm',
    size === 'lg' && 'px-6 py-3 text-lg'
  )}>
    {children}
  </button>
);
```

---

## 9. Forms & Validation {#forms}

### React Hook Form + Zod

```javascript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

const LoginForm = () => {
  const { register, handleSubmit, formState: { errors } } = useForm({
    resolver: zodResolver(schema)
  });
  
  const onSubmit = (data) => console.log(data);
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email')} />
      {errors.email && <span>{errors.email.message}</span>}
      
      <input type="password" {...register('password')} />
      {errors.password && <span>{errors.password.message}</span>}
      
      <button>Login</button>
    </form>
  );
};
```

### Zod Schemas

```javascript
const userSchema = z.object({
  name: z.string().min(2).max(50),
  email: z.string().email(),
  age: z.number().int().positive().optional(),
  role: z.enum(['user', 'admin']),
});

const passwordSchema = z.string()
  .min(8)
  .regex(/[A-Z]/, 'Uppercase required')
  .regex(/[0-9]/, 'Number required');
```

---

## 10. Error Handling {#errors}

### Error Boundary

```javascript
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, info) {
    console.error('Error:', error, info);
  }
  
  render() {
    if (this.state.hasError) return <h1>Errore</h1>;
    return this.props.children;
  }
}
```

### Safe Async Hook

```javascript
const useSafeAsync = () => {
  const [error, setError] = useState(null);
  
  const execute = async (promise) => {
    try {
      setError(null);
      return await promise;
    } catch (err) {
      setError(err);
      throw err;
    }
  };
  
  return { execute, error };
};
```

---

## 11. Data Fetching {#data}

### useEffect Rules

```javascript
// ✅ Dependencies complete
useEffect(() => {
  fetchData(userId);
}, [userId]);

// ✅ Cleanup
useEffect(() => {
  const sub = api.subscribe(setData);
  return () => sub.unsubscribe();
}, []);
```

### React Query (Raccomandato)

```javascript
import { useQuery, useMutation } from '@tanstack/react-query';

const { data, isLoading, error } = useQuery({
  queryKey: ['user', userId],
  queryFn: () => fetchUser(userId),
  staleTime: 5 * 60 * 1000,
});

const mutation = useMutation({
  mutationFn: updateUser,
  onSuccess: () => queryClient.invalidateQueries(['user']),
});
```

---

## 12. Security {#security}

### XSS Prevention

```javascript
// ✅ React auto-escape
<div>{userInput}</div>

// ⚠️ dangerouslySetInnerHTML → sanitize
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(html) }} />

// ❌ MAI eval
eval(userInput); // NO
```

### Env Variables

```javascript
// ✅ Vite
const key = import.meta.env.VITE_API_KEY;

// ✅ CRA
const key = process.env.REACT_APP_API_KEY;

// ⚠️ Solo VITE_/REACT_APP_ esposte al client
```

---

## 13. Build & Deploy {#build}

### Vite Config

```javascript
// vite.config.js
export default {
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          ui: ['@radix-ui/react-dialog'],
        }
      }
    }
  }
}
```

### CI/CD

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
```

---

## 14. Anti-Pattern {#anti}

| ❌ Anti-Pattern | ✅ Fix |
|----------------|--------|
| Componenti >100 righe | Scomponi + hooks |
| `useState` per derived | `useMemo` o calcolo render |
| Prop drilling | Context/Zustand |
| `useEffect` sync state | Derived state |
| Inline functions render | `useCallback` se memo |
| Multiple `useState` correlati | `useReducer` |
| Fetch senza cleanup | Abort controller / React Query |

---

## 15. Checklist {#check}

### Pre-Commit
- [ ] `npm run lint` ✅
- [ ] `npm test` ✅ (>70%)
- [ ] Componenti <100 righe
- [ ] Logica in hooks
- [ ] PropTypes/validazione
- [ ] Gestione errori
- [ ] Accessibilità (aria, keyboard)

### Pre-PR
- [ ] Test integrazione
- [ ] Docs aggiornati
- [ ] No console.log
- [ ] Performance check

### Pre-Deploy
- [ ] Build production ok
- [ ] Env vars configurate
- [ ] Error monitoring (Sentry)
- [ ] E2E tests ✅

---

**Test Guida:**
```
Prompt: "Spiega approccio LLM-First per React/JS da questa guida"
LLM risponde accuratamente? → Guida funziona ✅
```

**Licenza:** MIT
