# Testing Concepts for ML Systems

## Testing focus in this course

This section clarifies which elements of testing are in scope for this course.

An ML system has three environments:

- research environment (Jupyter notebooks written by data scientists)
- development environment (make the research environment useable in the larger setting of our software)
- production environment

Research environment is out of scope for this course.

Development environment has:

- unit, integration, and acceptance tests
- differential testing
- benchmark testing
- load testing

Production environment has:

- shadow mode testing
- canary releases
- observability and monitoring
- logging and tracing
- alerting

## Why test?

Why should we write tests at all?

If part of your business is software-based, you should be invested in that software doing what you expect it to do.

**Confidence:** You want to have confidence in your system. Do this by analyzing data provided by monitoring past system behavior and making predictions from data about that past behavior.

**Predicting:** We want to know that our software will be reliable in the future. What uncertainty will be incurred by change?

**Functionality:** Functionality should remain unchanged unless we're deliberately changing it. Testing is the mechanism by which we ensure the functionality has not changed.

**Testing is the way we show our system functionality is what we expect it to be, even as we make changes to the system.**

Testing's real value is shown *when the system changes*.

Not testing is like building up debt.

## Testing theory

Google's online [Site Reliability Engineering book](https://landing.google.com/sre/sre-book/toc/index.html) has a lot of great info on testing.

The hierarchy of tests is:

- **unit tests** at the base; these test a separable unit of code independent of other components
- **integration tests** in the middle; these test that a module made of multiple components is working as expected
- **system tests** at the top; these test the end-to-end functionality of the system

We typically have more unit tests than integration tests and more integration tests than system tests. (Some say that we should have more integration than unit tests, however.)

### How much testing?

Required amount of tests can be seen as a function of the desired reliability of the system. 

Optimizing blindly for test coverage can lead to diminishing returns or even make development more difficult. But having very low coverage is dangerous as little of the functionality is tested. There is a balance.

### What to test?

- Can you prioritize the codebase into sections whose functionality has greater importance?
- What is mission critical?
- **Does a test reduce uncertainty about how your system will function when it undergoes changes? If so, it's a useful test.**

## Testing machine learning systems

Why is ML hard to test?

In traditional software, the code is the only thing to test. In ML systems, we **also have to test the model and the data**.

Traditional software is constructed deductively, whereas ML systems are constructed inductively. The rules governing an ML system are thus less clearly defined.

A useful paper is [The ML Test Score: A Rubric for ML Production Readiness and Technical Debt Reduction](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/aad9f93b86b7addfea4c419b9100c6cdd26cacea.pdf).

### Key testing principles

Pre deployment:

- Use a schema for features (schemas should expose feature expectations)
- Model specification tests (model configuration like hyperparameters need unit testing)
- Test input feature code (bugs in features are hard to detect further down the pipeline)
- Validate model quality prior to serving
- Training is reproducible (important!)
- Integration testing on the full pipeline

### Summary

| Unit testing                 | Integration testing   | System testing                                       | Production testing | Monitoring                                        |
| ---------------------------- | --------------------- | ---------------------------------------------------- | ------------------ | ------------------------------------------------- |
| Schema for inputs            | Reproducible training | Reproducible predictions                             | Shadow deployments | Monitor system serving performance (latency)      |
| Model specification          | Pipeline testing      | Model-infrastructure compatibility                   | Canary releases    | Monitor model quality (accuracy, skew, staleness) |
| Input feature code           |                       | Validate model quality (differential tests)          |                    |                                                   |
| Model quality (algorithmic)  |                       | Validate computational performance (benchmark tests) |                    |                                                   |
| Model quality (benchmarking) |                       | Validate system performance (load tests)             |                    |                                                   |

## Assignment 1: Unit testing input data

This assignment is notebook exercise_notebooks > unit_testing_exercise > unit_testing_input_data.ipynb

We implement an input schema unit test that checks that all data is within value ranges (that we set by looking at the min and max of the data) and that all data is of the expected data type.

This type of testing ensures that our assumptions about incoming data are true.

## Assignment 2: Unit testing data engineering code

This assignment is notebook exercise_notebooks > unit_testing_exercise > unit_testing_data_engineering.ipynb

We add a standard scaler to our pipeline and write tests that ensure it's working: is the mean 0 and is the standard deviation 1?

This type of testing ensures that our data engineering pipeline works as expected.

## Assignment 3: Unit testing model quality

This assignment is notebook exercise_notebooks > unit_testing_exercise > unit_testing_model_predictions_quality.ipynb

We write two types of tests:

- **benchmark test**: compare model accuracy to a simple benchmark
- **differential test**: compare model accuracy from one version to the next

Our benchmark in this Iris classification example just guesses class 1 for all examples.

Our two prediction models are the simple pipeline from Assignment 1 and the feature engineering pipeline from Assignment 2. The differential test ensures that feature engineered pipeline is better than the simple pipeline.

## Assignment 4: Unit testing model config

This assignment is notebook exercise_notebooks > unit_testing_exercise > unit_testing_model_configuration.ipynb

We introduce a pipeline with configuration options that get passed to our algorithm. Passing the configuration from our pipeline to the model is an example of **dependency injection**. 

In this scenario, we've found that there are configurations that we don't want to use, so our tests will ensure that bad configurations aren't being used.

We can think of this as a defense against misconfiguring the model after we've learned that certain configurations are bad.

## Wrap up

Next up we're going to leave Jupyter and enter a realistic codebase.

