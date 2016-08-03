---
layout: post
title: CI / CD stack at PetalMD for web applications
---


# CI / CD stack at Petalmd for web applications
## Context
At [PetalMD](https://petalmd.com) we build cloud applications for Healthcare professionals. We have a backend running Ruby on Rails (API / admin app) and front-end apps in pure Javascript. All applications have unit/end-to-end tests and can be deployed as needed, typically several times a day.

The purpose of the article is to give a quick overview of what we have in place currently and begin a series of technical posts on our CI/CD stack.


## Required steps
- Each time a developper start a new project, he creates a branch and a corresponding Pull Request to master.

- When a PR is created or a new commit is made to a PR, the CI should rerun all tests and all previous approvals (from designers or product owners for example) should be reset. When a build is done (and successful), we should have a docker image ready to test/deploy.

- The entire test harness should run as fast as possible (unit and end-to-end tests). For our Ruby on Rails application, we have more than 12K tests and each build should take less than 1h to run.

- The code quality and code coverage should not be worse than master.

- Each time master builds (usually after a PR is merged), the application should be deployed in production if the build succeeds. If not, a notification should be sent to a Slack channel.

## CI/CD flow
![CI/CD flow](../assets/posts/ci_cd_stack.png)

## Tools selection
### GitHub
We previously tried Assembla, GitLab and Github for our versionning part and we now use Github principaly due to all the 3rd party integrations available. The second reason for our choice is the possibility to fork open source libraries we use and send back our changes in few clicks.

### Jenkins
We also looked at different testing SaaS tools (Travis, CircleCI, etc) but, for our requirements, the price was really an issue (a lot of concurrency, and to many minutes of build usage per month). We finally built our solution, based on the open-source project Jenkins.

### Docker
Within Jenkins, we need to setup different slaves per application type and, for the end-to-end tests, setup all services needed (MySQL, Redis, ElasticSearch, Sidekiq).  For the end-to-end tests on front-end applications, we need to configure Firefox, some virtual display (xvfb), etc.

The maintenance and cleanup each time we change versions added some complexity that we didn't want to deal with. This is why we use docker and docker compose: we run our applications in containers linked with a docker-compose file.

__The only requirement for a slave is now to have Docker and Docker compose installed.__

### EC2
We would like to always have slave for running our jobs as needed, and with the lowest cost possible.

Jenkins has an integration with AWS EC2 and Digital Ocean, but our tests are faster with a higher CPU clock and only the C3/C4 series from AWS have the clocking we want.

In order to reduce the build time even further, we split Ruby tests in 15 processes on a single machine, that runs MySQL, Elasticsearch and Redis at the same time. We use C4.4Xlarge that have 16 cores and 30G of RAM.

To reduce the price, we use spot instances in the Oregon zone (more stable and lower price currently). The regular price is 0.838 dollars per hour, in spot instance it's between 0.15 and 0.20 dollars per hour (18-24% of regular price).

### Pull Approve / Code Climate / Codecov
For our Code Quality tools, we choose them for their simplicity of use and the integration with GitHub.

- Each time Jenkins run a build, we upload our coverage data to Codecov.io.
- Each time a commit is made in a PR, Code Climate analyzes our code and we reset all approvals in the Pull Approve integration.

Codecov and Pull Approve have billing per private repository, Code Climate uses a per account pricing model.

## Next
- How to configure a Jenkins slave to auto connect to Jenkins master on start
- How to parallelize Ruby tests in different slaves with the Jenkins pipeline

