# BUILD FULL DEVOPS TOOLCHAIN USING CENTOS
## 01 ) Build script to install base servers

- sysadmin engineer accounts, hostname (dns)
- ntp, cli commands that you use
often, ...
- jenkins server

## 02 ) Write simple python cli with two sub commands
- list all jobs with name contain a keyword. Ex: 
```
./cli find report
# result
auto_clear_request_report_queue
auto_report_aging_daily
```
- trigger build for a specific job. Ex:
```
./cli build auto_clear_request_report_queue
# result
jenkins job build link
```

- Write Dockerfile to dockerize above application:
  - Sensitive information such as jenkins user’s api-key is configured via
environment variables
  - Use click framework (https://click.palletsprojects.com/en/7.x/), follow pep8 style
  - Utilize docker layer caching to speed up image building process

## 03) Write a CI/CD pipeline using Jenkins pipeline to build, test, deploy above application to Kubernetes cluster

- master branch to production cluster
- dev branch to dev cluster
- Write shared library in order to reuse your code in pipeline https://jenkins.io/doc/book/pipeline/shared-libraries/

## REFERENCE
https://github.com/ernesen/infra-ansible
https://medium.com/design-and-tech-co/end-to-end-automated-environment-with-vagrant-ansible-docker-jenkins-and-gitlab-32bb91fbee40
https://github.com/wunzeco/ansible-jenkins-extra


## HOW TO RUN THIS CONFIGURATION

- Modify init/config_multi-nodes.yaml to change server's IP/mac/cpu/memory/linux image

- Generate vagrant public key in put in init/scripts
```
ssh-keygen -t rsa -f ~/.ssh/vagrant.id_rsa
cp ~/.ssh/vagrant.id_rsa.pub init/scripts/
```

- Create docker machine using Vagrantfile
```
cd ./init/
vagrant up --provider=virtualbox
```

- Run ansible playbook to setup server
```
# Checkout to test branch, edit the credential and server's IP according to the latest commit
git checkout test_branch_with_cred
git pull --rebase
git log -p

# Setup DNS/Nameserver environment
ansible-playbook -i inventories/develop -e "remote_user=vagrant" playbook/nameserver.yml

# Setup Jenkins Server on develop environment
ansible-playbook -i inventories/develop -e "remote_user=vagrant" playbook/jenkins.yml
```


## Create Jenkins credential
* To find out the REST API of Jenkins to create credential
-> Go to credential, create an example credential, open on browser option View -> Developer -> Developer Tools -> Network

* Example with ssh key credentials (remember to save key in one line separate by \n)
```
KEY='-----BEGIN OPENSSH PRIVATE KEY-----\nblablablablan\nline2line2\n...etc...\n-----END OPENSSH PRIVATE KEY-----'

CRUMB=$(curl -s --cookie-jar /home/vagrant/cookie -u admin:admin 'http://localhost:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)')
url='http://localhost:8080/credentials/store/system/domain/_/createCredentials'
curl -H $CRUMB --cookie /home/vagrant/cookie -u admin:admin -X POST "$url" \
--data-urlencode 'json={
  "": "0",
  "credentials": {
    "scope": "GLOBAL",
    "id": "phamvuong-github",
    "username": "phamvuong",
    "privateKeySource": {
      "privateKey": "'"$KEY"'",
      "$class": "com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey$DirectEntryPrivateKeySource"
    },
    "passphrase": "",
    "$redact": "passphrase",
    "$class": "com.cloudbees.jenkins.plugins.sshcredentials.impl.BasicSSHUserPrivateKey"
  }
}'
```
- Example with using username and password
```
CRUMB=$(curl -s --cookie-jar /home/vagrant/cookie 'http://admin:admin@localhost:8080/crumbIssuer/api/xml?xpath=concat(//crumbRequestField,":",//crumb)')
curl -H $CRUMB --cookie /home/vagrant/cookie -X POST 'http://admin:admin@localhost:8080/credentials/store/system/domain/_/createCredentials' \
--data-urlencode 'json={
  "": "0",
  "credentials": {
    "scope": "GLOBAL",
    "id": "identification",
    "username": "manu",
    "password": "bar",
    "description": "linda",
    "$class": "com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl"
  }
}'
```
- Example with using Python
```
python /var/lib/jenkins/create-credentials-config.py ssh-private-key -I phamvuong -D python-pipeline -U phamvuong -K /var/lib/jenkins/.ssh/phamvuong -u admin -p admin
```
