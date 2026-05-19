# flux-snobol — SNOBOL4 Constraint Engine

SNOBOL4 implementation of the Flux constraint validation pipeline. Pattern matching as control flow, success/failure as the error mask.

## How to Read SNOBOL4

SNOBOL4 (1966) is a string-processing language where pattern matching is the primary operation:

```sno
       &ANCHOR = 1                ;* anchor patterns to start of string
       &TRIM = 1                  ;* trim input

       VAR = 'hello'              ;* assignment (no :)

*-     Pattern match: does VAR match 'hel' at the start?
       VAR 'hel'                  ;* succeeds (ANCHOR = 1)
       :S(LABEL1) :F(LABEL2)      ;* goto LABEL1 on success, LABEL2 on fail

*-     DEFINE a function
       DEFINE('MYFUNC(X,Y)')
MYFUNC MYFUNC = X + Y             ;* return value = function name
       :(RETURN)                  ;* explicit return

*-     Control flow: predicates as goto conditions
       GT(X, 10) :S(BIG)          ;* if X > 10 goto BIG
       LT(X, 0) :S(NEG)          ;* if X < 0 goto NEG
       :(DEFAULT)                  ;* else goto DEFAULT

*-     TABLE = associative array
       T = TABLE()
       T<'KEY.1'> = 42

*-     ARRAY
       A = ARRAY(10)
       A<3> = 99
```

Key ideas:
- Every line is a statement ending at `.` (period marks statement end in label context — actually, lines are statements)
- Labels start in column 1+, code is indented
- `:S(label)` = goto on success, `:F(label)` = goto on failure
- `:(label)` = unconditional goto
- No block structure — all control flow is goto-based on pattern success/failure
- TABLE = associative array (string keys, any values)
- `DEFINE('FUNC(ARGS)')` declares a function; code follows the label

## What SNOBOL4 Forces You to Think About

### 1. Success/Failure IS the Error Mask
SNOBOL4 has no exceptions, no booleans, no error codes. Every operation either **succeeds** or **fails**, and you branch accordingly:

```
       LT(VALUE, LO) :S(VIOLATED)
       GT(VALUE, HI) :S(VIOLATED)
       :(OK)
```

This IS the constraint check. The `:S` and `:F` labels ARE the bitmask. In SNOBOL4, control flow IS the error reporting mechanism. No intermediate data structure needed.

This is the Maybe monad before monads (1966). Success = Just (continue), Failure = Nothing (branch to handler).

### 2. TABLE as Schema-Free Storage
SNOBOL4's TABLE is an associative array with string keys. You build composite keys: `'LO.3'` means "lower bound for constraint 3". This is essentially a flattened struct — and it teaches you that:

- Data identity is a **string key**, not a memory address
- "Objects" are emergent from naming conventions
- This is the same pattern as URL routing, environment variables, and database key-value stores

### 3. No Block Structure
Every function is a labeled section with an explicit `:(RETURN)`. There are no nested scopes, no local variables by default. This forces you to:
- Think about data flow, not scope
- Use naming conventions to avoid collisions
- Make state transitions explicit (no hidden mutations in nested closures)

### 4. String Processing Constraints
SNOBOL4 makes you think of everything as pattern matching. "Is this value in range?" is a pattern. "Does this constraint exist?" is a pattern. The entire constraint engine is a pattern-matching pipeline.

## The Architectural Insight

**SNOBOL4's success/failure flow control IS the error mask in control-flow form.** Every constraint check is a pattern match that branches to success or failure. The accumulated goto paths form the same error mask that COBOL stores in a register.

In 1966, SNOBOL4 invented:
- The Maybe monad (success/failure branching)
- Pattern matching as a first-class operation (now in Rust, Scala, Elixir)
- Associative arrays as the universal data structure (now JSON, Redis, environment variables)

The constraint engine doesn't need an error mask variable in SNOBOL4 — the control flow IS the mask. But we compute one anyway for interop, proving that the concept is language-independent even when the implementation differs.

## Files

| File | Purpose |
|------|---------|
| `FLXCHECK.sno` | Core constraint checking with pattern-match flow control |
| `FLXFRACT.sno` | Fracture engine: BFS connectivity as reachability patterns |
| `FLXSEDIMNT.sno` | Sediment layer manager with TABLE-based storage |
| `FLXMAIN.sno` | Main program wiring the full pipeline |

## Syntax Notes

These files use standard SNOBOL4 syntax as defined in Griswold, Poage, and Polonsky (1971). Conventions:
- Labels at column 1+, code indented
- `:S(label) :F(label)` for success/failure branching
- `:(RETURN)` for function returns
- TABLE for associative storage, ARRAY for indexed storage
- Comments begin with `-*`
