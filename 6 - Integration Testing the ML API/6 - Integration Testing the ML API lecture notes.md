# Integration testing the ML API

## Section overview

Will discuss:

- adding a Flask API to our codebase
- running our service via docker-compose
- adding integration tests

## API conceptual guide

We're now adding a Flask app which exposes a REST API to serve our recommendations.

We're going to specify our endpoints not in code but in an Open API (formerly known as Swagger) specification. This is useful for keeping (automated) documentation up to date.

Our API will validate incoming requests against expectations before passing it to our model. Then the model makes predictions and the Flask app returns predictions back to the client.

Our integration tests ensure functionality works as expected *across* the API and model components.

## Overview of the codebase

This section takes place at [this commit](https://github.com/trainindata/testing-and-monitoring-ml-deployments/commit/8c6dd5921d75a4b8d479777a9278ce08752424cc).

We now have an `ml_api` package in our packages directory. It requires our gradient boosting model (as a published package) as a dependency.

It also requires Flask and [Connexion](https://github.com/zalando/connexion), which allows you to define your API using Open API and have this interoperate with Flask.

To create an app, we point Connexion at our specification and it handles mapping all the routes to the Python functions which handle the corresponding endpoints. (I.e. the spec says `/foo/bar` is handled by Python method `app.foo_bar`.)

The `controller` module contains the handlers for the health check endpoint and the prediction endpoint.

The `Makefile` runs our app.

Lastly, we have a Dockerfile and `docker-compose.yml`. The Dockerfile is for our Flask app and the `docker-compose` just tells is to expose port 5000 and what command to run when it starts (which is the command specified in the Makefile). It's *not* attaching more than one Docker container together.

(Decent Makefile guide [here](https://opensource.com/article/18/8/what-how-makefile).)

## Using our Open API spec (part 1)

This section uses the [same commit](https://github.com/trainindata/testing-and-monitoring-ml-deployments/commit/8c6dd5921d75a4b8d479777a9278ce08752424cc) as the last.

In the `ml_api` directory, we use the command:

```
docker-compose -f docker/docker-compose.yml up -d --build
```

The `-f` argument specifies the docker-compose file, `-d` runs in the background, and `--build` (re)builds it first.

## Using our Open API spec (part 2)

Now we'll *use* the API.

Going to `localhost:5000` we get the health-check response with a 200 status code.

When we set the `connexion` dependency for our API, we included the `swagger-ui` optional packages. For free we get `localhost:5000/ui`, a Swagger UI with documentation and demos of each endpoint.

For our prediction endpoint, in order to have an example request in our Swagger UI, we defined a schema with example values in our `api.yaml` specification under `components.schemas.HouseDetails` which we link to the endpoint via `content.application/json.items.$ref`. (This sounds more confusing than it is; see the `api.yaml` file.)

We can now use the Swagger UI to try out our prediction endpoint and we see that it returns a response with a prediction and no errors.

## Integration testing theory

## Integration testing hands-on code

## Note on benchmark integration tests