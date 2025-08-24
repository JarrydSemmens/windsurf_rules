# C# Coding Rules
- We use Microsoft's C# coding conventions with some modifications.

## Scope
- Applies to all C# in this workspace.
- Intent: readability, consistency, modern C#.

## Language
- Prefer modern features. Avoid obsolete constructs. :contentReference[oaicite:1]{index=1}
- Catch only exceptions you can handle. Don’t catch `System.Exception` without a filter. :contentReference[oaicite:2]{index=2}
- Use async/await for I/O-bound work. :contentReference[oaicite:3]{index=3}
- Use C# keywords for built-in types: `string`, `int`, etc. (incl. `nint`, `nuint`). :contentReference[oaicite:4]{index=4}
- Use `int` unless unsigned semantics are required. :contentReference[oaicite:5]{index=5}
- Use `var` only when the RHS makes the type obvious; otherwise spell the type. :contentReference[oaicite:6]{index=6}
- Prefer string interpolation; use `StringBuilder` for large/loop appends. Prefer raw string literals when helpful. :contentReference[oaicite:7]{index=7}
- Use `using` statements (including declaration form) instead of `try/finally` for `Dispose`. :contentReference[oaicite:8]{index=8}
- Use `&&` and `||` for comparisons (short-circuit). :contentReference[oaicite:9]{index=9}
- Use concise `new()` forms when the variable type matches. Use object/collection initializers. :contentReference[oaicite:10]{index=10}
- Use lambdas for event handlers you won’t remove. :contentReference[oaicite:11]{index=11}
- Call static members via the type name. :contentReference[oaicite:12]{index=12}
- LINQ: meaningful range/result names, alias anonymous properties, rename ambiguous properties; use implicit typing in queries. :contentReference[oaicite:13]{index=13}

## Using directives
- Place `using` directives **outside** namespaces. Use `global::` if needed. :contentReference[oaicite:14]{index=14}

## Style and Layout
- Indent with 4 spaces. No tabs. :contentReference[oaicite:15]{index=15}
- Allman braces: `{` and `}` on their own lines. :contentReference[oaicite:16]{index=16}
- One statement per line. One declaration per line. :contentReference[oaicite:17]{index=17}
- Add a blank line between members. Indent continuation lines by one level. :contentReference[oaicite:18]{index=18}
- Use parentheses to make precedence clear when needed. :contentReference[oaicite:19]{index=19}

## Comments
- Use `//` for short explanations. Avoid block comments for long prose; move prose to docs. :contentReference[oaicite:20]{index=20}
- XML doc comments for public APIs. Start with uppercase, end with a period; one space after `//`. :contentReference[oaicite:21]{index=21}

## Naming
- PascalCase: namespaces, types, all public/protected members, and local functions. :contentReference[oaicite:22]{index=22}
- Interfaces start with `I`; attribute types end with `Attribute`. :contentReference[oaicite:23]{index=23}
- Enums: singular for non-flags; plural for flags. :contentReference[oaicite:24]{index=24}
- Avoid `__` (double underscore); reserved for compiler. Prefer clear, descriptive names. Avoid unclear acronyms. :contentReference[oaicite:25]{index=25}
- Locals: camelCase. :contentReference[oaicite:26]{index=26}
- Constants: **PascalCase** (fields and local const). :contentReference[oaicite:27]{index=27}
- Private instance fields: `_camelCase`. Private/internal static fields: `s_`. Thread-static: `t_`. :contentReference[oaicite:28]{index=28}
- Primary constructors: classes/structs use camelCase parameters; records use PascalCase (they become public props). :contentReference[oaicite:29]{index=29}
- Avoid single-letter names except simple loop counters. :contentReference[oaicite:30]{index=30}
- Parameters: camelCase with the prefix `in_`. :contentReference[oaicite:31]{index=31}

## Do Not

- Don’t place using inside namespaces. :contentReference[oaicite:32]{index=32}
- Don’t overuse var when the type is unclear. :contentReference[oaicite:33]{index=33}
- Don’t catch broad exceptions without handling. :contentReference[oaicite:34]{index=35}

## Examples (concise)
```csharp
// Using directives
using System.Text;

// Naming + fields
public class DataService(IWorkerQueue in_workerQueue, ILogger in_logger)
{
    private static IWorkerQueue s_fallbackQueue;
    private IWorkerQueue _workerQueue = in_workerQueue;

    public ILogger Logger { get; } = in_logger;

    public async Task<string> LoadAsync(string in_id)
    {
        // var only when obvious
        var sb = new StringBuilder();
        sb.Append($"{Logger?.GetType().Name}: {in_id}");
        using var stream = await FetchAsync(in_id).ConfigureAwait(false);
        return sb.ToString();
    }
}
