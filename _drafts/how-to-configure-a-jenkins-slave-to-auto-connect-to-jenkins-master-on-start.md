---
layout: post
title: How to configure a Jenkins slave to auto connect to Jenkins master on start
date: 2016-08-04 22:33:29.000000000 -05:00
---

## Context
At [PetalMD](https://petalmd.com) we build cloud applications for Healthcare professionals. We have a backend running Ruby on Rails (API / admin app) and front-end apps in pure Javascript. All applications have unit/end-to-end tests and can be deployed as needed, typically several times a day.

The purpose of the article is to give a quick overview of how 

## Requirements
- When Jenkins doesn't have enough workers for running pending jobs, it need to increase the number of slaves.
- When it has more slave than running + pending jobs, it needs to remove sleeping slaves

## Implementation
### Jenkins configuration

### EC2 AMI configuration
#### Add base dependencies: java jre, git, docker and docker compose
```
apt-get install -y docker-engine git openjdk-7-jre
systemctl enable docker
curl -L https://github.com/docker/compose/releases/download/1.6.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

#### Add a script that will autoconnect slave to jenkins master
Create a new file `/usr/bin/userdata` with this content:

```
#!/usr/bin/python
import os
import httplib
import string

conn = httplib.HTTPConnection("169.254.169.254")
conn.request("GET", "/latest/user-data")
response = conn.getresponse()
userdata = response.read()

args = string.split(userdata, "&")
jenkinsUrl = ""
slaveName = ""

for arg in args:
    if arg.split("=")[0] == "JENKINS_URL":
        jenkinsUrl = arg.split("=")[1]
    if arg.split("=")[0] == "SLAVE_NAME":
        slaveName = arg.split("=")[1]

os.system("wget --http-user=JENKINS_USER --http-password=JENKINS_USER_TOKEN " + jenkinsUrl + "jnlpJars/slave.jar -O slave.jar")
os.system("java -jar slave.jar -jnlpCredentials JENKINS_USER: JENKINS_USER_TOKEN -jnlpUrl " + jenkinsUrl + "computer/" + slaveName + "/slave-agent.jnlp")
```
Don't forget to replace `JENKINS_USER ` and `JENKINS_USER_TOKEN ` with your jenkins user and token.

#### Load previous script on boot
Add a new line at the end of `rc.local` with:

```
python /usr/bin/userdata
```