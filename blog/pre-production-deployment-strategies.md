# Pre-Production deployment strategies

1. Product finalises a release; tags tickets with the release number in Jira; plans Sprint, and development starts for that specific release version.
2. Freezing of Release Candidate for QA to test and Sign-off for production release

Once deployed to Pre-Prod, and QA starts testing, we cannot push any new changes to any services; except the bug fixes that have to go in this release.

3. Developers should get an environment to test their development independently. It should be possible if a front-end engineer wants to try their feature integration with a back-end service while it is live or in development.
   1. Developers can use live (pre-prod) services via the link for almost all applications and point to in-development services from test deployments ensuring non-redundant deployments. Developers will have to deploy the changed services for testing manually. Manual configuration of changes is a fair practice as developers are clear about what all services needs modifications compared to the current production setup.


Ref:

- [Testing Microservices: an Overview of 12 Useful Techniques - Part 1](https://www.infoq.com/articles/twelve-testing-techniques-microservices-intro/)