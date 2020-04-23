# Software quality in general

## Identify and list qualities of your MiniTwit system from the four perspectives (except of the transcendental) from the Kitchenham paper.

* user view (quality as fitness for purpose)
** reliability, responsiveness
** usability

* manufacturing view (quality as conformance to specification)
** automated continuous integration
** possible to make change without disrupting production
** discover bugs early
** peer-review

* product view (characteristics of the product)
** static code analysis (code metrics)
** web security

* value-based view (amount a customer is willing to pay for it)
** no monetization, so not relevant

## Did you focus on any perspective or any qualities, perhaps even without being aware of it? If yes, list these.
We mostly focused on reliability from the user’s view, aiming for reliability, low response time, and low number of dropped requests. We also focused on the manufacturing view to some extent by setting up a CI pipeline with static code analysis, and when focusing on security issues.

## Rank the identified qualities per perspective by decreasing importance to you and provide an argument for why you choose certain as the most important.
Reliability from user view
Manufacturing view
Web security from product view
Usability from user view
This was mostly motivated by the tasks from the lecture and how “performance” is measured.

## Think about and discuss with your group fellows, how you can measure the qualities that you ranked the most important. That is, try to define a set of metrics that would allow to measure these (multiple metrics per quality can be possible).

* Grafana dashboard for reliability (avg. response time etc)
* number of bugs that reach production (maybe also severity of bugs)
* Sonarcloud metrics for product view

# How maintainable are your systems

## How can you identify and measure maintainability of your MiniTwit systems

In order to measure how maintainable the project is we will be looking at the following measures:

- Code Coverage
  - Lines to cover
  - Uncovered lines
- Duplications
- Code smells
- Security vulnerabilities and hotpots

These factors will be taken into consideration when calculating the technical debt in our project. Our maintainability will be calculated on SonarCloud.io. Which can be seen [here](https://sonarcloud.io/dashboard?id=backendlazyops).

## MiniTwit maintainability and technical debt

The system is maintainable as a proper architectural style has been implemented in the Backend API. And thus allows a lot of modularity. Looking at the analysis of the code, we can see that the solution only has a 3 hours debt, and grad A from SonarCloud.

The main issues with the current codebase is not enough code coverage, minor security hotpots and useless code that should be removed.

Besides the codebase the infrastructure provided by CI/CD allows us to easily deploy a new update to the production environment. The infrastructure is built on docker swarm, which will deploy only to one instance out of 4 available. This should result in nearly 0% downtime, thus keeping our user base satisfied.
