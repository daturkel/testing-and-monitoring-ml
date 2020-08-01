# Unit testing a production ML model

## Section overview

This section will apply previous lessons to a more realistic codebase.

## Code conventions

The codebase uses PEP8 style.

Codebase also uses type hints per PEP484.

F-strings per Pep498.

Some functions force keyword arguments so that argument names have to be supplied, e.g.:

```python
def my_func(*, foo):
    return foo

my_func(foo="bar")
```

## Pytest

### Fixtures

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

### Parametrization

Using the `pytest.mark.parametrize` decorator, we can pass multiple parameters to a test.

```python
@pytest.mark.parametrize(
    'inputs', [2, 3, 4]
)
def test_square(inputs):
    # the items in 'inputs' are passed one at a time
    assert isinstance(inputs, int)
```

## Setup - Kaggle data

We download our dataset from [Kaggle](https://www.kaggle.com/c/house-prices-advanced-regression-techniques/data) and put it in `packages/gradient_boosting_model/gradient_boosting_model/datasets`, renaming `train.csv` to `houseprice.csv`.

## Setup - Tox

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

## Code-base overview

This overview is based on the state of the repo at [this commit](https://github.com/trainindata/testing-and-monitoring-ml-deployments/tree/01738f4ebade0824a31576747e0e4d5a0615e146).

The code-base is a mono-repo: there are multiple packages in one repository.

One package is `gradient_boosting_model`.

Dependencies of `gradient_boosting_model`:

- numpy, sklearn, pandas
- feature_engine (by Train In Data, the people that make the course)
- joblib will be used for serialization
- strictyaml for parsing yaml configs
- pydantic also for parsing
- marshmallow for creating a schema
- setuptools and wheel for packaging

### Modules of gradient boosting model

#### train_pipeline

This module contains the `run_training` method which loads data, divides it, fits the pipeline, and persists the pipeline for later transformation use.

#### pipeline

An sklearn pipeline. Encodes variables, drops unnecessary features, and contains the gradient boosting regressor as the final step.

#### config.yml

Specifies which features we expect, which features are categorical/numerical, which features cannot have missing data, train split proportions, random states, hyperparameters, and so on.

#### config

The config.yml is loaded by `config/core.py`. The `Config` class contains `ModelConfig` and `AppConfig`, all of which are Pydantic models.

#### predict

This module defines `make_prediction` which uses the `predict` function of our pipeline.

#### processing

Module containing various data and pipeline processing code.

##### data_management

Helpers for loading dataset and persisting dataset.

##### preprocessors

Steps which get fed into our pipeline and transform features.

##### validation

A Marshmallow schema for sanitizing and validating inputs. The aforementioned `make_predictions` function uses this to validate all input data before predicting.

##### errors

Defines custom exceptions.

## Preprocessing and feature engineering testing theory — why do this?

Feature engineering/preprocessing is the very top of the pipeline and so has the possibility to introduce issues before the model code even begins. Bugs in features can be almost impossible to detect further downstream.

Preprocessing code may also be designed by a DS team and implemented or productionized by a data engineering team, so testing ensures that expected behavior is maintained throughout this process.

### Examples:

- A feature is within a set scale (e.g. 0–1)
- Calculations are correct
- Missing data is correctly replaced
- Do distributions of transformed data match expectations?
- Are outliers handled as desired?

## Preprocessing and feature engineering unit testing

This section takes place at [this commit](https://github.com/trainindata/testing-and-monitoring-ml-deployments/commit/f8ddadf448207bf250e8d9ad84aedcd4db12e1db).

In `conftest.py`, we create a `pipline_inputs` fixture that provides train and test splits of our data.

In `test_preprocessors.py`, we test:

- does the pipeline drop unnecessary features?
- does the temporal variable transformer work?

In `test_pipeline.py`, we test the same functionality as above, but we're ensuring that this happens when we run *the full pipeline*. We use the `_fit` method rather than `fit` because it returns the transformed input data.

## Model config unit testing theory — why do this?

Model specification might seem like "configuration," but it can still have bugs!

If we know there are certain options that lead to bad performance, we should test that we aren't using them.

ML systems may depend heavily on model configurations, so even small configuration changes can lead to large change sin behavior. Thus configuration testing is essential in ML systems.

One tricky situation is when **configurations implicitly rely on code-defined defaults**. 

Another tricky situation is when **configurations are preprocessed by some script**.

## Model config unit testing

This section takes place at [this commit](https://github.com/trainindata/testing-and-monitoring-ml-deployments/commit/dcb0636279799b5b94ae6c3b25a27706cd4780c3).

This commit adds one file: `test_config.py`.

The `test_fetch_config_structure` test makes use of [pytest's built-in `tmpdir` fixture](https://docs.pytest.org/en/stable/tmpdir.html#the-tmpdir-fixture) to give us a place to write test files.

One test ensures a config can be loaded and populate the config object correctly.

Another test ensures that Pydantic raises a `ValidationError` when an invalid config is supplied.

One cool trick in the Pydantic model is that the config includes `loss` and `allowed_loss_functions`, and the validator method for `loss` ensures that the former is in the latter.

```python
@validator("loss")
def allowed_loss_function(cls, value, values):
    allowed_loss_functions = values.get("allowed_loss_functions")
    if value in allowed_loss_functions:
        return value
    raise ValueError("...")
```

This defends against mis-configurations down the line.

One last field checks that missing required fields lead to a Pydantic `ValidationError`.

## Input data testing theory — why do this?

This relates to testing raw input data, though the methods may look similar to how we test feature engineering (checking the values are within ranges, of the right type, etc.).

Ideally we can encode assumptions about the input data into a schema.

Catching errors this early is helpful because it can signal upstream issues, save us a lot of time, and give more verbose explanation of what's wrong than catching the problem further down the pipeline would.

### Implications of erroneous feedback loops

If predictions are used to generate more training data, this can create a negative feedback loop that causes data errors and issues in model performance to amplify over time.

## Input data unit testing

This section takes place at [this commit](https://github.com/trainindata/testing-and-monitoring-ml-deployments/commit/fd102c92ab591bc76041241c1a26bc05c221e122).

We add a new fixture to `conftest.py`: `sample_input_data`, which loads test data from a test file.

We introduce `test_validation.py`. Validation of our input data uses a [marshmallow](https://marshmallow.readthedocs.io/en/stable/) `Schema` class and includes methods that drop rows where certain columns have null values and so on.

We test that the data is valid and we **also test that our ability to detect errors in the data is working** (e.g. does the schema throw an error given invalid data?).

We add a method to `test_pipeline.py`, again zooming out to test that the pipeline can create predictions given validated data. (This is sort of an integration test.)

As an aside, why marshmallow *and* Pydantic? The course notes say this:

> A reasonable question to ask is: Why do we need marshmallow *and* Pydantic. The answer is that Pydantic is primarily a parsing library, not a validation library. Validation is a means to an end: building a model which conforms to the types and constraints provided.
>
> See this section of the Pydantic documentation which discusses this distinction:
>
> https://pydantic-docs.helpmanual.io/usage/models/
>
> This github issue also clarifies the distinction:
>
> https://github.com/samuelcolvin/pydantic/issues/578

An example: Pydantic will silently perform type conversions (when possible) so that the model conforms to the schema. A model that expects `a: int` will take `a = '1'` and output `a = 1`—it does *not* consider `a = '1'` an invalid input, it just converts it so that it is a valid output.

Marshmallow is more about ensuring that the raw inputs themselves conform to certain expectations.

## Model quality unit testing theory — why do this?

This is the least surprising category of ML testing. We want to test that the model behaves as expected *after training* but *before serving that model in production*.

As a guideline, keep the tests deterministic: seed your random number generators if necessary.

Also keep the tests short. They should be atomic and allow you to, e.g., test something related to outputs without having to retrain the entire model.

### Two types of degradation to test for

1. **Sudden degradation**: this typically happens with a bug in a new version where suddenly the model quality drops a lot
2. **Gradual degradation**: this is harder to detect: we don't just want to ensure that model performance over time is within a relative threshold as it could slowly drift downwards over time. We should have fixed thresholds in place that tell us if the model has dropped below an acceptable performance, and those thresholds may need to be updated over time if we have to change our validation data to match changes in live data.

It is often important to create test datasets that exploit edge-cases to ensure that performance isn't just a matter of luck or chance.

## Model quality unit testing

This section takes place at [this commit](https://github.com/trainindata/testing-and-monitoring-ml-deployments/commit/2ca89131c60c7e0085461949c84fc14358da73be).

We add a new dependency to `requirements.txt`: `tid-regression-model`, a regression model from the sister Deploying ML Models course. It's based on lasso regression from sklearn and serves as our baseline.

We add a new fixture to `conftest.py`: `raw_training_data` gives us the full, unsplit dataset.

We add `test_predict.py`:

- test against a fixed benchmark (in this case, we're testing that the predicted prices are within a fixed distance from their true value)—this protects against gradual degradation
- test against another model to check our model has lower MSE (we test against the regression model)—this protects against sudden degradation

We also add `train_pipeline.py` to our tox pipeline because we need access to a trained model in order to make predictions.

## Quick lecture on tooling improvements

This section takes place at [this commit](https://github.com/trainindata/testing-and-monitoring-ml-deployments/commit/ebf76911f36bce6863fa2e7aeaa9409d944c2dde).

We split the `requirements.txt` into `requirements.txt` and `test_requirements.txt`. 

To `test_requirements.txt`, we add `black` for code-formatting, `flake8` for PEP8 enforcement, and `mypy` for type-checking.

Then we add `flake8` and `mypy` to `tox.ini`.

New test environments in `tox.ini` look like this:

```ini
[testenv:typechecks]
envdir = {toxworkdir}/unit_tests

deps =
     {[testenv:unit_tests]deps}

commands = {posargs:mypy gradient_boosting_model}
```

Now our tox output looks like this:

```
  unit_tests: commands succeeded
  typechecks: commands succeeded
  stylechecks: commands succeeded
  congratulations :)
```

## Wrap-up

We focused on four sections:

- data engineering tests (reduce risk of bugs in preprocessing and feature engineering)
- input data tests (catch unexpected inputs)
- config tests (reduce risk of mis-configuration)
- model quality tests (reduce risk of sudden and gradual drops in quality)

As an additional resource, see Martin Fowler's article on [Continuous Delivery for Machine Learning](https://martinfowler.com/articles/cd4ml.html).

Next we will discuss **integration testing**, but we'll first make a detour into Docker and Docker-compose.