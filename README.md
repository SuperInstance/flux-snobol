# Flux SNOBOL4 — What Pattern Matching Teaches About Constraint Pass/Fail

SNOBOL4 (1966) is a string-processing language where pattern matching is the primary control flow mechanism. It runs in academic contexts and legacy text-processing systems. This repo implements the Flux constraint engine in SNOBOL4.

## How to Read SNOBOL4

SNOBOL4 has no block structure, no if/else, no for loops in the modern sense. Every operation succeeds or fails, and you branch accordingly.

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
- **Every operation succeeds or fails.** Pattern matching, comparison, assignment — all produce success/failure.
- **`:S(label) :F(label)`** — goto on success / goto on failure. This IS your control flow.
- **`:(label)`** — unconditional goto.
- **Labels start at column 1**, code is indented. No block structure.
- **TABLE** — associative array with string keys. SNOBOL4's primary data structure.
- **DEFINE('FUNC(ARGS)')** — declares a function; the code follows the label.
- **`:(RETURN)`** — explicit return from function.
- **Comments** begin with `-*`

## How the Constraint Engine Maps to SNOBOL4

| Constraint Engine Concept | SNOBOL4 Mechanism |
|---------------------------|-------------------|
| Constraint check (pass/fail) | Pattern match success/failure — `LT(V,LO) :S(VIOLATED)` |
| Error mask | Accumulated goto paths — control flow IS the mask |
| Constraint bounds storage | TABLE with composite keys: `T<'LO.3'>` = lower bound for constraint 3 |
| Sediment layers | TABLE-based layer stack: `T<'SEDLO.3.1'>` = layer 1 correction |
| BFS fracture | Reachability via pattern-driven traversal |
| Pipeline stages | Labeled sections with `:S/:F` branching |

```
FLXCHECK.sno    — Core constraint checking with pattern-match flow control
FLXFRACT.sno    — Fracture engine: BFS connectivity as reachability patterns
FLXSEDIMNT.sno  — Sediment layer manager with TABLE-based storage
FLXMAIN.sno     — Main program wiring the full pipeline
```

## What SNOBOL4 Teaches Us

**Success/failure IS the error mask in control-flow form.** SNOBOL4 has no exceptions, no booleans, no error codes. Every operation either succeeds or fails, and you branch accordingly:

```
       LT(VALUE, LO) :S(VIOLATED)
       GT(VALUE, HI) :S(VIOLATED)
       :(OK)
```

This IS the constraint check. The `:S` and `:F` labels ARE the bitmask. No intermediate data structure needed. In SNOBOL4, control flow IS the error reporting mechanism.

This is the Maybe monad before monads (1966). Success = Just (continue), Failure = Nothing (branch to handler).

Three specific lessons:

1. **Pattern matching as constraint checking.** "Is this value in range?" is a pattern. SNOBOL4 makes you think of constraint checking as pattern matching — because it is. Every constraint is a question with a yes/no answer, and the branching tells you everything.

2. **TABLE as schema-free storage.** `T<'LO.3'>` means "lower bound for constraint 3." Composite string keys create emergent structure without structs. This is the same pattern as URL routing, environment variables, Redis keys, and JSON property paths. Data identity is a **string key**, not a memory address.

3. **No block structure forces explicit state transitions.** Every function is a labeled section with `:(RETURN)`. No nested scopes, no hidden mutations in closures. You must think about data flow, not scope. This discipline — making state transitions explicit — is exactly what constraint systems need.

## Files

| File | Purpose |
|------|---------|
| `FLXCHECK.sno` | Core constraint checking with pattern-match flow control |
| `FLXFRACT.sno` | Fracture engine: BFS connectivity as reachability patterns |
| `FLXSEDIMNT.sno` | Sediment layer manager with TABLE-based storage |
| `FLXMAIN.sno` | Main program wiring the full pipeline |

## Syntax Notes

Standard SNOBOL4 syntax as defined in Griswold, Poage, and Polonsky (1971). Labels at column 1+, code indented. Comments begin with `-*`.

## Where to Go Next

- **flux-pli** — PL/I takes the opposite approach: the error mask is a native BIT(8) data type, not control flow.
- **flux-mumps** — MUMPS adds persistence to SNOBOL4's string-centric worldview. Globals survive process exits.
- **flux-algol** — ALGOL 60 provides the block structure that SNOBOL4 deliberately rejects. Both perspectives teach something.
- **flux-docs** — Full documentation: error masks, fracture-coalesce, sediment, thermodynamic analogy.

## Author

Forgemaster ⚒️ — Constraint Theory Ecosystem, 2026-05-19
