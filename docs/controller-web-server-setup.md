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

## Important future step: deploy to another server

If the website is moved to another VM, for example:

```text
Jenkins Controller: 192.168.50.128
Web Server:         192.168.50.131
```

The controller's `/var/www/html` ownership no longer controls the remote server.

A later deployment design can be:

```text
GitHub
   |
   v
Webhook -> Jenkins Controller (.128)
              |
              | SSH / SCP
              v
        Web Server (.131)
              |
              v
        /var/www/html
              |
              v
            Nginx
```

In that setup, Jenkins needs SSH access to `.131`, and the remote web directory must be writable by the account used for deployment (or deployment must use a controlled privilege-escalation method).

Do not introduce the remote server for the first webhook demo. Keep Jenkins + Nginx on `.128` first, then move execution/deployment to separate nodes in later videos.
