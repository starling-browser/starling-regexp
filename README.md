# Starling.RegExp

A JavaScript-style regular expression engine for .NET. It uses a
[Pike virtual machine][pikevm] to run the match, [Russ Cox's epsilon
closure][cox] to walk the state set, a [Thompson NFA][thompson] compiler to
build the program, and a parser that catches the
[§22.2.1.1 early errors][ee] at compile time.

Pulled out of the Starling browser engine, where it powers the JavaScript
`RegExp` object.

[bcl]: https://learn.microsoft.com/dotnet/api/system.text.regularexpressions.regex
[pikevm]: https://swtch.com/~rsc/regexp/regexp2.html
[cox]: https://swtch.com/~rsc/regexp/
[thompson]: https://dl.acm.org/doi/10.1145/363347.363387
[ee]: https://tc39.es/ecma262/#sec-patterns-static-semantics-early-errors

## When to use this

Reach for `Starling.RegExp` only if one of these fits:

- You are building a JavaScript engine, browser, or sandbox, and need
  `RegExp` semantics that match the ECMAScript spec.
- You are taking a regex pattern that a user wrote as JavaScript (for
  example, from a web form, a config file, or `package.json`), and the
  pattern must keep its JavaScript meaning.
- You need the §22.2.1.1 early errors raised at parse time, so you can
  surface them as a `SyntaxError` to a JavaScript host.
- You are testing or teaching ECMAScript regex semantics and want a
  spec-faithful reference.

## When not to use this

Stick with [`System.Text.RegularExpressions`][bcl] if:

- You are doing general pattern matching in .NET.
- Your patterns were written for .NET regex syntax.
- You want the fastest matcher, source generators, or the rich
  `Regex` / `Match` / `Group` API surface.
- You do not care which spec your regex follows.

The built-in class is the right default. This library is a niche tool.

## Why the two are not the same

.NET regex and JavaScript regex look alike but are not. They differ in
ways you can see at runtime:

- how you write a named capture
- whether you can repeat a lookbehind
- the `u` and `v` flags
- `\p{...}` property escapes
- the Annex B legacy rules
- how `\k<name>` works
- which patterns are early errors

If you feed a JavaScript regex pattern to .NET's `Regex` class, some
patterns will throw, some will parse but match different text, and some
will match the same text but capture different groups. `Starling.RegExp`
exists to make those patterns mean what they would mean in a browser.

## Install

```
dotnet add package Starling.RegExp
```

Targets `net10.0`.

## Use

```csharp
using Starling.RegExp;

RegexFlagParser.TryParse("gi", out var flags, out _);
var re = CompiledRegex.Compile(@"(?<year>\d{4})-(?<month>\d{2})", flags);

var m = re.Exec("2026-05-26", 0);
if (m is not null)
{
    m.Group(0);                            // "2026-05"
    m.Group(re.NamedCaptures["year"]);     // "2026"
    m.Group(re.NamedCaptures["month"]);    // "05"
}
```

## What it supports

- All eight ECMAScript 2024 flags: `g i m s u v y d`
- Named captures, including the ECMAScript 2025 same-name-across-alternatives
  rule
- Backreferences (numeric and `\k<name>`), with forward references
- Lookahead and lookbehind, positive and negative
- `\p{...}` and `\P{...}` for the supported property whitelist
  (Letter, Number, Decimal_Number, White_Space, Punctuation, Symbol,
  Uppercase_Letter, Lowercase_Letter)
- Annex B legacy semantics in non-Unicode mode
- §22.2.1.1 early errors thrown at compile as `RegexSyntaxException`

Matching takes linear time on the common subset. If a pattern uses
backreferences or lookaround, it falls back to a recursive matcher.

## Layout

```
src/Starling.RegExp/
  RegexParser.cs       source → AST + early errors
  RegexAst.cs          AST node records
  RegexCharClass.cs    character classes + Unicode property tables
  RegexFlags.cs        flag enum + parser
  RegexCompiler.cs     AST → Pike VM bytecode (Thompson NFA construction)
  RegexInstruction.cs  opcode enum + instruction record
  RegexProgram.cs      compiled bytecode + side tables
  RegexPikeVm.cs       linear-time matcher + recursive fallback
  CompiledRegex.cs     public facade
  MatchResult.cs       match result with capture spans

tests/Starling.RegExp.Tests/
  RegexEarlyErrorTests.cs   §22.2.1.1 grammar coverage
  RegexPikeVmTests.cs       end-to-end matcher coverage
```

## Build

```
dotnet build
dotnet test
```

## License

BSD 2-Clause. See [`LICENSE`](LICENSE).
