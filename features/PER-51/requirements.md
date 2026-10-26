# Requirements — PER-51

**Ticket:** Choose the caller-facing readiness word-v3

**Product decision required:** Choose between "ready" and "available" as the readiness status word for callers of the graphql service.

**Implementation scope after decision:** Add exactly one function to `src/health.py` that returns the chosen word.

---

## Requirements

### REQ-1: Product chooses readiness word
**Given** two candidate words ("ready", "available")  
**When** product reviews the options  
**Then** product selects exactly one word for caller-facing readiness status

### REQ-2: New function returns chosen word in success-key format
**Given** product has chosen the readiness word  
**When** the new function in `src/health.py` is called  
**Then** it returns `{"<chosen_word>": "success"}`

**Example (if "ready" chosen):**
```python
{"ready": "success"}
```

**Example (if "available" chosen):**
```python
{"available": "success"}
```

### REQ-3: Existing health() unchanged
**Given** the new function is implemented  
**When** `health()` is called  
**Then** it still returns exactly `{"status": "ok"}`

---

## Constraints

- **One function only:** Add exactly one new function to `src/health.py`. Do not modify `health()`.
- **One repo:** Changes are confined to the `graphql` repo (`src/health.py`).
- **No framework/stack decisions:** This is a product outcome requirement. Implementation details (function name, signature) are left to the developer.
- **Sign-off gate:** No implementation work begins until product chooses the word.

---

## Non-goals

- Changing the existing `health()` function
- Adding new API endpoints or routes
- UI changes
- Edge case handling beyond the basic function return
- Additional design documents
- Work in other repos

---

## Success signal

- Product has documented their choice of "ready" or "available"
- The new function in `src/health.py` exists and returns the chosen word in the success-key format `{"<word>": "success"}`
- `health()` continues to return `{"status": "ok"}` unchanged
- Tests pass for both functions
