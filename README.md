# Starling.RegExp

A pure-managed ECMAScript 2024/2025 regular-expression engine for .NET. Pike-VM
matcher with a Russ Cox style epsilon closure, a Thompson-NFA compiler, and a
spec-faithful parser that enforces the §22.2.1.1 early errors.

Originally extracted from the [Starling browser engine](https://github.com/) — used in
production to back the JS `RegExp` object.

## Why not `System.Text.RegularExpressions`?

`Regex` is a great engine, but it follows .NET regex syntax, not the JavaScript
syntax that the ECMAScript spec defines. The two diverge on real, observable
things: named-capture syntax, lookbehind quantifiability, the `u` and `v`
flags, `\p{...}` property escapes, the Annex B legacy rules, `\k<name>`
behaviour, and the early-error grammar.

`Starling.RegExp` is built to match the spec, with the early errors enforced at
parse time so a JS host can surface them as `SyntaxError`.

## Install

```
dotnet add package Starling.RegExp
```

Targets `net10.0`. Pure managed, no native dependencies.

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

- All eight ES2024 flags: `g i m s u v y d`
- Named captures, including the ES2025 same-name-across-alternatives rule
- Backreferences (numeric and `\k<name>`), with forward references
- Lookahead and lookbehind, positive and negative
- `\p{...}` and `\P{...}` for the supported property whitelist
  (Letter, Number, Decimal_Number, White_Space, Punctuation, Symbol,
  Uppercase_Letter, Lowercase_Letter)
- Annex B legacy semantics in non-Unicode mode
- §22.2.1.1 early errors thrown at compile as `RegexSyntaxException`

Matching is linear-time on the common subset. Patterns that use backreferences
or lookaround fall back to a recursive matcher.

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

TBD.
