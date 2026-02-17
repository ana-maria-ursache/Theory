# 3. useId 

## What it does?
Returns a unique, stable ID string for accessibility attributes that must match (e.g., id + aria-describedby). It works with SSR/hydration without mismatches.

## Signature:
```
const id = useId(); // string
```

## When to use:
- Connect `<label htmlFor>` with `<input id>`.
- Tie controls to help text via aria-describedby, aria-labelledby.
- Generate multiple related IDs by suffixing.

## What to pay attention to:
- Not for keys in lists—use data-derived keys instead.
- Stable across renders, but not meant for persisted identifiers or database IDs.
- Allow external override through props for composition.

## Examples:

### a) Accessible Field component
```
function Field({ label, error, id: idProp, ...inputProps }) {
  const reactId = useId();
  const id = idProp ?? `field-${reactId}`;
  const errorId = `error-${reactId}`;
  const describe = error ? { 'aria-describedby': errorId } : {};

  return (
    <div>
      <label htmlFor={id}>{label}</label>
      <input id={id} {...describe} {...inputProps} />
      {error && <div id={errorId} role="alert">{error}</div>}
    </div>
  );
}
```