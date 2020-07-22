## Testing Concepts for ML Systems

### Testing focus in this course

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

### Why test?

Why should we write tests at all?

If part of your business is software-based, you should be invested in that software doing what you expect it to do.

**Confidence:** You want to have confidence in your system. Do this by analyzing data provided by monitoring past system behavior and making predictions from data about that past behavior.

**Predicting:** We want to know that our software will be reliable in the future. What uncertainty will be incurred by change?

**Functionality:** Functionality should remain unchanged unless we're deliberately changing it. Testing is the mechanism by which we ensure the functionality has not changed.

**Testing is the way we show our system functionality is what we expect it to be, even as we make changes to the system.**

Testing's real value is shown *when the system changes*.

Not testing is like building up debt.

### Testing theory

Google's online [Site Reliability Engineering book](https://landing.google.com/sre/sre-book/toc/index.html) has a lot of great info on testing.

The hierarchy of tests is:

- **unit tests** at the base; these test a separable unit of code independent of other components
- **integration tests** in the middle; these test that a module made of multiple components is working as expected
- **system tests** at the top; these test the end-to-end functionality of the system

We typically have more unit tests than integration tests and more integration tests than system tests. (Some say that we should have more integration than unit tests, however.)

#### How much testing?

Required amount of tests can be seen as a function of the desired reliability of the system. 

Optimizing blindly for test coverage can lead to diminishing returns or even make development more difficult. But having very low coverage is dangerous as little of the functionality is tested. There is a balance.

#### What to test?

- Can you prioritize the codebase into sections whose functionality has greater importance?
- What is mission critical?
- **Does a test reduce uncertainty about how your system will function when it undergoes changes? If so, it's a useful test.**

### Testing machine learning systems

### Wrap up