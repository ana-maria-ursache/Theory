# 4. useRef

## What it does?
Returns a mutable object { current } that persists for the lifetime of the component. Updating ref.current does not trigger a re-render. Two common uses:
- DOM refs (imperatively interact with a DOM node).
- Instance variables (store mutable values across renders, e.g., previous value, timers).

## Signature:
```
const ref = useRef(initialValue); // { current: T }
```

## When to use:
- Focus/measure DOM elements.
- Store last value to compare on next render.
- Keep mutable non-visual state (e.g., a WebSocket instance, interval ID).

## What to pay attention to:
- Changing ref.current won’t re-render. If the UI must update, use state.
- For DOM elements, initialize with null and pass to ref prop.
- If you need to react to ref changes, use a callback ref.
- When exposing DOM refs to parents, wrap component with forwardRef.

## Examples:

### a) Auto-focus an input
```
function SearchBox() {
  const inputRef = useRef(null);
  return (
    <>
      <input ref={inputRef} placeholder="Search…" />
      <button onClick={() => inputRef.current?.focus()}>Focus</button>
    </>
  );
}
```

### b) Store previous value
```
function Counter() {
  const [count, setCount] = useState(0);
  const prev = useRef(count);

  useEffect(() => {
    prev.current = count; // update after render
  }, [count]);

  return <p>Prev: {prev.current}, Now: {count} <button onClick={() => setCount(c => c + 1)}>+</button></p>;
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