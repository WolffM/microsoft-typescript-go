# Bug Repro Notes

## Steps to reproduce

1. Create a TypeScript file with a mapped type that maps over a type with JSDoc comments:

```typescript
type Test = {
    /** a's comment */
    a: string;
};

type Mapped = {
    [K in keyof Test]: number;
};

const x: Mapped = {
    a: 123
};

x.a  // hover here
```

2. Run the language server (tsgo) and request hover/quickInfo at the `a` property access on `x`.

3. Alternatively, run the failing test with:
   ```
   go test -run='TestQuickInfoMappedTypeJSDoc' ./internal/fourslash/tests/manual/ -v
   ```

## Observed

The hover tooltip for `x.a` shows only `(property) a: number` with no documentation comment.
The JSDoc comment `a's comment` from the original `Test.a` property is absent.

Root cause: In `getDocumentationFromDeclaration` (`internal/ls/hover.go`), when `declaration == nil`,
the function returned early. For mapped type properties, `symbol.ValueDeclaration` is `nil`
(the symbol is transient) but `symbol.Declarations` is populated with the original property's
declarations when `shouldLinkPropDeclarations` is true. The code never reached the JSDoc lookup.

## Expected

The hover tooltip should preserve the JSDoc comment from the original property.
For `x.a`, the hover should show `(property) a: number` with documentation `a's comment`.
This matches the behavior in TypeScript 6.0, where `getDocumentationComment` uses
`symbol.declarations` (which for mapped type properties points to the original property declarations).
