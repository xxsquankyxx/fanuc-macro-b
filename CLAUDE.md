# CLAUDE.md

## Project Overview

Fanuc Macro B is a lexer, parser, and interpreter for Fanuc Macro B NC (Numerical Control) machine code files. It enables offline debugging of CNC macro programs without requiring access to an actual CNC machine. Built with TypeScript using the Chevrotain parser library.

## Repository Structure

```
src/
  __tests__/           # Jest test files (one per feature/function)
  tokens/              # Chevrotain token definitions (lexer)
    allTokens.ts       # Ordered token array (order = precedence)
    categories.ts      # Token category groups
    tokens.ts          # Core tokens (operators, values, addresses)
    boolean.ts         # Boolean comparison operators (EQ, NE, LT, etc.)
    controlFlow.ts     # Control flow keywords (IF, THEN, GOTO, WHILE, DO)
    matchers.ts        # Custom token matchers
  monaco-dev/          # Monaco Editor language support (syntax highlighting, themes)
  testing/             # Custom Jest matchers (toMatchToken)
  MacroLexer.ts        # Lexer instance (wraps Chevrotain Lexer)
  MacroParser.ts       # Parser rules (CST-based, extends CstParser)
  MacroInterpreter.ts  # CST visitor/interpreter (extends BaseCstVisitorWithDefaults)
  MacroVariables.ts    # Macro variable register management (Map<number, number>)
  MacroConstants.ts    # G-code address character constants
  utils.ts             # Pipeline functions: lex, parse, interpret, evaluate, validate
  index.ts             # Public API exports
playground/            # Interactive web playground (React + Monaco Editor)
demo/                  # Demo scripts runnable with ts-node-dev
scripts/               # Dev utility scripts
types/                 # Shared TypeScript type definitions
```

## Architecture

Three-stage pipeline: **Lex** -> **Parse** -> **Interpret**

- **Lexer** (`MacroLexer.ts`): Tokenizes input using Chevrotain's `Lexer` with token definitions from `tokens/`
- **Parser** (`MacroParser.ts`): Produces a CST (Concrete Syntax Tree) using Chevrotain's `CstParser`
- **Interpreter** (`MacroInterpreter.ts`): Visits CST nodes using Chevrotain's `BaseCstVisitorWithDefaults` to evaluate expressions and assign macro variables

Key utility functions in `utils.ts`:
- `lex(text)` - tokenize only
- `parse(text)` - lex + parse
- `interpret(text, rule)` - full pipeline with a specific parser rule
- `evaluate(text)` - shorthand for `interpret(text, "lines")`
- `validate(text)` - shorthand for `interpret(text, "program")`

## Common Commands

| Command | Description |
|---------|-------------|
| `yarn test` | Run all Jest tests |
| `yarn lint` | Run ESLint on `src/` and `demo/` |
| `yarn fix` | Auto-fix ESLint issues |
| `yarn build` | Build the playground with Parcel |
| `yarn play` | Start playground dev server on port 3000 |
| `yarn demo:vars` | Run variable manipulation demo |
| `yarn demo:intr` | Run interpreter demo |

## Testing

- **Framework**: Jest with ts-jest preset, plus jest-extended matchers
- **Location**: `src/__tests__/` — one file per feature or built-in function
- **Test naming**: Files named after the feature they test (e.g., `SIN.test.ts`, `macros.test.ts`, `lexer.test.ts`)
- **Run tests**: `yarn test`
- **Run single test**: `yarn test -- --testPathPattern=<pattern>`
- **Coverage**: `yarn test -- --coverage` (outputs to `coverage/`)
- **Custom matchers**: `toMatchToken()` defined in `src/testing/toMatchToken.ts`

Tests use `evaluate()` or `validate()` from `src/utils.ts` to run code through the full pipeline, then assert on `macros`, `parseErrors`, or `lexResult`.

## Code Style and Formatting

- **Prettier**: 80 char width, double quotes, no trailing commas, 2-space indent, LF line endings, no parens on single-arg arrows
- **ESLint**: `@typescript-eslint`, `simple-import-sort` (sorted imports required), `jest-formatting`
- **Import order**: Enforced by `simple-import-sort` — external deps first, then local imports, sorted alphabetically
- **Pre-commit**: Husky runs `lint-staged` which applies ESLint with `--fix` on staged `.ts` files

## TypeScript Configuration

- **Target**: ES2015, CommonJS modules
- **Strict mode**: Enabled (`strict: true`, `noImplicitReturns`, `noFallthroughCasesInSwitch`)
- **noImplicitAny**: Disabled (set to `false`)
- **Path alias**: `@cnc4me/*` maps to `packages/*`
- **JSX**: Preserved (for playground React components)

## Key Conventions

- **File naming**: PascalCase for classes/modules (`MacroParser.ts`), camelCase for utilities (`utils.ts`)
- **Token definitions**: Use Chevrotain's `createToken()` with `name`, `pattern`, `categories`, and `longer_alt` fields. Token order in `allTokens.ts` determines lexer precedence.
- **Parser rules**: Defined with `this.RULE("name", () => { ... })` using `CONSUME`, `MANY`, `OR`, `OPTION`, `SUBRULE`
- **Interpreter methods**: Named to match parser rules, receive CST context objects, return evaluated values
- **Pattern matching**: Uses `ts-pattern` library's `match()` for clean conditionals
- **Macro variables**: Referenced by register number (e.g., `#1`, `#500`), stored in `MacroVariables` as `Map<number, number>`
- **Bracket expressions**: Fanuc Macro B uses `[` `]` for grouping (not parentheses — parentheses denote comments in G-code)

## CI/CD

GitHub Actions workflow (`.github/workflows/main.yml`):
- Triggers on push to `main` and PRs targeting `main`
- Runs on `ubuntu-latest` with Node 14
- Installs with `yarn install`, runs `yarn test`

## Dependencies

- **chevrotain** (`^10.0.0`): Parser/lexer framework — the core of the project
- **ts-pattern** (`^3.3.5`): Pattern matching for TypeScript
- **@monaco-editor/react** (`^4.3.1`): Monaco Editor bindings (playground only)
- **Parcel** (`^2.2.1`): Bundler for the playground app
