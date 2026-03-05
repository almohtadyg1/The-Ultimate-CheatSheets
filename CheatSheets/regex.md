# Regex Cheat Sheet
### From Zero to Expert

---

## What is Regex?

A **Regular Expression** (regex) is a pattern that describes a set of strings. It lets you search, match, extract, and replace text with surgical precision — in virtually every programming language and tool.

```
Pattern:   \b[A-Z][a-z]+\b
Matches:   "Hello"  "World"  "Cairo"
No match:  "hello"  "123"    "UPPER"
```

---

## The Absolute Basics

| Pattern | Matches |
|---|---|
| `abc` | Literal string `abc` |
| `.` | **Any** single character except newline |
| `\.` | A literal dot (escaped) |
| `^` | Start of string/line |
| `$` | End of string/line |
| `\n` | Newline |
| `\t` | Tab |

> **Escape rule:** Special characters `. ^ $ * + ? { } [ ] \ | ( )` must be escaped with `\` to match literally.

---

## Character Classes

| Pattern | Matches |
|---|---|
| `[abc]` | `a`, `b`, or `c` |
| `[^abc]` | Any character **except** `a`, `b`, `c` |
| `[a-z]` | Any lowercase letter |
| `[A-Z]` | Any uppercase letter |
| `[0-9]` | Any digit |
| `[a-zA-Z0-9]` | Any alphanumeric character |

### Shorthand Classes

| Shorthand | Equivalent | Meaning |
|---|---|---|
| `\d` | `[0-9]` | Digit |
| `\D` | `[^0-9]` | Non-digit |
| `\w` | `[a-zA-Z0-9_]` | Word character |
| `\W` | `[^a-zA-Z0-9_]` | Non-word character |
| `\s` | `[ \t\n\r\f\v]` | Whitespace |
| `\S` | `[^ \t\n\r\f\v]` | Non-whitespace |

---

## Quantifiers

Quantifiers control **how many times** the preceding element repeats.

| Quantifier | Meaning |
|---|---|
| `*` | 0 or more |
| `+` | 1 or more |
| `?` | 0 or 1 (optional) |
| `{n}` | Exactly n times |
| `{n,}` | n or more times |
| `{n,m}` | Between n and m times (inclusive) |

```
\d+        → "42"  "007"  "1"
colou?r    → "color"  "colour"
\w{3,5}    → "cat"  "hello"  "world"  (not "hi" or "toolong")
```

### Greedy vs Lazy

By default, quantifiers are **greedy** — they match as much as possible.  
Add `?` to make them **lazy** — match as little as possible.

```
Input:    <b>bold</b> and <i>italic</i>

<.+>      (greedy)  → "<b>bold</b> and <i>italic</i>"   ← entire string
<.+?>     (lazy)    → "<b>"  then  "</b>"  then  "<i>"  then  "</i>"
```

---

## Anchors & Boundaries

| Pattern | Meaning |
|---|---|
| `^` | Start of string (or line in multiline mode) |
| `$` | End of string (or line in multiline mode) |
| `\b` | Word boundary (between `\w` and `\W`) |
| `\B` | Not a word boundary |
| `\A` | Start of string (never changes in multiline) |
| `\Z` | End of string (never changes in multiline) |

```
\bcat\b    → matches "cat" in "the cat sat"  but NOT in "concatenate"
^\d+$      → matches strings that are entirely digits: "42" but not "42abc"
```

---

## Groups & Alternation

### Capturing Groups `( )`
Groups a sub-pattern **and captures** the matched text for later use.

```
(\d{4})-(\d{2})-(\d{2})    matches "2025-03-05"
                             Group 1: "2025"
                             Group 2: "03"
                             Group 3: "05"
```

### Non-Capturing Groups `(?: )`
Groups without capturing. Use when you need grouping but don't need the value.

```
(?:https?|ftp)://    matches "http://" or "https://" or "ftp://"
```

### Alternation `|`
Acts like a logical OR between patterns.

```
cat|dog        → "cat" or "dog"
gr(a|e)y       → "gray" or "grey"
(jpg|jpeg|png) → any of the three image extensions
```

---

## Backreferences

Refer to a previously captured group **within the same pattern**.

```
([a-z])\1        → matches doubled letters: "aa", "bb", "zz"
<(\w+)>.*?</\1>  → matches valid HTML tags: <div>content</div>
```

Named backreference: `(?P=name)` (Python) or `\k<name>` (JS/.NET)

---

## Lookahead & Lookbehind (Lookarounds)

Match a pattern only if it **is** (or **isn't**) preceded/followed by another — without consuming those characters.

| Syntax | Type | Meaning |
|---|---|---|
| `(?=...)` | Positive lookahead | Followed by... |
| `(?!...)` | Negative lookahead | NOT followed by... |
| `(?<=...)` | Positive lookbehind | Preceded by... |
| `(?<!...)` | Negative lookbehind | NOT preceded by... |

```
\d+(?= dollars)     → matches "100" in "100 dollars" but not in "100 euros"
\b\w+(?!ing)\b      → matches words NOT ending in "ing"
(?<=@)\w+           → matches domain in "user@gmail" → "gmail"
(?<!\$)\d+          → matches digits NOT preceded by "$"
```

> Lookarounds **assert** a condition — they match a position, not characters.

---

## Flags / Modifiers

Flags change how the entire pattern behaves.

| Flag | Symbol | Effect |
|---|---|---|
| Case-insensitive | `i` | `Hello` matches `hello`, `HELLO` |
| Multiline | `m` | `^` and `$` match each line, not just start/end of whole string |
| Dotall | `s` | `.` also matches newline characters |
| Global | `g` | Find all matches, not just the first |
| Extended | `x` | Allows whitespace and `#` comments inside the pattern |
| Unicode | `u` | Enable full Unicode support |

```
/hello/i           → matches "Hello", "HELLO", "hElLo"
/^item/m           → matches "item" at the start of any line
/start.+end/s      → matches across multiple lines
```

---

## Named Groups

Give a group a **meaningful name** instead of a number.

```python
# Python syntax:
(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})

match.group('year')   → "2025"
match.group('month')  → "03"
```

```js
// JavaScript syntax:
(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})

match.groups.year   → "2025"
```

---

## Common Practical Patterns

```
# Email (simplified but robust)
^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$

# URL
https?://[\w\-]+(\.[\w\-]+)+([/\w\-.?=%&+#]*)?

# IPv4 address
\b(\d{1,3}\.){3}\d{1,3}\b

# Date (YYYY-MM-DD)
\b\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])\b

# Time (HH:MM or HH:MM:SS)
\b([01]\d|2[0-3]):[0-5]\d(:[0-5]\d)?\b

# Hex color
#([0-9a-fA-F]{3}|[0-9a-fA-F]{6})\b

# Strong password (min 8 chars, upper, lower, digit, special)
^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[\W_]).{8,}$

# Slug (URL-friendly string)
^[a-z0-9]+(?:-[a-z0-9]+)*$

# Quoted string (handles escaped quotes)
"([^"\\]|\\.)*"
```

---

## Regex in Code

### Python
```python
import re

re.search(r'\d+', 'abc 42 def')       # first match anywhere
re.match(r'\d+', '42 abc')            # match at start only
re.fullmatch(r'\d+', '42')            # must match entire string
re.findall(r'\d+', 'a1 b2 c3')        # → ['1', '2', '3']
re.sub(r'\s+', '-', 'hello world')    # → 'hello-world'
re.split(r'[,;]\s*', 'a, b; c')       # → ['a', 'b', 'c']

# Compiled pattern (faster when reused):
pattern = re.compile(r'(\w+)@(\w+)', re.IGNORECASE)
```

### JavaScript
```js
const str = 'Price: $42.99';

str.match(/\d+\.\d+/);                // first match
str.match(/\d+/g);                    // all matches → ['42', '99']
str.replace(/\$(\d+)/, 'USD $1');     // backreference in replacement
str.split(/\s*,\s*/);                 // split on comma with optional spaces
/^\d+$/.test('123');                  // → true
```

### Common Language Syntax

| Language | Raw pattern | Flags |
|---|---|---|
| Python | `re.compile(r'pattern', re.I)` | `re.I re.M re.S` |
| JavaScript | `/pattern/flags` | `i g m s u` |
| Java | `Pattern.compile("pattern", Pattern.CASE_INSENSITIVE)` | constants |
| Go | `regexp.MustCompile(`pattern`)` | inline `(?i)` |
| Rust | `Regex::new(r"pattern")` | inline `(?i)` |

---

## How Regex Works Internally

Understanding this helps you write **better and faster** patterns.

1. **NFA (Nondeterministic Finite Automaton)** — Most regex engines (Perl, Python, JS) use NFA-based backtracking. Flexible but can be slow on pathological patterns.
2. **DFA (Deterministic Finite Automaton)** — Tools like `grep`, `awk`, `re2` compile to DFA. Guaranteed O(n) time but no backreferences.
3. **Backtracking** — When a match fails, the engine backs up and tries alternatives. This enables lookaheads and backreferences but risks catastrophic backtracking.

### Catastrophic Backtracking — Avoid This
```
Pattern: (a+)+$
Input:   "aaaaaaaaab"
```
This causes **exponential** backtracking. The nested quantifier creates an explosion of possible groupings to try.

**Fix:** Use atomic groups `(?>...)` or possessive quantifiers `++`, `*+` where available, or restructure the pattern.

---

## Expert Tips

**Prefer specific over general.**  
`\d{4}` is faster and safer than `.{4}` — it fails fast when input doesn't match.

**Anchor when you can.**  
Starting a pattern with `^` or `\b` lets the engine skip impossible positions immediately.

**Use non-capturing groups `(?:)` by default.**  
Capturing groups have overhead. Only use `()` when you actually need the captured value.

**Pre-compile patterns used in loops.**  
`re.compile()` in Python, `new RegExp()` stored in a variable in JS — avoids recompiling on every iteration.

**Don't parse structured formats with regex alone.**  
HTML, JSON, and XML have nesting that CFGs handle but regex cannot reliably. Use a proper parser for those; use regex for extracting flat fields within them.

**Test with both matching and non-matching cases.**  
A pattern that matches everything you want but also matches what you don't want is broken.

---

## Quick Reference Card

```
CHARACTERS          QUANTIFIERS         ANCHORS
.   any char         *    0 or more      ^   start
\d  digit            +    1 or more      $   end
\w  word char        ?    0 or 1         \b  word boundary
\s  whitespace       {n}  exactly n      \A  string start
\D  non-digit        {n,} n or more      \Z  string end
\W  non-word char    {n,m} n to m
\S  non-whitespace   *?   lazy 0+        GROUPS
                     +?   lazy 1+        (x)    capture
CLASSES              ??   lazy 0 or 1    (?:x)  non-capture
[abc]  a, b, or c                        (?=x)  lookahead
[^abc] not a/b/c    FLAGS                (?!x)  neg. lookahead
[a-z]  range         i   insensitive     (?<=x) lookbehind
                     g   global          (?<!x) neg. lookbehind
SPECIAL              m   multiline       (?P<n>x) named group
|  alternation       s   dotall          \1  backreference
\  escape            x   verbose
```
