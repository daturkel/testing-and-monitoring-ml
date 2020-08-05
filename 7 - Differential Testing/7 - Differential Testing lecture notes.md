# Differential testing

## Section overview

We'll discuss:

- What are differential tests, why do they matter?
- Applying differential tests to our code

Differential tests are an extension of integration tests, but they're particularly important to ML systems.

## Differential testing theory

A **differential test** compares the outputs of systems for a fixed input from one version of a system to the next. This could include, in our case, updates to a model or an entirely new model.

Differential tests are sometimes called "back to back" tests.

These are useful for finding bugs which don't raise errors or exceptions but which degrade performance.

We need to tune them to not miss degradations but also not be so sensitive to alert all the time.

Differential tests catch unanticipated errors rather than specific failures we expected could happen.

## Differential testing implementation

This section takes place at [this commit](https://github.com/trainindata/testing-and-monitoring-ml-deployments/commit/cf7a3f6b629d341f0fdf1bafa81c6e2610c4ebdc).

We've added our baseline regression model to our `controller.py` and updated our API schema to support a `/v1/predictions/regression` and `/v1/predictions/gradient` endpoint.

We will mark our tests with `@pytest.mark.differential` so we can run them on their own.

Our differential test makes a prediction request with the old model and one with the new model and compares the differences.

To tune our test, we set `abs_tol` and `rel_tol` parameters to supply to `math.isclose` to set boundaries on absolute and relative acceptable differences (tolerance).

We define a helper module for comparing predictions in `tests/compare.py` which has a `compare_differences` function we'll use in our tests. It raises a `ValueError` when differences are too high.

We make a separate test environment in `tox.ini` for differential tests and can run just that environment and it's command with `tox -e differential_tests`.