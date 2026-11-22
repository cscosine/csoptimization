---
name: csoptimization-libs
description: The cscosine optimisation C++ libraries released by csoptimization - csNelson (nonlinear least squares like g2o - sections of variables, unary / binary / n-ary edges, Gauss-Newton, Levenberg-Marquardt, Schur complement for bundle adjustment) and csBlockMatrix (block-structured dense and sparse matrices on Eigen, DimensionDescriptor). Use when C++ code in a cscosine project needs a nonlinear least-squares solver, bundle adjustment, pose-graph or block-sparse linear algebra, includes a csNelson / csBlockMatrix header, or its CMakeLists.txt finds one of these packages.
---

# The csoptimization libraries

csoptimization builds and releases two first-party libraries (C++17, on Eigen and csCore; csCMake packages), each in
its own repository: `cscosine/csNelson`, `cscosine/csBlockMatrix`. Both are **early stage**: their API is not settled;
read the library's `ASSESSMENT.md` (design review, next steps) and its tests before building on it, and copy the
patterns of the tests rather than guessing.

## Which library for what

| Need | Library | Where to look |
|---|---|---|
| nonlinear least squares (calibration, pose graph, bundle adjustment) | csNelson | headers `csNelson/*.hpp`; tests `csNelson/tests/` (one problem per section layout); `samples/` (`Problems.hpp`, performance samples on a bundle adjustment problem) |
| block-structured matrices (Hessians, Schur complement), dense or sparse | csBlockMatrix | headers `csBlockMatrix/*.hpp`; `samples/`; tests |

csNelson's model (from its README and headers):

- **Sections**: the groups of variables. `SingleSection` (one group); `DoubleSection` (two groups, with the U / V / W
  partition of the Hessian for the Schur complement, e.g. cameras and points of a bundle adjustment).
- **Edges**: the residuals. `EdgeUnary`, `EdgeBinary`, `EdgeNary` (CRTP bases, e.g.
  `EdgeUnarySectionBaseCRPT<Section, SectionAdapter, Derived>`): the user writes the residual, the Jacobians and the
  Hessian block updates.
- **Solvers**: `GaussNewton`, `LevenbergMarquardt` (`csNelson/GaussNewton.hpp`, `LevenbergMarquardt.hpp`).
- The matrices are csBlockMatrix ones: dimensions described by a `DimensionDescriptor` (blocks of compile-time,
  runtime or variable size; a compile-time or dynamic number of blocks), dense or sparse layouts
  (`DenseMatrixBlock`, `SparseMatrixBlock`, `DiagonalMatrixBlock`, `VectorBlock`, `SparsityPattern`).

For the Lie group parametrisation of poses (exp / log, Jacobians) use csLie (csfoundation plugin, `csfoundation-libs`).

## In CMake (csCMake)

```cmake
cs_find_package(myLib PUBLIC csNelson REQUIRED CONFIG)          # brings csBlockMatrix, csCore, Eigen3
cs_target_link_libraries(myLib PUBLIC csNelson::csNelson)
cs_find_package(myLib PUBLIC csBlockMatrix REQUIRED CONFIG)     # csBlockMatrix alone
cs_target_link_libraries(myLib PUBLIC csBlockMatrix::csBlockMatrix)
```

Each package has one default component named like it (`csNelson`, `csBlockMatrix`); their dependencies (Eigen3,
csCore `core` / `eigenUtils`, csBlockMatrix for csNelson) are found by their package configs, and must be installed.

## Getting them into a project

A csorchestrator project takes the prebuilt packages from the csoptimization release (its provider): in
`<name>_config.py`, a `ManifestGithub(org="cscosine", git_repo="csoptimization", project_name="csoptimization",
project_version="0.1.0", release_tag="v0.1.0")` and the libraries each target needs, e.g.
`{"csoptimization": {"myApp": ["csNelson"]}}`; csoptimization's own providers (csfoundation, third_party_base_libs)
come with it. Then `./<name>_project.py generate-github-workflow`. See the `csorchestrator-usage` skill of the
`csorchestrator` plugin.

Release versions: the [csoptimization releases](https://github.com/cscosine/csoptimization/releases). Changing a
library itself: its repository, built and validated by csoptimization's CI (an inner repository: no CI of its own).
