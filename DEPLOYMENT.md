# Deployment Evidence

Fill this in as you go. Paste real output, not descriptions of output. A TA reads this
file with you at recitation.

## 1. Deployed URL and instance id

<!-- The ServiceUrl and InstanceId outputs. Paste both here every time
describe-stacks prints them, for the healthy deploy and for scenario 2. Both
change on every recreate, and you will need them for curls and sessions. -->
### Scenario 1
InstanceId: i-09fd61a3e8fb1be42

ServiceUrl: http://ec2-13-218-135-192.compute-1.amazonaws.com:808

### Scenario 2: broken one

InstanceId: i-014e68a181f03054d

ServiceUrl: http://ec2-3-95-231-83.compute-1.amazonaws.com:8080

### Scenario 2: fixed one

InstanceId: i-060b629d0cfe999d0

ServiceUrl: http://ec2-54-166-150-89.compute-1.amazonaws.com:8080

## 2. External health check

Run the check from your own machine, not from the instance. Paste the command and the
response.

```
yuanzhehuang@Sad-MacBook-Pro f26-lab04 % curl http://ec2-13-218-135-192.compute-1.amazonaws.com:8080/api/health
{"status":"ok"}% 
```

## 3. What the template created

Three or four sentences, your own words. What compute, what network access, and what
glue made the service start.

<!-- Your answer here. -->
The service runs on a t3.micro EC2 instance using Amazon Linux 2023. A security group allows inbound traffic on port 8080 and outbound access so the instance can install Docker and pull the container image. The UserData script installs and starts Docker, then runs the service container with the correct port mapping and environment variable. CloudFormation ties these pieces together by creating the EC2 instance, attaching the security group, and starting the container automatically.

## 4. Scenario 2 diagnosis

**The failing curl** (command and output):

```
yuanzhehuang@Sad-MacBook-Pro f26-lab04 % curl http://ec2-3-95-231-83.compute-1.amazonaws.com:8080/api/health   
curl: (28) Failed to connect to ec2-3-95-231-83.compute-1.amazonaws.com port 8080 after 75026 ms: Couldn't connect to server
```

**The log line that told you what was wrong:**

```
sh-5.2$ sudo docker ps
CONTAINER ID   IMAGE                                     COMMAND                  CREATED         STATUS         PORTS                                       NAMES
a1ddf2b9ae6f   ghcr.io/cmu-17-214/lab04-service:latest   "/__cacert_entrypoin…"   7 minutes ago   Up 7 minutes   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   lab04-service

sh-5.2$ sudo docker logs lab04-service
lab04-service listening on 9090
```

**What was wrong, and the fix you applied:**

<!-- One or two sentences. Say what you changed and where you changed it. -->
In template.yaml, PortOverride was set to 9090, causing the Java service to listen on 9090 while traffic was routed to container port 8080. I fixed this by setting PortOverride to 8080 or leaving it empty to use the default ServicePort.

**The healthy curl after the fix:**

```
yuanzhehuang@Sad-MacBook-Pro f26-lab04 % curl http://ec2-54-166-150-89.compute-1.amazonaws.com:8080/api/health
{"status":"ok"}% 
```

## 5. Teardown proof

Paste the delete output, or describe the console evidence that the resources are gone.

```
yuanzhehuang@Sad-MacBook-Pro f26-lab04 % aws cloudformation describe-stacks --stack-name lab04-service

aws: [ERROR]: An error occurred (ValidationError) when calling the DescribeStacks operation: Stack with id lab04-service does not exist
```
