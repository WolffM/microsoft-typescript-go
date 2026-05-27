## Steps to reproduce
1. Install dependencies with `npm install` in the repository root.
2. Build once with `npx hereby build` to ensure the workspace is healthy.
3. Add and run the focused reproduction test: `go test ./internal/fourslash/tests -run TestQuickInfoJSDocMappedTypePropertyRegression -count=1`.
4. Observe the quick info assertion failure for mapped-type property hover documentation.

## Observed
The test fails consistently. The hover payload contains only the code block for `(property) a: number` and omits the JSDoc text. The assertion diff shows that the expected documentation line `a's comment` is missing from hover markdown content. This reproduces the behavior difference where mapped type property hovers drop source JSDoc.

## Expected
For parity with TypeScript 6.0 behavior described in the Bloomberg-related feedback, quick info on `x.a` should include inherited documentation from the source property declaration. The hover markdown should include both the signature `(property) a: number` and the doc text `a's comment`, instead of returning an empty documentation section.
