---
name: use-platemo
description: Run, inspect, debug, and extend the local PlatEMO MATLAB platform in this repository. Use when Codex needs to work with benchmark problems, user-defined problems, algorithms, metrics, GUI behavior, or source-level integration inside `PlatEMO/`, and when answers should be derived from the local PlatEMO docs and code instead of assumptions.
---

# Use PlatEMO

## Overview

Use the local checkout under `PlatEMO/` as the source of truth. Start from the actual MATLAB entrypoint and class hierarchy, then read the target algorithm, problem, or metric source before making claims, because PlatEMO's directory layout, labels, algorithms, and problem sets change across releases.

## Start Here

Work from the repository root that contains `PlatEMO/` and `skills/`.

- PlatEMO docs: `PlatEMO/README.md`, `PlatEMO/Doc/releasenote.md`, `PlatEMO/PlatEMO/manual.pdf`
- Entrypoint: `PlatEMO/PlatEMO/platemo.m`
- Core classes: `PlatEMO/PlatEMO/Algorithms/ALGORITHM.m`, `PlatEMO/PlatEMO/Problems/PROBLEM.m`, `PlatEMO/PlatEMO/Problems/UserProblem.m`, `PlatEMO/PlatEMO/Problems/SOLUTION.m`

Read `references/run-workflow.md` for concrete invocation patterns. Read `references/source-map.md` when you need to trace behavior or find the correct source file quickly.

## Workflow

1. Confirm the task type.
   Benchmark run, user-defined problem, metrics/result collection, GUI usage, source debugging, or extension work.
2. Confirm the environment boundary.
   PlatEMO without GUI requires MATLAB R2018a or newer. GUI requires MATLAB R2020b or newer. The local manual also lists Parallel Computing Toolbox and Statistics and Machine Learning Toolbox as requirements.
3. Read the real entrypoints before acting.
   For execution semantics, read `platemo.m`.
   For algorithm behavior, read `ALGORITHM.m` and the target algorithm file.
   For problem behavior, read `PROBLEM.m`, `UserProblem.m`, and the target problem file.
4. Prefer command-line execution for reproducible work.
   Use `matlab -batch` and `cd` into `PlatEMO/PlatEMO` before calling `platemo(...)`.
5. When anything is unclear, read source rather than guessing.
   The release note shows frequent additions and refactors. Do not assume an algorithm, problem, label, or directory from memory.

## Task Guide

### Run an existing algorithm on an existing problem

Use `platemo('algorithm',..., 'problem',...)`. Keep the first pass minimal, then add `N`, `M`, `D`, `maxFE`, `maxRuntime`, `save`, `run`, or `metName` only if needed.

If you need examples, load `references/run-workflow.md`.

### Solve a user-defined problem without creating a new class

Use `UserProblem` implicitly through `platemo(...)` parameters such as `objFcn`, `conFcn`, `evalFcn`, `initFcn`, `decFcn`, `gradFcn`, `encoding`, `lower`, `upper`, `data`, and `once`.

Prefer `evalFcn` when repair, objective calculation, and constraint calculation are tightly coupled. Use `once`, matrix operations, or parallel logic when the evaluation can process multiple candidates at once.

### Inspect or extend PlatEMO itself

Read the target implementation first.

- New algorithm: subclass `ALGORITHM`, implement `main`, and use `ParameterSet` and `NotTerminated`.
- New problem: subclass `PROBLEM`, implement at least `Setting` and `CalObj`.
- Custom metric flow: inspect `Metrics/*.m` and `PROBLEM.CalMetric`.
- GUI behavior or labels: inspect `GUI/GUI.m`, `GUI/uilist.m`, and `GUI/module_*.m`.

### Trace a bug or unclear behavior

Follow the runtime chain:

`platemo.m` -> problem construction -> algorithm construction -> `ALGORITHM.Solve` -> algorithm `main` -> `PROBLEM.Initialization` / `PROBLEM.Evaluation` -> `ALGORITHM.NotTerminated` -> `outputFcn` / metrics.

Then open the specific algorithm/problem/helper function actually used in that chain.

## Source-First Rules

Read local source before answering questions such as "what parameters does this algorithm support", "where is this metric computed", "how are labels parsed", "where are saved results written", or "why did GUI filter this file".

Use file headers as structured metadata.

- The first comment line is typically the class definition.
- The second comment line often contains GUI labels such as year, objective-count class, encoding, and difficulty tags.
- Subsequent comment lines often contain parameter declarations in the form `name --- default --- description`.
- Reference blocks in comments often point to the original paper.

Do not trust stale mental models of folder names.

- Algorithms and problems are added frequently.
- Some capabilities are new enough that older tutorials will be incomplete.
- When in doubt, search the current tree and read the current implementation.

## Practical Notes

Use `rg` to find classes and helper functions quickly.

- `rg --files PlatEMO/PlatEMO/Algorithms`
- `rg --files PlatEMO/PlatEMO/Problems`
- `rg -n "classdef .* < ALGORITHM" PlatEMO/PlatEMO/Algorithms`
- `rg -n "classdef .* < PROBLEM" PlatEMO/PlatEMO/Problems`
- `rg -n "function .*CalMetric|feval\\(metName" PlatEMO/PlatEMO/Problems PlatEMO/PlatEMO/Metrics`

If MATLAB emits warnings about nonexistent directories or odd startup hooks, treat that as a MATLAB environment issue first, not a PlatEMO bug. Verify the working directory, `path`, `startup.m`, and any user exit hooks before changing PlatEMO code.
