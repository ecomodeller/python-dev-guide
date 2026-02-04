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
   [Numpydoc](https://github.com/numpy/numpy/blob/master/doc/HOWTO_DOCUMENT.rst.txt) format.
1. Use concise and descriptive names. Avoid abbreviations for anything that isn't an acronym.

## House rules

1. Python 3.11+ only.
1. Use [`pathlib`](https://docs.python.org/library/pathlib.html) for handling file paths - never split by literal slashes.
1. Separate I/O from logic: split tasks into (1) input, (2) processing (pure functions), and (3) output.
1. Prefer f-strings for string formatting. Avoid the old `%` syntax or `+str(something)+`.

## Environment and dependency management

Use [uv](https://docs.astral.sh/uv/) for environment and dependency management.

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

1. Use `pyproject.toml` for all project metadata and configuration
1. Automate the release

# Compatibility

When designing your package, assume as little as possible about the system that is used to run it. You _must not_ assume a certain folder structure. You _may_ assume a certain target OS, if keeping your code multi-platform compliant would impose an unreasonable amount of extra maintenance.

If your code does not run on other systems, it is worthless.

# Command Line Interfaces (CLI)

A good CLI is crucial to the usability and thus success of your package, but also one of the hardest things to get right. There is no silver bullet, but these general guidelines may help:

1. Use a well-established library to handle the CLI for you. For Python, we prefer [Typer](https://typer.tiangolo.com/) or [click](https://click.palletsprojects.com/).
1. Follow GNU conventions:

   1. Arguments specified with a single `-` may only be followed by a single character; otherwise, use `--`.

      - Good: `-i` and `--input`
      - Bad: `-in` or `--i`

   1. Use hyphens to split words in your arguments:
      - Good: `--input-folder`
      - Bad: `--inputfolder` or `--input_folder`

1. Make proper use of flags and multi-argument commands. `--debug` is better than `--debug=True`. `--input-files a.txt b.txt c.txt` is better than `--input-files a.txt,b.txt,c.txt`. If the former is not possible, use `-i a.txt -i b.txt -i c.txt`.
1. Write expressive help messages. Include default values for arguments.

Also, think whether you need a CLI at all. Is your package just a Python wrapper around a command line tool? You may be better off calling the tool directly and using Python for pre-processing.

# Testing

1. If you contribute code to a repository with CI, you _must_ write at least one test for your code.
1. If the repository measures code coverage, your addition must not decrease coverage. If code is _inherently untestable_ for technical reasons, use `# pragma: no cover` to exclude it from coverage reports.
