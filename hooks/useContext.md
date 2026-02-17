# 2. useContext

## What it does?
Reads the current value from the nearest matching <Context.Provider>. It’s for passing data through the tree without prop drilling.

Prop drilling usually occurs because React has a unidirectional data flow (data only flows down from parent to child). If your state lives in Component A but is needed in Component D, you must pass it through B and C first.

While not technically an error, heavy prop drilling makes code harder to maintain for several reasons:
- Boilerplate: You end up writing the same prop names in every component along the chain.
- Refactoring pain: If you rename a prop or change the data structure, you have to update every single component in the drilling path.
- Component coupling: Middleman components become unnecessarily "aware" of data they don't care about, making them harder to reuse elsewhere.

## Signature:
```
const value = useContext(MyContext);
```

## When to use:
- Global-ish values: theme, locale, auth user, feature flags, a client instance.
- Avoid threading the same prop down many levels.

## What to pay attention to:
- Re-renders: All consumers re-render when the provider’s value identity changes. Memoize value objects/functions with useMemo/useCallback to keep identity stable.
- Context size: Big values that change frequently cause many re-renders—consider splitting into multiple contexts.
- Default value: If you rely on createContext(defaultValue), ensure it makes sense for “no provider” cases, or throw in a custom hook.
- Custom hooks wrap context read to centralize usage and enforce invariants.

## Examples:

### a) Theme context with stable provider value

```
import { createContext, useContext, useMemo, useState } from 'react';
const ThemeContext = createContext(null);

export function ThemeProvider({ children }) {
  const [mode, setMode] = useState('light');
  const toggle = () => setMode(m => (m === 'light' ? 'dark' : 'light'));

  // Memoize to avoid re-renders in consumers when things didn’t change
  const value = useMemo(() => ({ mode, toggle }), [mode]);

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

export function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used within <ThemeProvider>');
  return ctx;
}

// Consumer
function Header() {
  const { mode, toggle } = useTheme();
  return (
    <header data-theme={mode}>
      <button onClick={toggle}>Toggle theme</button>
    </header>
  );
}
```