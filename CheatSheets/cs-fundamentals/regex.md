# Regular Expressions: A Complete Progressive Tutorial

---

## 1. What & Why

A regular expression (regex) is a pattern written in a formal mini-language that describes a set of strings. You write the pattern once, and the regex engine tests whether any string matches it — then tells you where, and optionally replaces or extracts parts of it.

Why does this exist? Because searching text with plain string matching breaks the moment your inputs vary. You can check if a string equals `"2024-01-15"` easily. But checking if a string *looks like* a valid date in that format — any year, any valid month, any valid day — requires logic. Regex encodes that logic as a compact pattern rather than dozens of `if` statements.

Where you will use this in practice: validating form input (emails, phone numbers), parsing log files, finding and replacing code patterns in bulk, extracting structured data from unstructured text, writing URL routers, building tokenizers and linters.

Every major programming language has regex support built in. The syntax is 95% identical across Python, JavaScript, Java, Go, Ruby, Rust, and most CLI tools like `grep`, `sed`, and `awk`. Learn it once, use it everywhere.

---

## 2. Mental Model

Think of a regex pattern as a series of filters applied left to right, position by position.

The engine places a cursor at position 0 of your input string and asks: "Does the pattern match starting here?" If yes, it reports the match. If no, it advances the cursor one position and tries again.

```
Input:  "The price is $42.99 today"
Pattern: \$\d+\.\d+

Cursor at 'T' -> no match
Cursor at 'h' -> no match
... (skipping ahead) ...
Cursor at '$' -> tries to match \$ -> YES
             -> tries \d+ -> "42" -> YES
             -> tries \. -> "." -> YES
             -> tries \d+ -> "99" -> YES
             -> MATCH FOUND: "$42.99"
```

Each element in the pattern consumes characters from the input. When all elements consume successfully in sequence, you have a match. When any element fails, the engine backtracks and tries the next starting position.

Think of it as a lock with multiple tumblers that must all align: each part of the pattern is one tumbler, and the input is the key being tested.

---

## 3. Progressive Examples

### Level 1: Literal Matching and the Dot

The simplest regex is a plain string. `hello` matches the literal sequence h-e-l-l-o.

The first metacharacter to know is `.` (dot), which matches any single character except a newline.

```python
import re

text = "The cat sat on the mat"

# Literal match
print(re.findall("cat", text))      # ['cat']

# Dot matches any character
print(re.findall("c.t", text))      # ['cat'] — 'c', any char, 't'
print(re.findall(".at", text))      # ['cat', 'sat', 'mat'] — any char before 'at'

# To match a literal dot, escape it with backslash
version = "Python 3.12 and 3.11"
print(re.findall(r"3\.\d+", version))   # ['3.12', '3.11']
# Without escape: re.findall("3.\d+", version) would also match "3X12"
```

Note the `r` prefix on string literals: this is a raw string, which means Python does not interpret backslashes. Without it, `\d` becomes a real tab character before the regex engine ever sees it. Always use raw strings for regex patterns.

### Level 2: Character Classes and Quantifiers

A character class `[...]` matches exactly one character from a defined set. Quantifiers say how many times the preceding element must appear.

```python
import re

log = "2024-01-15 ERROR: connection failed after 30 retries"

# [0-9] matches any digit — same as \d
print(re.findall(r"[0-9]+", log))
# ['2024', '01', '15', '30']

# \d is shorthand for [0-9]
# \w is shorthand for [a-zA-Z0-9_]
# \s is shorthand for whitespace (space, tab, newline)

# Quantifiers:
# +  means "one or more"
# *  means "zero or more"
# ?  means "zero or one" (makes the element optional)
# {n,m} means "between n and m times"

# Extract the date portion
print(re.findall(r"\d{4}-\d{2}-\d{2}", log))
# ['2024-01-15']

# Match the word "ERROR" or "WARN" or "INFO"
print(re.findall(r"ERROR|WARN|INFO", log))
# ['ERROR']

# Find any word (sequence of word characters)
print(re.findall(r"\w+", "hello_world foo-bar"))
# ['hello_world', 'foo', 'bar']
# Note: hyphen is not a word character, so "foo-bar" splits at the hyphen
```

### Level 3: Anchors, Groups, and Capture

Anchors assert a position in the string without consuming characters. Groups let you capture submatches and apply quantifiers to multi-character sequences.

```python
import re

emails = """
valid: user@example.com
also valid: first.last+tag@company.co.uk
invalid: @nodomain.com
invalid: user@
"""

# ^ anchors to start of string, $ anchors to end
# Without anchors, the pattern can match anywhere in the string
print(re.match(r"\d+", "42 apples"))    # matches "42" at start
print(re.match(r"\d+", "I have 42"))    # None — no digits at start

# Capturing groups with ()
# This pattern captures year, month, day separately
date_pattern = r"(\d{4})-(\d{2})-(\d{2})"
match = re.search(date_pattern, "Logged on 2024-01-15 at noon")

if match:
    print(match.group(0))   # '2024-01-15'  — the full match
    print(match.group(1))   # '2024'        — first group
    print(match.group(2))   # '01'          — second group
    print(match.group(3))   # '15'          — third group

# Named groups — cleaner than numbered groups for complex patterns
date_pattern_named = r"(?P<year>\d{4})-(?P<month>\d{2})-(?P<day>\d{2})"
match = re.search(date_pattern_named, "2024-01-15")
print(match.group("year"))   # '2024'
print(match.group("month"))  # '01'

# findall with groups returns tuples of group values
matches = re.findall(r"(\w+)@(\w[\w.]+)", emails)
for local, domain in matches:
    print(f"  user: {local}, domain: {domain}")
# user: user, domain: example.com
# user: last+tag, domain: company.co.uk
```

### Level 4: Greedy vs. Lazy, Lookarounds

This is where most intermediate users make mistakes. Understanding greedy vs. lazy matching is essential for extracting the right content from nested or repeated structures.

```python
import re

html = "<b>bold</b> and <i>italic</i>"

# Greedy: .+ matches as MUCH as possible
greedy = re.findall(r"<.+>", html)
print(greedy)   # ['<b>bold</b> and <i>italic</i>']
# The engine matched from the first < to the LAST > in the entire string

# Lazy: .+? matches as LITTLE as possible
lazy = re.findall(r"<.+?>", html)
print(lazy)     # ['<b>', '</b>', '<i>', '</i>']
# Now it stops at the first > it finds after each <

# Lookahead: match X only if followed by Y (Y is not consumed)
prices = "apple $1.50, orange $2.00, grape $0.75"
# Find amounts only when preceded by $
amounts = re.findall(r"(?<=\$)\d+\.\d+", prices)
print(amounts)   # ['1.50', '2.00', '0.75']

# Negative lookahead: match X only if NOT followed by Y
words = "color colour honor honour"
# British spellings end in "our" — match American spellings
american = re.findall(r"\b\w+(?<!our)\b", words)
print(american)  # ['color', 'honor']

# Real-world: extract JSON values for a known key
data = '{"name": "Alice", "age": 30, "city": "Cairo"}'
# Capture value after "name":
name_match = re.search(r'"name":\s*"([^"]+)"', data)
print(name_match.group(1))   # 'Alice'
# [^"]+ means "one or more characters that are NOT a quote"
# This is safer than .+? for string content
```

### Level 5: Flags, Substitution, and Real-World Patterns

```python
import re

# --- Flags ---
# re.IGNORECASE (re.I): case-insensitive matching
print(re.findall(r"error", "Error: FATAL ERROR occurred", re.I))
# ['Error', 'ERROR']

# re.MULTILINE (re.M): ^ and $ match start/end of each LINE
log_lines = """ERROR: disk full
INFO: backup started
ERROR: network timeout"""
errors = re.findall(r"^ERROR:.+", log_lines, re.M)
print(errors)
# ['ERROR: disk full', 'ERROR: network timeout']

# re.DOTALL (re.S): dot matches newlines too
html_block = "<div>\n  content\n</div>"
match = re.search(r"<div>(.+?)</div>", html_block, re.S)
print(match.group(1))   # '\n  content\n'

# --- Substitution ---
text = "  too   many    spaces  "
# Replace one or more whitespace chars with a single space
cleaned = re.sub(r"\s+", " ", text).strip()
print(cleaned)   # 'too many spaces'

# Use backreferences in replacement strings
# \1 refers to the first captured group
dates = "Today is 2024-01-15 and tomorrow is 2024-01-16"
reformatted = re.sub(r"(\d{4})-(\d{2})-(\d{2})", r"\3/\2/\1", dates)
print(reformatted)   # 'Today is 15/01/2024 and tomorrow is 16/01/2024'

# Use a function as replacement for complex transforms
def mask_credit_card(match):
    number = match.group(0)
    return "X" * (len(number) - 4) + number[-4:]

text = "Card: 4532015112830366, backup: 5425233430109903"
masked = re.sub(r"\b\d{16}\b", mask_credit_card, text)
print(masked)   # 'Card: XXXXXXXXXXXX0366, backup: XXXXXXXXXXXX9903'

# --- Compiled patterns: use when the same pattern runs in a loop ---
email_re = re.compile(
    r"^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$"
)

test_emails = [
    "user@example.com",    # valid
    "bad@",                # invalid
    "no-at-sign.com",      # invalid
    "first.last@co.uk",    # valid
]

for email in test_emails:
    valid = bool(email_re.match(email))
    print(f"{'VALID' if valid else 'INVALID'}: {email}")
```

### Level 6: JavaScript Regex (The Differences That Matter)

```javascript
// JavaScript regex literals: /pattern/flags
// No r prefix needed — backslashes work correctly inside /.../

const logLine = "2024-01-15 ERROR connection refused (retry 3 of 5)";

// .test() — returns boolean
const hasError = /ERROR|WARN/.test(logLine);  // true

// .match() — returns first match (or all with /g)
const numbers = logLine.match(/\d+/g);   // ['2024', '01', '15', '3', '5']

// Named groups with destructuring
const datePattern = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const { groups: { year, month, day } } = logLine.match(datePattern);
console.log(year, month, day);   // '2024' '01' '15'

// .replace() with a function
const sanitized = logLine.replace(/\d+/g, (match) => {
    return match.length >= 4 ? match : "N";  // preserve years, mask small numbers
});
// '2024-N-N ERROR connection refused (retry N of N)'

// String.matchAll() — returns an iterator of all matches with groups
const csv = "alice,30,engineer\nbob,25,designer";
const rowPattern = /(?<name>\w+),(?<age>\d+),(?<role>\w+)/g;

for (const match of csv.matchAll(rowPattern)) {
    const { name, age, role } = match.groups;
    console.log(`${name} is a ${age}-year-old ${role}`);
}
// alice is a 30-year-old engineer
// bob is a 25-year-old designer
```

---

## 4. Common Mistakes & Misconceptions

**Mistake 1: Forgetting to escape the dot**

```python
# WRONG: matches "3X12", "3512", "3.12" — dot means ANY character
re.findall("3.12", "Python 3.12 and 3X12")   # ['3.12', '3X12']

# CORRECT: escape the dot to match a literal period
re.findall(r"3\.12", "Python 3.12 and 3X12")   # ['3.12']
```

**Mistake 2: Greedy matching consumes more than intended**

```python
# Trying to extract each HTML tag
html = "<b>bold</b><i>italic</i>"

# WRONG: greedy — matches from first < to last >
re.findall(r"<.+>", html)     # ['<b>bold</b><i>italic</i>']

# CORRECT: lazy — stops at the first >
re.findall(r"<.+?>", html)    # ['<b>', '</b>', '<i>', '</i>']

# BETTER: use a negated class — explicitly exclude >
re.findall(r"<[^>]+>", html)  # ['<b>', '</b>', '<i>', '</i>']
# [^>]+ means "one or more characters that are not >"
# This is clearer in intent and slightly faster
```

**Mistake 3: Using `re.match` when you mean `re.search`**

```python
text = "The price is $42"

# re.match only checks at the START of the string
print(re.match(r"\$\d+", text))     # None — no $ at position 0

# re.search checks everywhere in the string
print(re.search(r"\$\d+", text))    # <Match object: '$42'>
```

**Mistake 4: Forgetting the global flag in JavaScript**

```javascript
const text = "cat and dog and bird";

// Without /g — only finds the FIRST match
text.match(/\w+/)   // ['cat']

// With /g — finds ALL matches
text.match(/\w+/g)  // ['cat', 'and', 'dog', 'and', 'bird']

// Important: when using /g, .match() does NOT return groups
// Use .matchAll() instead if you need groups with global matching
```

**Mistake 5: Using regex to parse HTML or JSON**

```python
# WRONG: this will fail on nested tags, self-closing tags, attributes...
html = '<a href="https://example.com">Click here</a>'
re.findall(r"<a>(.+?)</a>", html)   # [] — doesn't account for attributes

# CORRECT: use a proper parser
from bs4 import BeautifulSoup
soup = BeautifulSoup(html, "html.parser")
print(soup.find("a").text)   # 'Click here'

# Same principle for JSON: use json.loads(), not regex
```

**Mistake 6: Catastrophic backtracking on pathological patterns**

```python
import re, time

# This pattern has nested quantifiers — exponential backtracking
bad_pattern = r"(a+)+"

# Works fine on short inputs
re.match(bad_pattern, "aaaaaaaaab")  # OK

# On longer inputs, this can hang for seconds or minutes
# In a web server, this is a ReDoS (Regular Expression Denial of Service) attack
# Example: re.match(r"(a+)+$", "a" * 30 + "b")  # DO NOT RUN in production

# Fix: eliminate nested quantifiers, use atomic groups where available,
# or use a linear-time engine like Google's re2
```

---

## 5. The "Why Does This Work" Layer

### How the NFA Engine Backtracks

Most regex engines (Python, JavaScript, Java, Ruby) use an NFA — Nondeterministic Finite Automaton — with backtracking. Here is what actually happens when you run `re.search(r"a+b", "aaac")`:

```
Input:  a a a c
        ^
Pattern position 0: 'a+'

Step 1: 'a+' is greedy. It consumes ALL the a's: positions 0,1,2.
        Cursor is now at position 3 ('c').

Step 2: Pattern needs 'b'. Input has 'c'. FAIL.
        Engine backtracks: gives back one 'a'. Cursor at position 2.

Step 3: Pattern needs 'b'. Input has 'a'. FAIL.
        Engine backtracks again. Cursor at position 1.

Step 4: Pattern needs 'b'. Input has 'a'. FAIL.
        Engine backtracks again. Cursor at position 0.
        
Step 5: 'a+' has given back everything — it matched 0 times now.
        But 'a+' requires at LEAST 1 match. FAIL.

Step 6: Engine advances starting position to 1 ('a'). Repeats.
        ... eventually exhausts all positions. No match found.
```

This backtracking is what allows powerful features like lookaheads and backreferences. The cost is that pathological patterns (nested quantifiers, alternation with overlap) cause exponential time complexity.

DFA-based engines (used in `grep`, `awk`, Google's RE2, Rust's `regex` crate) compile the pattern to a state machine that traverses the input exactly once — guaranteed O(n) — but cannot support backreferences or lookarounds.

### Why Word Boundaries Work the Way They Do

`\b` is not a character — it is a zero-width assertion. It matches the position between a `\w` character and a `\W` character (or string start/end). This is why:

```python
re.findall(r"\bcat\b", "the cat concatenate")
# ['cat'] — matches 'cat' between spaces, NOT inside 'concatenate'

# At position 4 (the 'c' in 'cat'):
#   character to the left: ' ' (space) — which is \W
#   character at cursor: 'c' — which is \w
#   \b asserts: "there is a transition from \W to \w here" -> YES

# At position 10 (the 'c' in 'concatenate'):
#   character to the left: ' ' — \W  -> a boundary exists
#   BUT at the end of 'cat' inside 'concatenate':
#   character after 'cat': 'e' — which is \w
#   so \b at end of 'cat' would require \w -> \W transition, but it's \w -> \w
#   -> NO boundary -> no match
```

### Why `[^>]+` Is Better Than `.+?` for Parsing Tags

```
Pattern 1: <.+?>
Pattern 2: <[^>]+>

For input: <a href="page.html">

Pattern 1 (.+? lazy):
  - Matches '<', then tries the shortest possible .+?
  - '<a' — tries to match '>' — 'space' is not '>', keep going
  - Eventually finds the '>' at the end. Works. But the engine
    does MORE backtracking attempts along the way.

Pattern 2 ([^>]+ negated class):
  - Matches '<', then [^>]+ consumes ALL non-> characters in one pass
  - Hits '>' — stops immediately. No backtracking needed.
  - Faster and more explicit about intent.
```

---

## 6. Quick Reference

### Metacharacters

| Pattern | Meaning |
|---------|---------|
| `.` | Any character except newline |
| `\d` | Digit `[0-9]` |
| `\D` | Non-digit `[^0-9]` |
| `\w` | Word character `[a-zA-Z0-9_]` |
| `\W` | Non-word character |
| `\s` | Whitespace (space, tab, newline, etc.) |
| `\S` | Non-whitespace |
| `\b` | Word boundary |
| `\B` | Non-word boundary |
| `^` | Start of string (or line with `re.M`) |
| `$` | End of string (or line with `re.M`) |

### Quantifiers

| Pattern | Meaning |
|---------|---------|
| `*` | 0 or more (greedy) |
| `+` | 1 or more (greedy) |
| `?` | 0 or 1 (greedy) |
| `{n}` | Exactly n |
| `{n,m}` | Between n and m |
| `*?`, `+?`, `??` | Lazy versions of above |

### Groups and Lookarounds

| Pattern | Meaning |
|---------|---------|
| `(x)` | Capturing group |
| `(?:x)` | Non-capturing group |
| `(?P<name>x)` | Named group (Python) |
| `(?<name>x)` | Named group (JavaScript) |
| `\1`, `\2` | Backreference to group 1, 2 |
| `(?=x)` | Positive lookahead |
| `(?!x)` | Negative lookahead |
| `(?<=x)` | Positive lookbehind |
| `(?<!x)` | Negative lookbehind |

### Python API

```python
re.search(pattern, string)       # First match anywhere
re.match(pattern, string)        # Match at start only
re.fullmatch(pattern, string)    # Entire string must match
re.findall(pattern, string)      # All matches as list
re.finditer(pattern, string)     # All matches as iterator
re.sub(pattern, repl, string)    # Replace matches
re.split(pattern, string)        # Split on matches
re.compile(pattern, flags)       # Pre-compile for reuse
```

### Flags

| Python | JavaScript | Meaning |
|--------|-----------|---------|
| `re.I` | `i` | Case-insensitive |
| `re.M` | `m` | `^`/`$` match line boundaries |
| `re.S` | `s` | Dot matches newline |
| `re.X` | `x` | Allow whitespace/comments in pattern |
| — | `g` | Find all matches (not just first) |

### Production-Ready Patterns

```python
# Email (good enough for 99% of cases)
r"^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$"

# ISO date YYYY-MM-DD
r"\b\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])\b"

# URL (http/https)
r"https?://[\w\-]+(\.[\w\-]+)+([/\w\-.?=%&+#]*)?"

# IPv4 address
r"\b((25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(25[0-5]|2[0-4]\d|[01]?\d\d?)\b"

# Strong password (8+ chars, upper, lower, digit, special)
r"^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[\W_]).{8,}$"

# Quoted string (handles escaped quotes)
r'"([^"\\]|\\.)*"'

# Slug (lowercase, hyphens only)
r"^[a-z0-9]+(?:-[a-z0-9]+)*$"

# Hex color
r"^#([0-9a-fA-F]{3}|[0-9a-fA-F]{6})$"
```
