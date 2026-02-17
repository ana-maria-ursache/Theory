# 1. useEffect 

## What it does?
Runs a side-effect after the DOM has been painted. Use it to synchronize with external systems: fetch data, subscribe to events, manipulate non-React APIs, update document title, etc.

## Signature:
```
const cleanup = useEffect(
  () => {
    // effect body
    return () => { /* optional cleanup */ };
  },
  [dep1, dep2] // optional dependency array
);
```
#### Depending on [dep1, dep2]:
- [dep1, dep2, ..] -> the effect re-runs every time one of those dep changes
- [] -> runs 1 time after render/mount
- useEffect(()=>{...})(without dep) -> runs at every renders

## When to use:
- Fetching data, attaching event listeners, timers.
- Subscribing to websockets or external stores (and cleaning up).
- Syncing DOM things that React doesn’t handle (e.g., document.title).

## What to pay attention to:
- Cleanup: Return a cleanup to avoid leaks. Cleanup runs before the effect re-runs and on unmount.
- Race conditions: For fetch, use AbortController or an incrementing request ID to ignore stale results.
- StrictMode in dev: Effects mount/unmount twice in development to help catch issues (not in production).
- useEffect vs useLayoutEffect: useLayoutEffect runs before paint; use only if you must measure/mutate DOM before the browser paints.

## Examples:

### a) Data fetching with cancellation
```
function UsersList() {
  const [users, setUsers] = useState([]);
  const [status, setStatus] = useState('idle'); // 'idle' | 'loading' | 'success' | 'error'

  useEffect(() => {
    const controller = new AbortController();
    const load = async () => {
      setStatus('loading');
      try {
        const res = await fetch('/api/users', { signal: controller.signal });
        if (!res.ok) throw new Error('Network error');
        const data = await res.json();
        setUsers(data);
        setStatus('success');
      } catch (e) {
        if (e.name !== 'AbortError') setStatus('error');
      }
    };
    load();
    return () => controller.abort();
  }, []); // run once after mount

  if (status === 'loading') return <p>Loading…</p>;
  if (status === 'error') return <p>Failed to load</p>;
  return <ul>{users.map(u => <li key={u.id}>{u.name}</li>)}</ul>;
}
```

### b) Subscribing to a WebSocket
```
function StockTicker({ symbol }) {
  const [price, setPrice] = useState(null);

  useEffect(() => {
    const socket = new WebSocket(`wss://example.com/ticker?symbol=${symbol}`);
    socket.addEventListener('message', e => {
      const { price: p } = JSON.parse(e.data);
      setPrice(p);
    });
    return () => socket.close();
  }, [symbol]); // resubscribe if symbol changes

  return <div>{symbol}: {price ?? '—'}</div>;
}
```

### c) Document title
```
useEffect(() => {
  document.title = `Inbox (${unreadCount})`;
}, [unreadCount]);
```