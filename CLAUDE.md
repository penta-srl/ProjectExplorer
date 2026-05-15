# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

## 5. Commit Messages

Non inserire mai la firma `Co-Authored-By: Claude ...` nei messaggi di commit.

---

## 7. Working Directory

**Always edit the real project files — never the worktree.**

The primary working directory is `C:\apps\thor\Thor\Tools\Apps\Project Explorer\`. All edits must be made to files under that path, not to any `.claude\worktrees\...` copy.

If a worktree is not based on the currently active branch, its files will be stale — missing commits that exist only on the working branch. Before editing any file in a worktree, verify with `git log` that it shares the same base as the active branch.

**Correct flow:** use the worktree only for planning/exploration; apply all code changes directly to the real source files.

---

## 8. Project Guidelines

Write code consistent with the existing style of the application.

### VFP9 HELP FILE
In c:\vsrc\doc\external\vfp9_help you'll find the VFP9 language manual in HTML format

---

### VFP9 Code Style — Project Conventions

#### Method header

Every method has a declaration line followed by a header block:

```foxpro
	PROCEDURE methodname		&& Short description of what it does
		*==============================================================================
		* Method:			MethodName
		* Status:			Public
		* Purpose:			Same as the description on the declaration line
		* Author:			Doug Hennig
		* Last Revision:	DD/MM/YYYY
		* Parameters:		tcParam - description
		*					tlParam - description (or "none")
		* Returns:			.T. if ...
		* Environment in:	This.cProperty contains ...
		*					This.oOther contains ...
		* Environment out:	the result description
		*					This.cProperty is set to ...
		*==============================================================================
```

- `PROCEDURE` is uppercase; the method name is lowercase
- Two tabs between the method name and `&&`
- Protected methods: `PROTECTED PROCEDURE methodname`
- `Status` is either `Public` or `Protected`
- If there are no parameters or no side effects: write `none`
- Multi-line entries in `Parameters`, `Environment in`, `Environment out` are aligned by indenting continuation lines with `*\t\t\t\t\t`

#### Parameters and local variables

```foxpro
	lparameters tcFile, ;
		tlAll

	local lcProject, ;
		llReturn, ;
		lnDataSession, ;
		loException as Exception
```

- Use `lparameters` (not `parameters`) for all parameters
- One variable per line with `;` line continuation
- Blank line between `lparameters` and `local`, and after `local`

#### Hungarian notation

| Prefix | Type | Context |
|--------|------|---------|
| `lc` | Character | local variable |
| `ll` | Logical | local variable |
| `ln` | Numeric | local variable |
| `lo` | Object | local variable |
| `tc` | Character | parameter (passed) |
| `tl` | Logical | parameter |
| `tn` | Numeric | parameter |
| `to` | Object | parameter |
| `c` | Character | class property (`This.cProperty`) |
| `l` | Logical | class property |
| `n` | Numeric | class property |
| `o` | Object | class property |
| `a` | Array | class property |

#### Block closing — repeat the opening condition

`if`, `do case`, `for`, `try`, and `with` blocks are closed by repeating the opening condition, truncated with `...`:

```foxpro
if vartype(This.oProject) = 'O' and This.lOpenedProject
    ...
endif vartype(This.oProject) = 'O' ...

for each loProject in _vfp.Projects foxobject
    ...
next loProject

do case
    case ...
    otherwise
        ...
endcase

try
    ...
catch to loException
    ...
endtry
```

#### Comments

```foxpro
* Description of the following logical section.

lcAlias = This.cMetaDataAlias     && inline comment with no trailing period
```

- Section comments: `* Sentence starting with a capital letter and ending with a period.`, preceded and followed by a blank line
- Inline comments: `&& text` on the same line as the code, no trailing period
- TODO comments: `*-- todo: description`

#### Case of commands and functions

Everything **lowercase**: `if`, `endif`, `do case`, `endcase`, `local`, `lparameters`, `return`, `try`, `catch`, `endtry`, `for each`, `next`, `with`, `endwith`, `modify project`, `vartype()`, `file()`, `empty()`, `alltrim()`, etc.

#### Assignment alignment

When several consecutive assignments belong to the same object or logical group, align the `=` signs with spaces:

```foxpro
This.oProject        = _vfp.Projects.Item(lcProject)
This.lProjectVisible = This.oProject.Visible
llReturn             = .T.
```

#### Class structure

```foxpro
DEFINE CLASS classname AS parentclass OF "library.vcx"

    *<DefinedPropArrayMethod>
        *m: methodname    && description
        *p: cproperty     && description
    *</DefinedPropArrayMethod>

    *<PropValue>
        cproperty = defaultvalue
        Name = "classname"
    *</PropValue>

    PROCEDURE methodname		&& description
        ...
    ENDPROC

ENDDEFINE
```

#### Boolean values and string comparison

- Booleans: `.T.` and `.F.` (with dots, uppercase)
- Loose comparison (case-insensitive, trimmed): `=`
- Exact comparison: `==`

#### Error handling

```foxpro
try
    ...
catch to loException
    This.cErrorMessage = loException.Message
endtry
```

Error messages are always stored in `This.cErrorMessage`.
