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
use regex;
use sys.mem.page;

enum AppError {
    Regex: regex.Error,
}

fn app(gpa: &mut Allocator) void!AppError {
    let re = "vk[A-Z][A-Za-z0-9_]*".compile_regex(gpa)
        .map_err([](err: regex.Error) AppError { return .{ Regex: err }; })
        .!..&;
    defer re.deinit(gpa);

    let found = re.find("call vkCreateInstance before vkCreateDevice", gpa)
        .map_err([](err: regex.Error) AppError { return .{ Regex: err }; })
        .!;
    _ = found;
    return .{ Ok: {} };
}

fn main() i32 {
    let page = page()..&;
    let gpa = gpa().on(page)..&;
    defer gpa.deinit();

    let .{ Ok: _ } = app(gpa) else return 1;
    return 0;
}
```

Useful entry points:

- `regex.compile(pattern, alloc)` builds an owned reusable `Regex`.
- `pattern.compile_regex(alloc)` is the fluent form for reusable regexes.
- `re.matches(text, alloc)` returns true when the pattern appears in `text`.
- `re.is_full_match(text, alloc)` returns true when the whole text matches.
- `re.find(text, alloc)` returns the leftmost match span.
- `re.find_all(text, alloc)` returns owned match spans.
- `regex.matches(pattern, text, alloc)`, `pattern.matches_regex(text, alloc)`,
  `regex.find(pattern, text, alloc)`, and `pattern.find_regex(text, alloc)` are
  one-shot helpers.

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
