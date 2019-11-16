01 ) Build script to install base servers for production environment, use any automation tools
suits you (ansible, saltstack, terraform, ...). The solution should setup following components,
make any changes as you see fit, so we can assess your consideration for production-grade
system

● sysadmin engineer accounts, hostname (dns), ntp, cli commands that you use
often, ...
● Install jenkins server

02 ) Write simple python cli with two sub commands to
● list all jobs with name contain a keyword:
○ Ex: ./cli find report
-> result: - auto_clear_request_report_queue
- auto_report_aging_daily

● trigger build for a specific job
○ Ex: ./cli build auto_clear_request_report_queue
-> result: jenkins job build link
Write Dockerfile to dockerize above application
Suggestion:
● Sensitive information such as jenkins user’s api-key is configured via
environment variables
● Use click framework (https://click.palletsprojects.com/en/7.x/), follow pep8 style
● Utilize docker layer caching to speed up image building process

03) Write a CI/CD pipeline using Jenkins pipeline to build, test, deploy above application to
Kubernetes cluster

● master branch to production cluster
● dev branch to dev cluster
Suggestion:
● Write shared library in order to reuse your code in pipeline
https://jenkins.io/doc/book/pipeline/shared-libraries/
