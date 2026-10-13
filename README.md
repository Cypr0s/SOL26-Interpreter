# IPP 2026 — SOL26 Interpreter

Implementation of the **IPP 2026** project at FIT VUT Brno.

The project consists of a **SOL26 interpreter written in Python** and a
**PHP integration testing framework**, together with static analysis, code
quality checks and a containerized development environment.

SOL26 is a small Smalltalk-style object-oriented language. Programs are
parsed into an XML representation, which the interpreter then executes.

## Implementation

### SOL26 Interpreter

The interpreter executes SOL26 programs represented as XML and provides the
runtime environment required by the language.

Implemented functionality includes:

- program execution and runtime context
- scopes and variable handling
- classes, objects and methods
- built-in classes
- access modifiers
- `self` and `super`
- `DoesNotUnderstand` mechanism
- object-oriented interpreter structure

The lexer and parser were provided as part of the assignment; my work focused
on the **execution and runtime environment**.

### Integration Tester

A PHP-based integration testing framework for automated testing of the
interpreter.

Implemented functionality includes:

- recursive test discovery
- test case metadata parsing
- test filtering and exclusion
- parser-only, interpreter-only and combined tests
- parser and interpreter execution
- expected exit-code validation
- output comparison
- test categories and point calculation
- JSON test reports
- logging and configurable verbosity
- dry-run mode

The tester was implemented on top of the project structure and data model
provided by the assignment.

### Development & Code Quality

- MyPy static type checking
- Ruff, PHPStan and PHP_CodeSniffer
- automated integration testing
- dependency management (pip, Composer)
- Docker/Podman containerization with separate development, runtime and
  testing environments

## Containerization

A single multi-stage `Dockerfile` works with both Docker and Podman.
All commands below are run from the repository root.

| Stage | Base | Purpose |
|---|---|---|
| `check` | `python:3.14-slim` | Development environment with code-quality tools for both parts. Entry point is `bash`. |
| `runtime` | `python:3.14-slim` | Minimal image that only runs the interpreter, without dev tools. |
| `test` | `FROM runtime` | Adds PHP 8.5, the SOL-to-XML parser and the tester, and runs the test suite. |

### Running the interpreter

```sh
docker build --target runtime -t sol26 .
docker run --rm -v "$PWD/programs:/data" sol26 -s /data/program.xml -i /data/input.txt
```

- `-s` — program in SOL-XML format (required)
- `-i` — file used as the program's standard input (optional)

**Example:** `tests/spec/spec4.test` computes a factorial of the number read
from standard input. Test files start with a short metadata header, so the
XML program is extracted first:

```sh
mkdir -p programs
tail -n +6 tests/spec/spec4.test > programs/factorial.xml
cp tests/spec/spec4.in programs/input.txt     # contains: 7

docker run --rm -v "$PWD/programs:/data" sol26 -s /data/factorial.xml -i /data/input.txt
# prints: 5040
```

### Running the tests

```sh
docker build --target test -t sol26-test .
docker run --rm -v "$PWD/tests:/tests:ro" sol26-test -r /tests > report.json
```

The tester converts each `.test` file to XML, runs the interpreter and compares
the exit code and output with the expected values. Results are written as JSON.

### Code-quality checks

The sources are bind-mounted into the container:

```sh
docker build --target check -t sol26-check .
docker run --rm -it -v "$PWD/int:/src/int" -v "$PWD/tester:/src/tester" sol26-check

# inside the container
cd /src/int    && ./ruff check src && ./mypy src
cd /src/tester && ./phpstan analyse && ./phpcs
```

## Result

**18.3 / 20 points**

## Technologies

Python · PHP · XML/XSD · Docker · Podman · Composer · MyPy · Ruff · PHPStan