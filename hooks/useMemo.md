# 5. useMemo

## What it does?
Memoizes the result of a calculation to avoid recomputing on every render, or to keep stable object/array identities for children that rely on referential equality.

## Signature:
```
const memoizedValue = useMemo(() => computeSomething(a, b), [a, b]);
```

## When to use:
- Expensive computations derived from props/state.
- Creating stable objects/arrays passed to React.memo’d children or used as dependencies (e.g., table columns config, filter predicates).

## What to pay attention to:
- The function must be pure (no side effects).
- Include all dependencies. If there are too many, restructure code (e.g., move logic to a reducer or use refs).
- Don’t use it indiscriminately—there’s overhead. Optimize where profiling shows benefit.
- It memoizes values—not components. (You can memoize JSX, but it’s just a value; still ensure it truly avoids work.)

## Examples:

### a) Expensive filtering
```
function SearchableList({ items }) {
  const [query, setQuery] = useState('');
  const filtered = useMemo(() => {
    const q = query.toLowerCase();
    // Imagine this is expensive for large lists
    return items.filter(i => i.name.toLowerCase().includes(q));
  }, [items, query]);

  return (
    <>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <ul>{filtered.map(i => <li key={i.id}>{i.name}</li>)}</ul>
    </>
  );
}
```

### b) Stable options object for a memoized child
```
const Chart = memo(function Chart({ data, options }) {
  // re-renders only when data or options identity changes
  return <div>/* render chart */</div>;
});

function Dashboard({ data }) {
  const [dark, setDark] = useState(false);
  const options = useMemo(() => ({
    theme: dark ? 'dark' : 'light',
    smoothing: 0.7
  }), [dark]);

  return (
    <>
      <label>
        <input type="checkbox" checked={dark} onChange={e => setDark(e.target.checked)} />
        Dark mode
      </label>
      <Chart data={data} options={options} />
    </>
  );
}
```
Info:
- memo() is a higher‑order component from React that wraps your component and gives it this behavior:
- React will skip re‑rendering this component if its props have NOT changed (by shallow comparison).

Why do we need it? 
- React re-renders components when: their parent re-renders, their props change, their state changes

- But what if the parent re-renders … and the props are exactly the same?

→ Without memo(), React still re-renders the child unnecessarily.

→ With memo(), React skips re-rendering the child if props are the same by reference.