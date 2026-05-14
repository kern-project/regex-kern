# regex-kern

`regex-kern` provides byte-oriented regular expressions for Kern. The Craft
package name is `regex`, so applications import it with:

```kern
use regex;
```

The engine compiles patterns into a Thompson-style NFA and avoids recursive
backtracking. Matching is byte-based by design: UTF-8 text is accepted as bytes,
but Unicode character classes and grapheme semantics are not guessed silently.

## Quick Start

```toml
[dependencies]
regex = { git = "https://github.com/softfault/regex-kern.git" }
```

Local development can use a path dependency:

```toml
[dependencies]
regex = { path = "../regex-kern" }
```

## Usage

```kern
use base.mem.alloc.{Allocator, gpa};
use std.mem.Page;
use regex;

enum AppError {
    Regex: regex.Error,
}

fn app(gpa: &mut Allocator) void!AppError {
    let re = "vk[A-Z][A-Za-z0-9_]*".regex().compile(gpa)
        .map_err([](err: regex.Error) AppError { return .{ Regex: err }; })
        .?..&;
    defer re.deinit(gpa);

    let found = re.find("call vkCreateInstance before vkCreateDevice", gpa)
        .map_err([](err: regex.Error) AppError { return .{ Regex: err }; })
        .?;
    _ = found;
    return .{ Ok: {} };
}

fn main() i32 {
    let page = Page.{}..&;
    let gpa = gpa().on(page)..&;
    defer gpa.deinit();

    let .{ Ok: _ } = app(gpa) else return 1;
    return 0;
}
```

Useful entry points:

- `pattern.regex()` treats a byte slice as a regex pattern handle.
- `pattern.regex().compile(alloc)` builds an owned reusable `Regex`.
- `pattern.regex().matches(text, alloc)` compiles once and returns true when
  the pattern appears in `text`.
- `pattern.regex().find(text, alloc)` compiles once and returns the leftmost
  match span.
- `pattern.regex().find_all(text, alloc)` compiles once and returns owned match
  spans.
- `re.matches(text, alloc)` returns true when the pattern appears in `text`.
- `re.is_full_match(text, alloc)` returns true when the whole text matches.
- `re.find(text, alloc)` returns the leftmost match span.
- `re.find_all(text, alloc)` returns owned match spans.

## Errors And Ownership

`compile`, `find`, `matches`, `is_full_match`, and `find_all` can return
`regex.Error`. The error carries an `ErrorKind` and a byte `offset` into the
pattern. Matching can fail only because it needs temporary allocation for NFA
state lists.

`Regex` owns compiled instruction storage and must be released with
`re.deinit(alloc)`. `MatchList` owns collected spans and must be released with
`matches.deinit(alloc)`. A `Match` itself is just byte offsets into caller-owned
text.

## Pattern Syntax

The first engine supports byte regex syntax for literals, escaped
metacharacters, `.`, character classes, ranges, negated classes, grouping,
alternation, `*`, `+`, `?`, `{m}`, `{m,n}`, `{m,}`, `^`, and `$`.

## License

`regex-kern` is distributed under the MIT License. See `LICENSE`.
