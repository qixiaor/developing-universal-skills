---
name: frontend-contract-style
description: Enforce strict frontend API contract updates and consistent implementation style. Use when Codex changes frontend request payloads, parameter names, response handling, component logic, or related refactors that must converge on one standard contract without transitional compatibility and must keep concise, consistent code organization and comments.
---

# Frontend Contract Style

Apply these rules whenever a frontend task touches interface contracts or implementation style.

## Follow This Workflow

1. Confirm the standard contract first.
2. Search the whole frontend repo for old call sites before editing.
3. Choose one of two outcomes only:
   - Update every old call site to the standard contract.
   - Keep only the standard contract and remove old paths immediately.
4. Do not leave compatibility mappings, fallback fields, or mixed parameter names unless the user explicitly requires them.
5. Verify the touched flow with the smallest useful build, test, or search pass.

## Enforce API Contract Discipline

- Use `rg` to locate request helpers, component calls, stores, hooks, and constants tied to the same interface.
- Rename fields at every call site instead of translating old fields inside the API layer.
- Keep request bodies, query params, and function signatures aligned to the backend contract exactly.
- Remove dead branches and obsolete fields once the standard contract is in place.

## Keep Style Consistent

- Match surrounding code style for parameter passing, request helpers, naming, imports, and tool usage.
- Prefer concise code over abstraction layers added only for compatibility.
- Keep related constants, state, helpers, and handlers adjacent instead of scattering them.
- Use line comments to split logical sections when the file benefits from structure.
- Add a short comment to every variable declaration and every method definition in the changed code, unless the file's established style makes that impossible.
- Keep comments factual and brief; explain purpose, not syntax.

## Use This Search Pattern

```powershell
rg "oldFieldName|oldMethodName|targetEndpoint|requestFunction" src
```

Search before editing and again after editing to confirm old names are gone.

## Apply This Edit Pattern

```js
// ==================== api ====================
/**
 * Submit runner audit result.
 */
export function doAudit(data) {
  // Request payload.
  return post('/runner/admin/runner/audit/doAudit', data);
}

// ==================== submit ====================
/**
 * Submit audit action with the standard backend contract.
 */
async function handleSubmit() {
  // Audit payload.
  const payload = {
    // Record id.
    id: form.id,
    // Audit status.
    certStatus: form.certStatus,
    // Reject reason.
    rejectReason: form.rejectReason,
  };

  await doAudit(payload);
}
```

## Final Checks

- Re-run `rg` to confirm old fields or old helpers are removed.
- Build or test the affected frontend target when feasible.
- Report whether any unverified paths remain.
