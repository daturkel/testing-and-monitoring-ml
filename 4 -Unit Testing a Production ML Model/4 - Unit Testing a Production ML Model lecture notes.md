## Unit testing a production ML model

### Section overview

This section will apply previous lessons to a more realistic codebase.

### Code conventions

The codebase uses PEP8 style.

Codebase also uses type hints per PEP484.

F-strings per Pep498.

Some functions force keyword arguments so that argument names have to be supplied, e.g.:

```python
def my_func(*, foo):
    return foo

my_func(foo="bar")
```

### Pytest

#### Fixtures

Functions like a super-charged version of `unittests` setup and teardown. They can be scoped at different levels of the project and passed to other fixtures.

```python
import pytest

@pytest.fixture
def input_value():
    return 4

def test_fixture(input_value):
    assert input_value == 4
```

Fixtures used across files can be put in `conftest.py` unless you specify a scope in the `@pytest.fixture` decorator.

#### Parametrization

Using the `pytest.mark.parametrize` decorator, we can pass multiple parameters to a test.

```python
@pytest.mark.parametrize(
    'inputs', [2, 3, 4]
)
def test_square(inputs):
    # the items in 'inputs' are passed one at a time
    assert isinstance(inputs, int)
```

### Setup - Kaggle data

We download our dataset from [Kaggle](https://www.kaggle.com/c/house-prices-advanced-regression-techniques/data) and put it in `packages/gradient_boosting_model/gradient_boosting_model/datasets`, renaming `train.csv` to `houseprice.csv`.

### Setup - Tox

Tox creates (multiple) virtual environments for running tests in reproducible setups across different Python versions.

Tox is configured in the `tox.ini` file, e.g.:

```ini
[tox]
envlist = py37
skipsdist = True

[testenv]
deps = pytest
commands =
	pytest
```

Then the commands can be run by just running `tox`.

![](./tox_flow.png)

