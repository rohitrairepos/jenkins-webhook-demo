# Jenkins Controller Web Server Setup

These steps are for the first Jenkins webhook demo where Jenkins and Nginx run on the same controller VM.

## Controller

```text
Jenkins Controller: 192.168.50.128
```

## Install Nginx

```bash
sudo apt update
sudo apt install nginx -y
```

Check the service:

```bash
sudo systemctl status nginx
```

Test locally:

```bash
curl http://localhost
```

## Give Jenkins ownership of the Nginx web directory

For this controller-only lab demo, Jenkins can own the website directory so the pipeline can deploy files directly.

```bash
sudo chown -R jenkins:jenkins /var/www/html
sudo chmod -R 755 /var/www/html
```

Verify:

```bash
ls -ld /var/www/html
ls -l /var/www/html
```

Expected ownership:

```text
jenkins jenkins
```

## Why this is needed

Jenkins pipeline steps run as the Linux `jenkins` user by default when the job executes on the built-in/controller node.

Check the execution user from a pipeline with:

```groovy
echo "Running as user: $(whoami)"
```

Expected output:

```text
Running as user: jenkins
```

## Current deployment flow

```text
GitHub
   |
   | push
   v
GitHub Webhook
   |
   v
Cloudflare Tunnel
   |
   v
Jenkins Controller (192.168.50.128)
   |
   | Jenkins Pipeline
   v
/var/www/html
   |
   v
Nginx
   |
   v
Website
```

## Future Jenkins Agent story

This is where the Jenkins Agent story becomes interesting.

### Stage 1: Everything on the Controller

For the first demo, Jenkins and Nginx run on the same controller:

```text
GitHub
   |
   v
Webhook
   |
   v
Jenkins Controller (.128)
   |
   v
Nginx Website (.128)
```

The pipeline runs on the built-in/controller node as the `jenkins` Linux user.

### Stage 2: Separate Web Server

If the website is moved to another VM, for example `.131`, Jenkins can deploy to it over SSH/SCP:

```text
GitHub
   |
   v
Webhook
   |
   v
Jenkins Controller (.128)
   |
   | SSH / SCP
   v
Web Server (.131)
   |
   v
Nginx
```

Jenkins then needs SSH access to `.131`, and the remote web directory must be writable by the deployment account or use a controlled privilege-escalation method.

### Stage 3: Static Jenkins Agent

Next, move pipeline execution from the controller to a dedicated static agent, for example `.129`:

```text
GitHub
   |
   v
Webhook
   |
   v
Jenkins Controller (.128)
   |
   | SSH
   v
Static Jenkins Agent (.129)
   |
   | deployment
   v
Web Server (.131)
```

The important concept is that the Jenkins controller receives the webhook and coordinates the job, while the agent executes the pipeline workload.

For example:

```text
Controller:
whoami -> jenkins

Static Agent:
whoami -> rohit
```

### Stage 4: Docker Plugin

After understanding static agents, introduce dynamic execution with the Jenkins Docker Plugin:

```text
GitHub
   |
   v
Webhook
   |
   v
Jenkins Controller (.128)
   |
   | Docker Plugin
   v
Docker Host (.130)
   |
   v
Temporary Jenkins Agent Container
   |
   v
Pipeline execution
```

This progression demonstrates why Jenkins agents are useful instead of immediately introducing them before viewers understand the basic webhook-driven deployment.

## Important learning sequence

```text
1. Jenkins Pipeline on Controller
2. GitHub Webhook triggers Jenkins
3. Jenkins deploys website on Controller
4. Separate Web Server
5. Static SSH Agent
6. Docker Agent / Docker Plugin
```

Do not introduce the remote server or agents for the first webhook demo. Keep Jenkins + Nginx on `.128` first, then evolve the architecture in later videos.
