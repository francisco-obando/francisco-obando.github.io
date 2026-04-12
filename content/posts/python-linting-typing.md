---
title: "Scaling Python Quality: Linting and Type Checking with Ruff and Pyrefly"
date: 2026-04-11T10:00:00Z
tags: ["Python", "CI/CD", "Linting", "Type Checking", "Ruff", "Pre-commit"]
categories: ["Engineering"]
summary: "Implementing a tiered linting and typing strategy using Ruff, Pyrefly, and pre-commit to balance strict standards with developer flexibility."
---

## The Challenge of Python at Scale

As Python codebases grow, maintaining a consistent style and catching "silent" type errors becomes a significant overhead. In a multi-project environment—ranging from data pipelines to web services—a one-size-fits-all linting policy often leads to developer friction. 

To solve this, we implemented a tiered approach to static analysis that balances absolute standards with domain-specific flexibility.

---

## The Tooling Stack

We standardized our CI/CD pipelines around three core tools:

1.  **[Ruff](https://astral.sh/ruff)**: An extremely fast Python linter and code formatter written in Rust. It replaced multiple tools (Flake8, Isort, Black) with a single configuration.
2.  **[Pyrefly](https://github.com/skylight-ai/pyrefly)**: Used to enhance our code quality checks, particularly for identifying complex patterns and maintaining high-performance Python standards.
3.  **[pre-commit](https://pre-commit.com/)**: The framework that ties it all together, ensuring these checks run locally before every commit and as a mandatory gate in our CI.

---

## A Tiered Policy for Standards

Instead of a single "strict" mode, we categorized our checks into three levels:

### 1. Mandatory Global Checks
Every project, regardless of its purpose, must pass basic syntax validation, security linting (Bandit), and core stylistic rules.

### 2. Field-Specific Mandatory Checks
We recognized that a **Pandas**-heavy data science project has different needs than a **PostgreSQL**-driven API. We implemented specific rulesets for:
*   **Data Pipelines**: Focused on resource management and type safety.
*   **Database Services**: Strict checks on query construction and ORM usage.
*   **Pandas/NumPy**: Rules to prevent common vectorized performance anti-patterns.

### 3. Recommended Optional
To encourage exploration without blocking releases, we provide a "recommended" tier of checks that developers can opt into for better code health but aren't yet mandatory for the build to pass.

---

## Pragmatic Type Checking

Our philosophy on static typing is "strict where it counts, lax where it helps."

*   **Production Code**: Strict type checking is enforced to prevent runtime errors.
*   **Test Fixtures**: We require full type coverage for fixtures. Since fixtures are the foundation of our test suite, ensuring they are correctly typed prevents "false pass" scenarios where tests pass because the mock data was incorrectly structured.
*   **Test Cases**: We allow for more lax typing within individual test functions to keep test writing fast and expressive.

---

## Minimal Pre-commit Configuration

Here is a minimal `.pre-commit-config.yaml` that showcases this tiered implementation:

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.3.0
    hooks:
      - id: ruff
        args: [ --fix, --exit-non-zero-on-fix ]
      - id: ruff-format

  - repo: https://github.com/skylight-ai/pyrefly
    rev: v0.1.0
    hooks:
      - id: pyrefly
        # Field-specific checks can be enabled via args
        args: ["--domain", "data-pipelines"]

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.9.0
    hooks:
      - id: mypy
        additional_dependencies: [types-requests, types-setuptools]
        # Ensuring fixtures are checked even if tests are lax
        args: [--check-untyped-defs, --ignore-missing-imports]
```

By implementing this framework, we reduced our "time-to-review" as the machine handles the syntax and typing debates, leaving the humans to focus on the architectural logic.
