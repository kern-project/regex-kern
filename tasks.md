# regex-kern Tasks

`regex-kern` should become the default Kern package for byte-oriented regular
expression matching. The first implementation must be small enough to audit,
predictable enough for tooling and code generation, and idiomatic enough to be
used as a model package.

## Priority 0: Package Shape And Contracts

- [x] Create a standalone Craft package named `regex` with `src/lib.rn`, focused
  tests, README, MIT license, and no generated public API.
- [x] Keep the public API fluent and object-oriented:
  - `regex.compile(pattern, alloc)` creates an owned reusable `Regex`.
  - `pattern.compile_regex(alloc)` is the fluent compile path for pattern
    handles.
  - `regex.matches(pattern, text, alloc)` is a convenient one-shot path.
  - `re.matches(text, alloc)`, `re.find(text, alloc)`, and
    `re.find_all(text, alloc)` operate from the compiled handle.
  - `Match` carries byte offsets and borrowed text accessors.
- [x] Use explicit app-style examples in docs when allocation or parse errors are
  involved; do not teach `else return 1` as core library flow.
- [x] Add compile-only coverage for README-shaped examples so docs and API cannot
  drift apart.

## Priority 1: Safe Core Engine

- Parse into a compact AST for a byte regex grammar:
  - literals and escaped metacharacters
  - `.`
  - character classes `[abc]`, ranges `[a-z]`, negated classes `[^...]`
  - concatenation and alternation `|`
  - grouping `(...)`
  - quantifiers `*`, `+`, `?`, and `{m,n}`
  - anchors `^` and `$`
- Compile the AST into a Thompson-style NFA, not a recursive backtracking
  engine. Matching must avoid exponential behavior.
- Keep the first engine byte-oriented. Unicode classes and grapheme semantics
  should be planned explicitly later instead of guessed.
- Return structured errors with offsets:
  - invalid escape
  - unclosed group/class
  - bad range
  - bad repeat
  - empty repeat target
  - capacity overflow
  - out of memory

## Priority 2: Matching API

- Implement `Regex.matches(text, alloc)` for search semantics and
  `Regex.is_full_match(text, alloc)` for full-string matching.
- Implement `Regex.find(text, alloc)` returning the leftmost match.
- [x] Implement `Regex.find_all(text, alloc)` returning owned match spans.
- [x] Add module-level one-shot helpers for use where the pattern does not need to
  be reused.
- [x] Define zero-length match progress rules so `find_all` cannot loop forever.

## Priority 3: Documentation And Tests

- README should introduce installation, first match, reusable compiled regex,
  errors, and byte-oriented semantics.
- Module docs should explain grammar, matching semantics, allocation ownership,
  and engine guarantees.
- Public items need `///` contracts for ownership, allocation, and failure.
- Tests must cover:
  - literals, dot, classes, ranges, negation
  - grouping, alternation, quantifiers, anchors
  - repeat syntax errors and parse offsets
  - leftmost matching and zero-length progress
  - reusable compiled regexes and one-shot helpers

## Priority 4: Expansion Path

- Captures and named captures.
- Replacement helpers and split helpers.
- Case-insensitive and multi-line flags.
- Unicode-aware character classes as an explicit feature, not a silent default.
- Optional compile-time or build-time pattern checking if Kern grows the right
  hooks.

## Done For First Publishable Version

- `craft fmt --check --verbose --color never` passes.
- `craft test --color never` passes.
- `craft style --verbose --color never` reports no missing public docs for the
  hand-written public API.
- README examples have matching compile-only tests.
- No public duplicate compatibility API is kept for names that were replaced
  before publication.
