# PlatEMO Source Map

## Read Order

Use this order when you need to understand behavior fast:

1. `PlatEMO/PlatEMO/platemo.m`
2. `PlatEMO/PlatEMO/Algorithms/ALGORITHM.m`
3. `PlatEMO/PlatEMO/Problems/PROBLEM.m`
4. `PlatEMO/PlatEMO/Problems/UserProblem.m`
5. The target algorithm, problem, metric, or GUI module

This gets you from public API to runtime behavior with minimal guessing.

## Directory Map

### Documentation

- `PlatEMO/README.md`
  - Project overview, release highlight summary, citation
- `PlatEMO/Doc/releasenote.md`
  - Change log by version; use it to check whether a capability is likely new
- `PlatEMO/PlatEMO/manual.pdf`
  - User manual covering CLI, GUI, and extension points

### Runtime entrypoint

- `PlatEMO/PlatEMO/platemo.m`
  - Handles no-argument GUI launch
  - Parses name-value arguments
  - Chooses default problem or algorithm when omitted
  - Builds the problem first, then the algorithm
  - Returns `[Dec,Obj,Con]` when called with output arguments

### Algorithms

- `PlatEMO/PlatEMO/Algorithms/ALGORITHM.m`
  - Base class for algorithms
  - `Solve` owns runtime setup
  - `NotTerminated` stores progress, updates runtime, and triggers output
  - `ParameterSet` maps user-provided parameters over defaults
- `PlatEMO/PlatEMO/Algorithms/Multi-objective optimization/`
- `PlatEMO/PlatEMO/Algorithms/Single-objective optimization/`
- `PlatEMO/PlatEMO/Algorithms/NeuroEA/`
- `PlatEMO/PlatEMO/Algorithms/Utility functions/`
  - Common helpers like `OperatorGA`, `OperatorDE`, `NDSort`, `TournamentSelection`, `UniformPoint`

### Problems

- `PlatEMO/PlatEMO/Problems/PROBLEM.m`
  - Base class for benchmark problems
  - Owns initialization, evaluation, repair, metrics, and visualization defaults
- `PlatEMO/PlatEMO/Problems/UserProblem.m`
  - Adapts name-value function definitions into a problem object
  - Converts strings, function handles, matrices, and external data into executable behavior
- `PlatEMO/PlatEMO/Problems/Multi-objective optimization/`
- `PlatEMO/PlatEMO/Problems/Single-objective optimization/`
- `PlatEMO/PlatEMO/Problems/SOLUTION.m`
  - Container type for populations and utility accessors like `decs`, `objs`, `cons`, `best`

### Metrics

- `PlatEMO/PlatEMO/Metrics/`
  - Metric implementations such as `IGD.m`, `HV.m`, `GD.m`, `Spacing.m`
  - Read metric headers for labels and compatibility
  - Some problems override `CalMetric` to pass custom reference data

### GUI

- `PlatEMO/PlatEMO/GUI/GUI.m`
  - Loads algorithm/problem/metric lists and label filters
- `PlatEMO/PlatEMO/GUI/uilist.m`
  - Reads file headers to populate GUI metadata, summary text, parameters, and references
- `PlatEMO/PlatEMO/GUI/module_test.m`
- `PlatEMO/PlatEMO/GUI/module_app.m`
- `PlatEMO/PlatEMO/GUI/module_exp.m`
- `PlatEMO/PlatEMO/GUI/module_cre.m`

## Header Conventions

Many files encode important metadata near the top:

- Label line such as `% <2002> <multi> <real/integer/...> <constrained/none>`
- Short description line
- Optional parameter lines such as `% proC --- 1 --- Probability of crossover`
- Reference block

The GUI reads these comments directly. If you add or edit algorithms/problems/metrics, preserve this structure.

## Fast Search Patterns

### Find an algorithm class

```bash
rg -n "classdef .* < ALGORITHM" PlatEMO/PlatEMO/Algorithms
```

### Find a problem class

```bash
rg -n "classdef .* < PROBLEM" PlatEMO/PlatEMO/Problems
```

### Find where metrics are calculated

```bash
rg -n "CalMetric|feval\\(metName" PlatEMO/PlatEMO/Problems PlatEMO/PlatEMO/Metrics
```

### Find a helper used inside an algorithm

```bash
rg -n "OperatorGA|OperatorDE|NDSort|TournamentSelection" PlatEMO/PlatEMO
```

### Find label parsing logic in the GUI

```bash
rg -n "labelstr|<single>|regexp\\(.*<" PlatEMO/PlatEMO/GUI
```

## Runtime Trace

When you need to explain a run, use this sequence:

1. `platemo.m` parses inputs
2. Problem object is constructed
3. Algorithm object is constructed
4. `ALGORITHM.Solve` sets runtime state and calls `main`
5. The algorithm `main` calls problem methods such as `Initialization` and `Evaluation`
6. `ALGORITHM.NotTerminated` stores populations and drives termination
7. `outputFcn` or `DefaultOutput` displays or saves results
8. Metrics are computed through `ALGORITHM.CalMetric` and `PROBLEM.CalMetric`

## If Something Feels Inconsistent

Check the current tree before deciding something is missing:

- The repo updates often, so lists in older tutorials may be incomplete
- New labels or subdomains can appear in headers before you have seen them elsewhere
- Some problems and metrics override defaults in subtle ways

When a question is about real behavior, the current local source outranks memory and secondary docs.
