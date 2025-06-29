# AI Backend Infrastructure Demo

This project is a scalable backend system for image classification. It is designed to handle a high volume of requests and is built using Docker, Node.js, Python, and Nginx.

The system is composed of two main services: classifier-interface and classifier. The classifier-interface service is responsible for handling incoming requests, performing authentication, validation, and sanitation. It then forwards image inference requests to the classifier service. The classifier service is responsible for the actual image classification.

The system is designed to be scalable. It uses Nginx as a reverse proxy and Docker's hostname resolution to implement two load balancers. This allows the system to distribute incoming requests across multiple instances of the classifier-interface and classifier services. The number of instances can be easily adjusted in the docker-compose.yml file to meet the demand.

The system also includes security features such as rate limiting, JWT authentication, and input sanitation. It is also designed to be resistant to CSRF attacks.

For more detailed information about the system architecture, load tests, and security features, please refer to the relevant sections in the README file.

## How to run

In order to run the entire stack, go to the root folder and execute the following command:

```
docker-compose up --build
```

Then load the rudimentary client file at `client/infer_client.html` to test the services.

## Security


### Rate limiting

Rate limiting has been implemented so no more than 100 resquest every 15 minutes can come from the same IP

### Authentication

A basic JWT authentication has been implemented. Every new visitor to the basic client is assigned to a random identifier and given a token that allows them to use the API for 1h before expiring. JWTs need to be passed in the header of every request to the API.

### Other security concerns

The stack should be fairly protected against CSRF attacks due to JWT authentication combined with a CORS policy (Not fully configured since no front-end infra is defined).

Input sanitation, validation and CSP is also implemented.

## Scalability

In order to make the services in the stack scalable two load balancers were implemented using Ngix as a reverse proxy and Docker's hostname resolution. This way when external requests come to the external balancer it's passed to the `classifier-interface` service instances which performs authentication, validation and sanitation and then it forwards a image inference request to the internal load balancer that distributes the requests between the `classifier` instances.

### Load tests

The following three load tests were performed against the services. As it can be noticed, scaling up the number of `classifier` instances improve RPS processed and P50/P95 response. For at least 2 `classifier` instances adding a second `classifier-interface` didn't have a noticeble benefit. More hardware resources are needed for testing the stack with more instances, but as it can be noticed here and as it should be expected, the `classifier` service is doing most of the heavy-lifting and there should be more instances of it than `classifier-interface` if we want to serve more requests.
