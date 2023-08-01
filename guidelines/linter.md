# Linters and Code Formatters

- [flake8](#flake)
- [isort](#isort)
- [mypy](#mype)
- [pre-commit](#pre-commit)

## <a name="flake">flake8</a>

Flake8 is a Python linter that checks for PEP8 style compliance, as well as
other style and error checks.

Generally, you will want to use the following configuration:

```text
[flake8]
ignore = E203, E266, E501, W503
max-line-length = 88
max-complexity = 18
select = B,C,E,F,W,T4
```

- `ignore = E203, E266, E501, W503`

    - The ignore option tells Flake8 to ignore specific types of errors or warnings during linting.
    - The codes following `ignore` are specific error codes that you want to exclude from the linting process.
    - In your case, you are ignoring the following error codes:
        - E203: Whitespace before ':'
        - E266: Too many leading '#' for block comment
        - E501: Line too long (maximum allowed line length is set separately)
        - W503: Line break occurred before a binary operator (usually related to the and and or operators)

- `max-line-length = 88`

    - The max-line-length option sets the maximum allowed line length for your code.
    - In this case, the maximum line length is set to 88 characters. This means that Flake8 will raise an error if any line in your code exceeds this length.

- `max-complexity = 18`

    - The `max-complexity` option sets the maximum allowed McCabe complexity for your functions.
    - McCabe complexity is a metric that measures the complexity of a function based on the number of decision points (e.g., if statements, loops) it contains.
    - In your setup, the maximum complexity is set to 18. If a function's complexity exceeds this value, Flake8 will raise a warning.

- `select = B,C,E,F,W,T4`

    - The `select` option specifies which categories of errors or warnings you want to enable for linting.
    - The codes following `select` represent specific categories of checks you want to include in the linting process. Each code represents a different category:
        - `B`: Bug - Errors related to possible bugs in your code.
        - `C`: Complexity - Issues related to code complexity.
        - `E`: Error - General syntax and semantic errors.
        - `F`: Formatting - Issues related to code formatting and style.
        - `W`: Warning - Non-critical issues or potential problems in your code.
        - `T4`: Type hints related checks - Checks for type annotations and hints.

## <a name="isort">isort</a>

`isort` is a Python utility that helps you organize and sort import statements in your Python code automatically. It ensures that import statements are properly organized and formatted according to specific rules, making your code more readable and maintainable.

Generally, you will want to use the following configuration:

```text
[isort]
multi_line_output=3
include_trailing_comma=True
force_grid_wrap=0
use_parentheses=True
line_length=88
```

- `multi_line_output=3`
    - The `multi_line_output` option specifies how to format imports that span multiple lines.
    - In this case, `multi_line_output=3` means that isort will use the "Vertical Hanging Indent" style for multiline imports.
    - The Vertical Hanging Indent style aligns the imports vertically and indents them with respect to the first import statement.

- `include_trailing_comma=True`
    - The `include_trailing_comma` option determines whether a trailing comma is added after the last import in a multiline import block.
    - When set to True, isort will add a trailing comma after the last import in multiline import blocks.

- `force_grid_wrap=0`
    - The `force_grid_wrap` option controls the behavior of grid wrapping for multiline imports.
    - When set to `0`, isort will try to fit as many imports as possible on a single line before wrapping to the next line, based on the `line_length` option.

- `use_parentheses=True`
    - The `use_parentheses` option specifies whether imports should be grouped within parentheses.
    - When set to `True`, isort will wrap imports within parentheses if they span multiple lines or if they include trailing commas.

- line_length=88
    - The `line_length` option sets the maximum line length for the isort output.
    - In this case, the line length is set to 88 characters. If an import statement exceeds this length, isort will wrap it to the next line.

## <a name="mypy">mypy</a>

`mypy` is a popular static type checker for Python that analyzes your code and checks for type-related errors and inconsistencies. It helps catch potential bugs and improve code quality by enforcing strict typing.

Generally, you will want to use the following configuration:

```text
[mypy]
files=main
ignore_missing_imports=True

[mypy-main.app_benchmarks.migrations.*]
ignore_errors=True

[mypy-main.app_factsheets.migrations.*]
ignore_errors=True

[mypy-main.app_returndata.migrations.*]
ignore_errors=True
```

- `files=main`
    - The `files` option specifies which files should be checked by `mypy`.
    - In your case, `main` is the name of the Python package (or directory) containing your code. `mypy` will only analyze the files within this package.

- `ignore_missing_imports=True`
    - The `ignore_missing_imports` option determines whether `mypy` should report an error when it encounters an import statement for which it cannot find a type hint.
    - When set to `True`, `mypy` will not raise errors for missing type hints in import statements. This is useful when dealing with third-party libraries or external code that might not have type annotations.

- `[mypy-main.app_benchmarks.migrations.*], [mypy-main.app_factsheets.migrations.*], [mypy-main.app_returndata.migrations.*]`
    - These sections are used to configure specific settings for the indicated packages that contain migrations within your `main` package.
    - `migrations` typically refer to database migration scripts, and since these scripts might not always follow strict typing rules, you can use the `ignore_errors=True` option to suppress any type errors specifically within these migration scripts.

## <a name="pre-commit">pre-commit</a>

`pre-commit` is a Python utility that allows you to run various checks and tests on your code before committing it to your repository. It can be used to run linters, code formatters, and other tools that help improve code quality and maintainability.

Install `pre-commit` using `pip`:

```bash
pip install pre-commit
```

Generally, you will want to use the following configuration in the `.pre-commit-config.yaml` file:

```text
repos:
- repo: local
  hooks:
  - id: isort
    name: isort
    stages: [commit]
    language: system
    entry: isort
    types: [python]

  - id: black
    name: black
    stages: [commit]
    language: system
    entry: black
    types: [python]

  - id: flake8
    name: flake8
    stages: [commit]
    language: system
    entry: flake8
    types: [python]
    exclude: setup.py

  - id: mypy
    name: mypy
    stages: [commit]
    language: system
    entry: mypy
    types: [python]
    pass_filenames: false
```

Install hooks:

```bash
pre-commit install
```