# 6. useCallback

## What it does?
Returns a memoized function whose identity only changes when dependencies change. Useful when passing callbacks to memoized children or when a function is part of a dependency list.

## Signature:
```
const stableFn = useCallback((...args) => { /* body */ }, [dep1, dep2]);
```

## When to use:
- Passing callbacks to React.memo or useMemo to avoid unnecessary renders.
- Stable handlers required by libraries or effects (e.g., subscribe/unsubscribe APIs needing the same function identity).
- As a dependency for other hooks to prevent churn.

## What to pay attention to:
- Stale closures: Include all referenced values in the dependency array, or use functional updates (setState(prev => ...)) or refs to avoid capturing stale values.
- Pair with React.memo: useCallback helps only if the receiving child is memoized or the function identity itself is used by an effect.
- Don’t overuse; some prop changes are cheap and useCallback adds overhead.

## Examples:

### a) Stable handler for a memoized child
```
const TodoItem = memo(function TodoItem({ item, onToggle }) {
  console.log('render', item.id);
  return (
    <li>
      <label>
        <input
          type="checkbox"
          checked={item.done}
          onChange={() => onToggle(item.id)}
        />
        {item.text}
      </label>
    </li>
  );
});

function TodoList({ initial }) {
  const [todos, setTodos] = useState(initial);

  const handleToggle = useCallback((id) => {
    setTodos(ts => ts.map(t => t.id === id ? { ...t, done: !t.done } : t));
  }, []); // uses functional update, so no deps

  return <ul>{todos.map(t => <TodoItem key={t.id} item={t} onToggle={handleToggle} />)}</ul>;
}
```

### b) Stable subscription handler
```
function KeyboardShortcuts() {
  const onKeyDown = useCallback((e) => {
    if (e.key === 's' && (e.ctrlKey || e.metaKey)) {
      e.preventDefault();
      // save action
    }
  }, []);

  useEffect(() => {
    window.addEventListener('keydown', onKeyDown);
    return () => window.removeEventListener('keydown', onKeyDown);
  }, [onKeyDown]); // stable identity prevents resubscribing
  return null;
}
```

### c) Interval with ref (avoid stale closures)
```
function Ticker() {
  const [n, setN] = useState(0);
  const savedCallback = useRef(() => {}); // latest callback

  useEffect(() => {
    savedCallback.current = () => setN(x => x + 1);
  }, []);

  useEffect(() => {
    const id = setInterval(() => savedCallback.current(), 1000);
    return () => clearInterval(id);
  }, []);

  return <div>{n}</div>;
}
```