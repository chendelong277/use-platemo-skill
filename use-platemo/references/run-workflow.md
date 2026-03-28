# PlatEMO Run Workflow

## Core Facts

- Local root: `PlatEMO/PlatEMO`
- Main entrypoint: `platemo.m`
- Verified locally: `matlab -batch` can run `platemo(...)` from this checkout
- Version clues in this repo:
  - README highlights PlatEMO 4.14
  - `Doc/releasenote.md` includes updates through 2026-01

## Shell Pattern

Run PlatEMO from the shell like this:

```bash
matlab -batch "cd('PlatEMO/PlatEMO'); [Dec,Obj,Con]=platemo('algorithm',@GA,'problem',@SOP_F1,'N',20,'maxFE',40);"
```

Why this pattern:

- Run it from the repository root so the relative `cd('PlatEMO/PlatEMO')` resolves cleanly
- `cd` into the PlatEMO root so relative paths and `Data/` output land in the expected place
- `platemo.m` itself adds subdirectories with `addpath(genpath(cd))`
- Request output arguments when you want the final `Dec`, `Obj`, and `Con` directly

## Minimal MATLAB Patterns

### Benchmark problem

```matlab
platemo('algorithm',@NSGAII,'problem',@DTLZ2,'M',3,'N',100,'maxFE',10000);
```

### Algorithm or problem parameters via cell

```matlab
platemo('algorithm',{@KnEA,0.4},'problem',{@WFG4,16},'M',5);
```

The first element is the class/function handle. The remaining cell elements become `parameter` values consumed by `ParameterSet(...)`.

### User-defined problem from functions

```matlab
f1 = @(x)x(1)+sum(x(2:end));
f2 = @(x)sqrt(1-x(1)^2)+sum(x(2:end));
g1 = @(x)1-sum(x(2:end));
platemo('algorithm',@NSGAII,'objFcn',{f1,f2},'conFcn',g1,...
        'encoding',[1 1 1 2 2 4],'lower',0,'upper',[1 9 9 9 9 9]);
```

### User-defined problem with a single evaluation function

```matlab
function [x,f,g] = Eval(x)
    x = [round(x(1)/0.1)*0.1,x(2:end)];
    x = max(0,min([1,9,9,9,9,9],x));
    f(1) = x(1)+sum(x(2:end));
    f(2) = sqrt(1-x(1)^2)+sum(x(2:end));
    g = 1-sum(x(2:end));
end

platemo('algorithm',@NSGAII,'evalFcn',@Eval,...
        'encoding',[1 1 1 2 2 4],'lower',0,'upper',[1 9 9 9 9 9]);
```

Use this when repair, objectives, and constraints are coupled enough that splitting them across `decFcn`, `objFcn`, and `conFcn` is awkward.

### Batched evaluation

```matlab
d = rand(10)*2-1;
[d,~] = qr(d);
f1 = @(x,d)sum((x*d-0.5).^2,2);
platemo('objFcn',f1,'encoding',ones(1,10),'data',d,'once',1);
```

Set `once` to `1` when `evalFcn`, `decFcn`, `objFcn`, or `conFcn` can accept a matrix of solutions. This often matters for expensive evaluations.

## Result Collection

### Return the final population directly

```matlab
[Dec,Obj,Con] = platemo('algorithm',@GA,'problem',@SOP_F1);
```

### Save intermediate populations to MAT files

```matlab
platemo('algorithm',@NSGAII,'problem',@DTLZ2,'save',6,'run',3);
```

Saved files go under:

```text
PlatEMO/PlatEMO/Data/<AlgorithmName>/
```

The default output code writes files named like:

```text
<Algorithm>_<Problem>_M<M>_D<D>_<run>.mat
```

### Save metrics together with results

```matlab
platemo('algorithm',@NSGAII,'problem',@DTLZ2,...
        'save',6,'metName',{'IGD','HV'});
```

Metric calculation is driven by:

- `ALGORITHM.CalMetric`
- `PROBLEM.CalMetric`
- the selected metric file under `PlatEMO/PlatEMO/Metrics`

## GUI

Run:

```matlab
platemo();
```

Important version boundary from local source:

- `platemo.m` blocks GUI on MATLAB versions below R2020b
- `platemo.m` blocks non-GUI usage on MATLAB versions below R2018a

## What To Read Before Changing Invocation Code

- `PlatEMO/PlatEMO/platemo.m`
- `PlatEMO/PlatEMO/Algorithms/ALGORITHM.m`
- `PlatEMO/PlatEMO/Problems/PROBLEM.m`
- `PlatEMO/PlatEMO/Problems/UserProblem.m`

If behavior does not match your expectation, read these files first instead of patching call sites blindly.
