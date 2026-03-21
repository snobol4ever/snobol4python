# snobol4python

**SNOBOL4 pattern matching for Python — on PyPI**

[![License: LGPL v3](https://img.shields.io/badge/License-LGPL%20v3-blue.svg)](https://www.gnu.org/licenses/lgpl-3.0)
[![PyPI version](https://img.shields.io/badge/pypi-v0.5.0-blue)](https://pypi.org/project/SNOBOL4python/)

> *Part of the [snobol4ever](https://github.com/snobol4ever) project — SNOBOL4 everywhere, on every platform.*

---

## What This Is

snobol4python brings the full SNOBOL4 pattern vocabulary to Python as a first-class library. Not a regex wrapper. Not a translation layer. The real SNOBOL4 pattern engine — composable, backtracking, recursive — available as Python objects you build, store, pass to functions, and combine at runtime.

SNOBOL4 patterns are values. They compose. They backtrack. They capture intermediate results. They reference themselves recursively. They can express BNF grammars directly — something regular expressions cannot do. A SNOBOL4 pattern is not a string. It is not syntax. It is a data structure that encodes a computation.

```bash
pip install SNOBOL4python
```

---

## Quick Start

```python
from snobol4 import *

# Match a simple pattern
subject = "Hello, World"
pat = LIT("Hello") + SPAN(",! ") + REM

result = match(subject, pat)
# → matched: "Hello, World"

# Capture with immediate assign
name = SV()
pat = LIT("Hello, ") + REM $ name
match("Hello, World", pat)
# name.value → "World"

# Compose patterns — SNOBOL4's superpower
digit = ANY("0123456789")
digits = digit + ARBNO(digit)
integer = FENCE(digits)  # no backtracking past this point

# Recursive grammar — impossible in regex
open_p  = LIT("(")
close_p = LIT(")")
balanced = DEFERRED(lambda: open_p + ARBNO(balanced | ANY("abc")) + close_p)
match("(a(b)c)", balanced)  # ✅
```

---

## The Pattern Vocabulary

Full SNOBOL4/SPITBOL primitive set:

| Primitive | Matches |
|-----------|---------|
| `LIT(s)` | Literal string `s` |
| `ANY(s)` | Any single character in `s` |
| `NOTANY(s)` | Any single character not in `s` |
| `SPAN(s)` | Longest run of characters in `s` |
| `BREAK(s)` | Longest run of characters not in `s` |
| `BREAKX(s)` | Like BREAK but resumes on backtrack |
| `ARB` | Any string (shortest to longest on backtrack) |
| `ARBNO(p)` | Zero or more repetitions of pattern `p` |
| `BAL` | Balanced parentheses |
| `LEN(n)` | Exactly `n` characters |
| `POS(n)` | Cursor at position `n` from left |
| `RPOS(n)` | Cursor at position `n` from right |
| `TAB(n)` | Advance cursor to position `n` |
| `RTAB(n)` | Advance cursor to position `n` from right |
| `REM` | Remainder of subject |
| `FENCE` | Prevent backtracking past this point |
| `FENCE(p)` | Match `p` without backtracking |
| `ABORT` | Immediately fail the entire match |
| `FAIL` | Always fail (force backtracking) |
| `SUCCEED` | Always succeed (loop until exhausted) |
| `DEFERRED(fn)` | Recursive patterns via lambda |

Composition operators:

| Operator | Meaning |
|----------|---------|
| `p1 + p2` | Sequential: p1 then p2 |
| `p1 \| p2` | Alternation: try p1, then p2 |
| `p $ var` | Immediate assign: capture on each match |
| `p . var` | Conditional assign: capture on final success |
| `p @ n` | Cursor capture: record cursor position |

---

## Dual Backend

snobol4python ships with two backends. The correct one is selected automatically.

**C extension backend** (default when available) — wraps Phil Budne's SPIPAT engine, the pattern matching core from CSNOBOL4 2.3.3. Compiled C code with zero Python overhead on the hot path. **7–11× faster** than the pure-Python backend for complex patterns.

**Pure-Python backend** (fallback) — a complete Python implementation of the Byrd Box engine. No compilation required. Runs anywhere Python runs. Correct on all test cases.

```python
from snobol4 import backend
print(backend)  # → "c_extension" or "pure_python"
```

---

## Shift-Reduce Parser Stack

snobol4python includes a shift-reduce parse-tree construction mechanism that runs *inside* the pattern match. Instead of extracting strings and parsing them separately, you build your grammar as SNOBOL4 patterns and accumulate parse tree nodes during the match itself.

```python
from snobol4.parser import shift, reduce, get_tree

# shift(p, tag) — match p and push (tag, matched_text) onto the stack
# reduce(tag, n) — pop n items, emit a tree node tagged with tag

expr = shift(integer, "NUM") + ARBNO(
    shift(ANY("+-"), "OP") + shift(integer, "NUM") + reduce("BINOP", 3)
)
match("1+2+3", expr)
tree = get_tree()
# → ("BINOP", ("BINOP", ("NUM","1"), ("+"), ("NUM","2")), ("+"), ("NUM","3"))
```

This is the mechanism used to build the Penn Treebank parser and the CLAWS5 NLP corpus parser in snobol4csharp — ported from C# back to Python with the same interface.

---

## snobol4artifact — The C Extension

[snobol4artifact](https://github.com/snobol4ever/snobol4artifact) is the CPython C extension that powers the fast backend. It runs snobol4python pattern trees through a full Byrd Box engine in C — the direct ancestor of `engine.c` in snobol4x.

```bash
pip install snobol4artifact  # installs the C extension separately if needed
```

**70+ tests / 0 failures.** The extension is tested independently against the pure-Python backend — every result must match exactly.

The architecture: snobol4python builds a pattern tree (Python objects). snobol4artifact walks that tree through the four-port Byrd Box engine in C: α (proceed), β (recede), γ (succeed), ω (concede). When the C extension is installed, snobol4python delegates the match call to it transparently.

---

## What You Can Build

The shift-reduce stack and recursive patterns together make snobol4python a complete parsing toolkit:

**Recursive grammars** — Any context-free grammar expressible as SNOBOL4 patterns. Mutual recursion between patterns via `DEFERRED`. No yacc. No lex. No separate grammar formalism.

**Context-sensitive grammars** — Patterns that inspect state accumulated during earlier parts of the match. Things PCRE and PEG parsers cannot express.

**NLP pipelines** — CLAWS5 part-of-speech tagging corpus parsing, Penn Treebank parenthesized-tree extraction, and Porter Stemmer validation (23,531-word corpus) are all tested in snobol4csharp using the same pattern vocabulary.

**SNOBOL4 source parsing** — The pattern vocabulary is expressive enough to parse SNOBOL4 source code itself. This is tested in snobol4csharp's `Tests_Snobol4Parser` suite and is the basis for the `beauty.sno` port.

---

## Version

**v0.5.0** — stable, on PyPI.

---

## Relationship to snobol4ever

snobol4python is the Python library arm of the [snobol4ever](https://github.com/snobol4ever) project. The full SNOBOL4/SPITBOL *language* (not just pattern matching) is available on the JVM via [snobol4jvm](https://github.com/snobol4ever/snobol4jvm) and on .NET via [snobol4dotnet](https://github.com/snobol4ever/snobol4dotnet). The native compiler targeting x86-64 ASM, JVM bytecode, and .NET MSIL simultaneously is [snobol4x](https://github.com/snobol4ever/snobol4x).

---

## License

LGPL v3. See [LICENSE](LICENSE).

---

## Acknowledgments

**Ralph Griswold, Ivan Polonsky, David Farber** — SNOBOL4, Bell Labs, 1962–1967.

**Phil Budne** — CSNOBOL4 and the SPIPAT engine that powers the C extension backend.

**Lon Jones Cherryholmes** — architecture and snobol4python author.

---

*snobol4all. snobol4now. snobol4ever.*
