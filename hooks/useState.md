# 0. useSate

## What it does?
Adds local, reactive state to a function component. When you call the setter, React schedules a re-render with the new state value.

## Signature:
```
const [state, setState] = useState(initialValue);
```
#### [state, setState]:
- state = the value
- setState = function to set the state value, use: setState(x)(state = x)
- useState(initialValue): state = initialValue

## When to use:
- You need component-local state that affects rendering (form inputs, toggles, UI flags, async status).
- You need to remember values across renders and trigger a re-render when they change.

## Examples:

### a) Basic toggle with functional update (safe)
```
function Toggle() {
  const [on, setOn] = useState(false);
  return (
    <button onClick={() => setOn(prev => !prev)}>
      {on ? 'On' : 'Off'}
    </button>
  );
}
```

### b) Lazy initialization (expensive initial compute)
```
function expensiveInit() {
  // imagine heavy computation or parsing once on mount
  const items = Array.from({ length: 10000 }, (_, i) => i);
  return items;
}

function LargeList() {
  const [list, setList] = useState(() => expensiveInit()); // for expensive stuff

  return (
    <div>
      <p>Length: {list.length}</p>
      <button onClick={() => setList(prev => prev.slice(1))}>Pop first</button>
    </div>
  );
}
``
```

### c) Updating objects and arrays immutably
```
function ProfileEditor() {
  const [user, setUser] = useState({ name: 'Ana', role: 'Engineer', skills: ['JS'] });

  const addSkill = (skill) => {
    setUser(prev => ({
      ...prev,
      skills: [...prev.skills, skill]   // create a new array
    }));
  };

  const rename = (name) => {
    setUser(prev => ({ ...prev, name }));
  };

  return (
    <>
      <input value={user.name} onChange={e => rename(e.target.value)} />
      <button onClick={() => addSkill('React')}>Add React</button>
      <pre>{JSON.stringify(user, null, 2)}</pre>
    </>
  );
}
```

### d) Avoid stale state with timers (functional update)
```
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      // Always uses the latest value thanks to functional update
      setCount(c => c + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []);

  return <div>{count}</div>;
}
```

### e) Initialize from props only once (controlled-to-uncontrolled “draft” pattern)
```
function EditableName({ initialName }) {
  const [name, setName] = useState(() => initialName); // initialize from prop

  // If you need to reset when prop changes intentionally, handle it explicitly:
  useEffect(() => {
    setName(initialName);
  }, [initialName]);

  return (
    <>
      <input value={name} onChange={e => setName(e.target.value)} />
      <p>Preview: {name}</p>
    </>
  );
}
```

### f) Multiple updates in one event (batched)
```
function Stepper() {
  const [value, setValue] = useState(0);

  const addThree = () => {
    // If you used setValue(value + 1) three times, you’d end up with +1 (due to batching).
    // The functional form composes correctly:
    setValue(v => v + 1);
    setValue(v => v + 1);
    setValue(v => v + 1);
  };

  return (
    <>
      <p>{value}</p>
      <button onClick={addThree}>+3</button>
    </>
  );
}
```