# Setting the Scene and ML System Lifecycle

## Deploying a model to production

Deployment is making our model available to production environments where it can provide predictions to other systems.

The **research environment** is a non-production environment where we can **train and evaluate** our model and see results. The **live environment** is where the model gets **live data**.

**The machine learning pipeline** is the series of steps between getting data and making predictions. It includes feature engineering, training, scoring, and predicting. We make this pipeline in the research environment before deploying to the production environment.

We want the research pipeline to be identical to the production pipeline. We optimize the model in the research environment to maximize value to the business, so we want this model to be optimal in the live setting as well.

### Deployment scenarios:

1. Putting our first model in production when there was none before
2. Replacing an existing model in production with a new one
3. Redeploying the live model with small tweaks or changes

This course follows scenario 2: replacing an old model with a new one.

Differential testing, also discussed in this course, is very relevant to scenario 3.

## Course Scenario: Predicting House Sale Price

In this scenario, we have a model in production and a newer model in the research environment. We want to replace the production model with the research model.

We are real estate agents who get commission that's a percentage of final house sale price. We want to forecast future revenue by **predicting the sale price of houses**.

### Our modeling data

We have:

- historical data
- data on the houses we have sold (neighborhood, when it was remodeled, overall quality, etc.)
- price houses were sold

### The model

The model was built in the Deploying an ML Model course. The pipeline is feature engineering followed by linear regression.

End-users input variables of a house they want to sell and get an estimated sale price.

The *new* model is a gradient-boosting model. Furthermore, the entire pipeline has changed: inputs are different, transformation is different, etc.

We need to not only handle replacing the old model, but ensuring that we see the gains in business value in production that we predicted in the research environment.

### Maximising reproducibility

We will use **unit testing**, **integration testing**, **differential testing**, and **shadow mode** to ensure reproducibility.

Furthermore, we will monitor the performance over time to ensure the model continues to perform well.

## Introduction to dataset & model pipeline

(Walks through the gradient boosting notebook in the "research phase" folder of the Git repo.)

## ML system lifecycle

Diagram from paper ["Hidden Technical Debt in ML Systems"](https://papers.nips.cc/paper/5656-hidden-technical-debt-in-machine-learning-systems.pdf) illustrates that the ML code in an ML system is a very small component of the larger system. Other components include data collection, feature extraction, serving infrastructure, and monitoring. This is why we need specialized testing to maintain a system like this.

We're especially interested in testing:

- data collection in production environment
- feature extraction
- data verification/validation (sanity checking and cleaning data)
- configuration (settings, hyperparameters, and so on)

Machine learning systems require tests that are specialized to the task of ML, where certain behaviors are not hard-coded in the code but are instead learned from data. Thus we need data tests and data monitoring (and more).

