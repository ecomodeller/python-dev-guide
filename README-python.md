# Python

## General coding style

1. Lines may be up to 100 columns wide.
1. Use [ruff](https://docs.astral.sh/ruff/) for linting and formatting. Configure in `pyproject.toml`:
   ```toml
   [tool.ruff]
   line-length = 100

   [tool.ruff.lint]
   select = ["E", "F", "I", "UP", "B", "SIM"]
   ```
1. Many reasonable suggestions are found in [Google's Python Style Guide](https://google.github.io/styleguide/pyguide.html). Read it carefully, and violate it only in exceptional circumstances.
1. Write [docstrings](https://sphinxcontrib-napoleon.readthedocs.io) for all public functions, in
   [Numpydoc](https://numpydoc.readthedocs.io/en/latest/format.html) format.
1. Use concise and descriptive names. Avoid abbreviations for anything that isn't an acronym.

1. Target Python 3.11+ (prefer latest stable).
1. Use [`pathlib`](https://docs.python.org/library/pathlib.html) for handling file paths - never split by literal slashes.
1. Separate I/O from logic: split tasks into (1) input, (2) processing (pure functions), and (3) output.
1. Prefer f-strings for string formatting. Avoid the old `%` syntax or `+str(something)+`.

### Type hints

1. Use type annotations for all function signatures (parameters and return types).
   ```python
   def read_records(path: Path, *, skip_header: bool = True) -> list[dict[str, str]]:
   ```
1. Use modern built-in generics (`list[int]`, `dict[str, Any]`, `tuple[int, ...]`, `str | None`) instead of importing from `typing`. These have been supported since Python 3.10+.
1. Use [`mypy`](https://mypy.readthedocs.io/) for static type checking. Add it to CI alongside ruff. A reasonable mypy configuration in `pyproject.toml`:
   ```toml
   [tool.mypy]
   strict = true
   ```
1. Avoid `Any` unless interfacing with untyped libraries. If you reach for `Any`, consider whether a `Protocol`, `TypeVar`, or more specific type would work.
1. For structured data (config files, model parameters), prefer [`dataclasses`](https://docs.python.org/3/library/dataclasses.html) over plain dicts. For data from external sources that needs runtime validation (APIs, user input), consider [Pydantic](https://docs.pydantic.dev/) models.

## Scientific Python

1. Prefer [NumPy](https://numpy.org/) for numerical arrays and [pandas](https://pandas.pydata.org/) for tabular/time series data. Use [xarray](https://xarray.dev/) when working with labelled multi-dimensional data (e.g. results from mikeio).
1. Use [matplotlib](https://matplotlib.org/) for plotting. Keep plot-generation code separate from data processing so results can be reproduced without re-running expensive computations.
1. Avoid reinventing numerical routines — check NumPy, SciPy, and pandas before writing your own.
1. Use `logging` instead of `print()` for status messages in long-running processing scripts. It allows controlling verbosity without changing code.

## Data files

1. Avoid loading entire files into memory when only a subset is needed. Libraries like mikeio support reading specific time steps or items.
1. Use context managers (`with` statements) when working with file handles.
1. Keep large binary files (dfsu, dfs0, dfs2, mesh) out of git. Use [Git LFS](https://git-lfs.com/) if they must be versioned, or store them in a shared location and document the path.
1. For test data, keep sample files as small as possible — a few time steps and a coarse mesh are enough to verify correctness.

## Environment and dependency management

Use [uv](https://docs.astral.sh/uv/) for environment and dependency management.

**Installing Python:**
```bash
uv python install 3.13       # install a specific version
uv python list                # list available and installed versions
```

**Creating a virtual environment:**
```bash
uv venv
source .venv/bin/activate  # or .venv\Scripts\activate on Windows
```

**Installing dependencies:**
```bash
uv pip install -r requirements.txt
uv pip install -e ".[dev]"  # editable install with dev extras
```

**Managing a project with uv:**
```bash
uv init                    # create a new project
uv add requests            # add a dependency
uv add --dev pytest ruff   # add dev dependencies
uv sync                    # install dependencies from uv.lock
uv run pytest              # run commands in the project environment
```

## Dependencies

1. Think carefully before adding a dependency. Every dependency adds installation time, maintenance burden, and potential security surface. If you only need a small part of a big library, think triple carefully.
1. Rule of three: Refrain from moving code to a separate package until you want to use it from at least [three](<https://en.wikipedia.org/wiki/Rule_of_three_(computer_programming)>) different projects.

## Packaging and releases

1. Use `pyproject.toml` for all project metadata and configuration.
1. Automate releases with CI (e.g. GitHub Actions triggered by a version tag). Use `uv build` to produce wheels and sdists.

# Compatibility

When designing your package, assume as little as possible about the system that is used to run it. You _must not_ assume a certain folder structure. You _may_ assume a certain target OS, if keeping your code multi-platform compliant would impose an unreasonable amount of extra maintenance.

Code that only runs on your machine provides limited value to the team.

# Command Line Interfaces (CLI)

1. Consider whether you need a CLI at all. If your package just wraps a command line tool, call it directly. Many processing scripts are better off as importable functions called from a short `__main__.py`.
1. When you do need a CLI, the stdlib [`argparse`](https://docs.python.org/3/library/argparse.html) is often sufficient. For more complex CLIs, use [Typer](https://typer.tiangolo.com/) or [click](https://click.palletsprojects.com/).
1. Follow GNU conventions: single-char flags with `-` (`-i`), words with `--` (`--input-folder`, not `--input_folder`).
1. Write expressive `--help` messages. Include default values for arguments.

# Notebooks

1. Keep notebooks for exploration, visualization, and communication — not for production logic. Extract reusable code into proper modules and import it.
1. Consider [marimo](https://marimo.io/) as an alternative to Jupyter. Marimo notebooks are stored as plain `.py` files (diffable, lintable) and enforce a DAG execution order with reactive re-execution — no hidden state.
1. Keep cells short and focused. Each cell should do one thing.
1. Never commit notebooks with large outputs (plots, dataframes). Clear outputs before committing, or use a tool like [nbstripout](https://github.com/kynan/nbstripout). This does not apply to marimo notebooks.

# Testing

1. Use [pytest](https://docs.pytest.org/). No reason to use `unittest` in new code.
1. If you contribute code to a repository with CI, you _must_ write at least one test for your code.
1. If the repository measures code coverage, your addition must not decrease coverage. If code is _inherently untestable_ for technical reasons, use `# pragma: no cover` to exclude it from coverage reports.
1. Structure tests to mirror your source tree (e.g. `src/mypackage/io.py` → `tests/test_io.py`).
1. Test behavior, not implementation. Tests should not break when you refactor internals.
1. Use fixtures for shared setup. Prefer `tmp_path` over manually creating temp directories.
1. Keep tests fast. Mock or stub slow externals (network, databases, filesystems). Use [`pytest-mock`](https://github.com/pytest-dev/pytest-mock) for cleaner patching.
1. Use `pytest.raises` for expected exceptions and `pytest.approx` for scalar floating-point comparisons. For array comparisons, use `np.testing.assert_allclose`.
1. For data-heavy functions, use [`pytest.mark.parametrize`](https://docs.pytest.org/en/stable/how-to/parametrize.html) to cover multiple inputs without duplicating test code.
1. For numerical code, consider reference-data tests: compare output against a known-good result file. Store reference files in `tests/testdata/` and keep them small.
