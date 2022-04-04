Issue: https://github.com/rome/tools/issues/2275

### Mob Session 1 Learnings:
1. How to use the Rome playground to see the AST that was being constructed, given syntax.
2. Given syntax input, how to use .format to tell `rome_formatter` to navigate ("visit") a node, and that there is essentially a 1:1 mapping between a file (`impl ToFormatElement for`) per Abstract Syntax Tree Node for the given syntax input
3. Where to add test files,  and how to generate snapshot tests. ( https://github.com/RustPhilly/tools/blob/main/crates/rome_formatter/docs/write_tests.md )

### Mob Session 2 Learnings:
1. Understanding that our goal, when adding logic to `rome_formatter`, is to create an Intermediate representation of FormatElements, where `FormatElement` are just a handful of enums. That this (Formatter IR) is an additional abstraction on top of the AST/CST that the parser generates. (also shown on the Rome playground).
2. How to build and open docs for rome_formatter's formatter
3. That the formatter helper methods are just helpers that construct pieces of this Formatter IR, ( https://github.com/RustPhilly/tools/blob/main/crates/rome_formatter/docs/implement_the_formatter.md ) (and may construct pieces of this IR that we don't want/need - for example, `formatter.format_list` injecting `LineMode::Empty` or `LineMode::Hard`, leaving us w/ questions like:

Outstanding Questions:
1. When do we want to use an existing formatter helper?
2. When do we want to leave formatter untouched and keep our logic within a specific Node type's ToFormatElement ?
3. When do we want to add a new helper to formatter?
4. When do we want to modify an existing helper in formatter?
implementation? 

Answers:
