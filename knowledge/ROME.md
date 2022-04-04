Issue: https://github.com/rome/tools/issues/2275

### Mob Session 1 Learnings:
1. How to use the Rome playground to see the AST that was being constructed, given syntax.
2. Given syntax input, how to use .format to tell `rome_formatter` to navigate ("visit") a node, and that there is essentially a 1:1 mapping between a file (`impl ToFormatElement for`) per Abstract Syntax Tree Node for the given syntax input
3. Where to add test files,  and how to generate snapshot tests. ( https://github.com/RustPhilly/tools/blob/main/crates/rome_formatter/docs/write_tests.md )

### Mob Session 2 Learnings:
1. Understanding that our goal, when adding logic to `rome_formatter`, is to create an Intermediate representation of FormatElements, where `FormatElement` are just a handful of enums. That this (Formatter IR) is an additional abstraction on top of the AST/CST that the parser generates. (also shown on the Rome playground).
2. How to build and open docs for rome_formatter's formatter.
3. That the formatter helper methods are just helpers that construct pieces of this Formatter IR, ( https://github.com/RustPhilly/tools/blob/main/crates/rome_formatter/docs/implement_the_formatter.md )

![alt text](https://github.com/RustPhilly/tools/blob/feature/format-jsx-element/crates/rome_formatter/docs/high-level-rome-formatter-overview.jpeg)

4. Formatter helper methods may construct pieces of this IR that we don't want/need - for example, `formatter.format_list` injecting `LineMode::Empty` or `LineMode::Hard`, leaving us w/ questions like:

Outstanding Questions:
1. When do we want to use an existing `formatter` helper?
2. When do we want to leave `formatter` untouched and keep our logic within a specific Node type's `ToFormatElement` ?
3. When do we want to add a `formatter` helper?
4. When do we want to modify an existing `formatter` helper implementation?

In deriving answers for these questions, we've further uncovered that there are three distinct areas of the codebase that matter while contributing to the rome formatter:

1. `formatter` (`Formatter` instance) - tends to be language agnostic (it should not know anything about the AST) and it's the one in charge of manipulating tokens and trivias. It's like a CRUD for our internal IR.
2. `utils` - function groups common patterns around a specific language (as for now, only JS and its super languages). These `utils` usually work directly on the AST too.
3. `format_element` - functions that you need to create the `rome_formatter` IR. Our IR is essentially an `enum`, called `FormatElement`. But we found that using utility functions are way better because they better help to shape and create the IR. For example, it's better to use a function called `soft_line_break_or_space` instead of directly using its implementation. 

This means, when we aren't using or modifying formatter helpers (`formatter` methods), we need to determine whether we should be using or modifying `utils` or writing custom logic in `ToFormatElement`.

# Rome formatter FAQ

When should I

### use an existing `formatter` method?
- Try to find existing helpers but it's OK to use the low-level helpers like `join_elements` / `concat_elements` where necessary.
- Be sure to test your formatting with comments because that's what many of those helpers handle for you.

### leave a `formatter` method untouched and keep our logic within a specific Node type's `ToFormatElement`?
- Anything that is node specific should remain in the node's `ToFormatElement` implementation.
- Theoretically you always use the `formatter`, even when you indirectly call `token.format_or_empty(formatter)`. Usually you directly use the `formatter` when you don't have to format recurring patterns such as separated lists, delimited blocks, etc.;

### add a new `formatter` method
- When it's a frequent pattern used by a variety of nodes. If it only applies to a specific group of nodes, prefer `utils` (for example, formatting expressions with left and right hand sides)

### modify an exsting `formatter` method
- when there's a bug or a new requirement in an existing pattern that pops up
