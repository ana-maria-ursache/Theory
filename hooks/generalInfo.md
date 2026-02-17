# Quick “When should I use what?” Cheatsheet

- ### I need to fetch / subscribe / set up a timer / touch the DOM → useEffect (with cleanup).
- ### I need to read a value from high up without prop drilling → useContext (memoize provider value).
- ### I need unique, SSR-safe IDs for accessibility → useId (not for keys).
- ### I need to imperatively access a DOM node or store a mutable value across renders → useRef.
- ### I need to avoid recomputing an expensive value / keep object identity stable → useMemo.
- ### I need a stable function identity for a memoized child or as a dependency → useCallback.


# useState vs useRef

- Use useState when the value affects rendering and you want the UI to update when it changes.
- Use useRef when updating the value should not trigger a re-render (e.g., storing setInterval ID, an imperative API instance, or the previous value for comparison).


# Summary
| Hook | Main idea |
| --- | --- |
| 0. useState | Add local reactive state to a component. Use when a value should persist across renders and update the UI (e.g., input values, toggles, loading flags). |
| 1. useEffect | Run side effects after render to sync with the outside world (fetch data, subscriptions, timers, non-React DOM APIs). Include all dependencies; return a cleanup to avoid leaks. Not for pure computations. |
| 2. useContext | Read a value from the nearest Provider to avoid prop drilling (theme, locale, auth user, clients). Memoize provider values to avoid unnecessary re-renders; split contexts if parts change at different rates. |
| 3. useId | Generate unique, stable, SSR-safe IDs for accessibility linkage (label/input, aria-describedby). |
| 4. useRef | Keep a mutable value that doesn’t trigger re-renders (DOM node refs, instance variables, timers, previous values). Use when you need imperative access or non-visual state. If UI must update, use state instead. |
| 5. useMemo | Memoize the result of an expensive/pure computation or stabilize object/array identities passed to memoized children. |
| 6. useCallback | Memoize a function so its identity is stable across renders (useful for React.memo children, subscriptions, or dependency arrays). Include all referenced values in deps or use functional updates/refs to avoid stale closures. |
