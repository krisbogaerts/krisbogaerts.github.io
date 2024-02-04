---
layout: post
title: "Nginx Proxy Manager, Open-appsec automatic web application security using machine learning"
date: 2024-01-14
categories: [Open Source, Cybersecurity, Web Application Firewall]
tags: [Security, open-appsec, Nginx Proxy Manager, Machine Learning]
---

![open-appsec - automatic web application & API security using machine learning ](https://static.wixstatic.com/media/bc6682_731d6c9ec2e34225b9a4c92e56218e86~mv2.png/v1/fill/w_740,h_117,al_c,q_85,usm_0.66_1.00_0.01,enc_auto/bc6682_731d6c9ec2e34225b9a4c92e56218e86~mv2.png)


## Intro

The open-appsec team [announced](https://www.openappsec.io/post/announcing-open-appsec-waf-integration-with-nginx-proxy-manager) the beta release of a new integration of open-appsec WAF with NGINX Proxy manager. This will allow NGINX Proxy Manager (NPM) users to protect their web applications and web APIs exposed by NGINX Proxy Manager by easily activating and configuring open-appsec protection for each of the configured Proxy Host objects in NPM directly from the NPM Web UI and also to monitor security events.


![Architecture](https://2130324589-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FNcZmX14M2KdTBrq9EOnI%2Fuploads%2FT4JFMoyMBllGFe8yX4dt%2Farchitecture-npm-docker-large.png?alt=media&token=8e3d5dcd-ae7c-4d19-a5c2-d5ee19c2ef51)

This new integration of open-appsec WAF with NGINX Proxy Manager not only closes the security gap caused by the soon end-of-life ModSecurity WAF, but provides a modern, even stronger protection alternative in form of open-appsec, a preemptive, machine-learning based, fully automatic WAF that does not rely on signatures at all. NGINX Proxy Manager and open-appsec are both open-source solutions.


Open-appsec WAF provides automatic, preemptive threat prevention for reverse proxies like NGINX. It is machine learning based, which means it doesn’t require signatures (or updating them) at all. This enables it to provide threat prevention even for true zero-day attacks while significantly reducing both, administrative effort as well as the amount of false-positives.

Website: https://www.open-appsec.io

GitHub: https://github.com/openappsec

Docs: https://docs.open-appsec.io



## Deployment

Docs: https://docs.openappsec.io/integrations/nginx-proxy-manager-integration

In my case NGINX Proxy Manger(NPM)) was already active. I merged the existing docker compose file with the example provided in the official docs. 

Minor Adjustments against the docker compose file from the docs:
- Running processes as a user/group (in the Open-appsec official docs NPM runs as root)
- Disabling IPv6
- Using the (SQlite) database from my existing NPM setup
- IPC sharing only from the npm-attachment container to the openappsec-agent and not full host shared memory access
- I am using Portainer for management, changed to full paths for the docker volumes

### Docker compose

```yaml
version: '3'
services:
  appsec-npm:
    image: 'ghcr.io/openappsec/nginx-proxy-manager-attachment:latest'
    container_name: npm-attachment

    healthcheck:
      test: ["CMD", "/bin/check-health"]
      interval: 10s
      timeout: 3s
    restart: unless-stopped
    environment:
      PUID: 1000
      PGID: 1000
      DISABLE_IPV6: 'true'
    volumes:
      - /opt/npm-open-appsec/config.json:/app/config/production.json
      - /opt/npm-open-appsec/data:/data
      - /opt/npm-open-appsec/letsencrypt:/etc/letsencrypt
      - /opt/npm-open-appsec/appsec-logs:/ext/appsec-logs
      - /opt/npm-open-appsec/appsec-localconfig:/ext/appsec
    ports:
      - '80:80' 
      - '81:81'
      - '443:443'
    ipc: "shareable"
    
  appsec-agent:
    container_name: openappsec-agent
    network_mode: service:appsec-npm
    image: 'ghcr.io/openappsec/agent:latest'
    restart: unless-stopped
    environment:
      - user_email=kris@bogaerts.org
      - nginxproxymanager=true
      - autoPolicyLoad=true
 
    volumes:
      - /opt/npm-open-appsec/appsec-config:/etc/cp/conf
      - /opt/npm-open-appsec/appsec-data:/etc/cp/data
      - /opt/npm-open-appsec/appsec-logs:/var/log/nano_agent
      - /opt/npm-open-appsec/appsec-localconfig:/ext/appsec
    command: /cp-nano-agent --standalone
    ipc: "container:npm-attachment"

    
```


### Create the following environment variables

```
MYSQL_ROOT_PASSWORD=
MYSQL_USER=
MYSQL_PASSWORD=
OAPPSEC_USER_EMAIL=
```

### Deploy the stack 

Deploy te stack or run ```docker compose up -d```


### Connect

Connect to ```http://[hostname or IP of your host]:81```

Use the following default administrator user credentials and create a new you own admin user and password:

E-mail address:       admin@example.com
Password:                 changeme


![NPM](https://2130324589-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FNcZmX14M2KdTBrq9EOnI%2Fuploads%2FGdXiyvPzs3yBbW3IhH2S%2Fimage.png?alt=media&token=0a6d1d86-8372-47b3-b886-dc6f45da3de2){: w="500" h="100" }


### Configuration


Once you created a new Proxy Host within NGINX Proxy Manager WebUI you can now enable and configure open-appsec protection :
1. Enable open-appsec by enabling “open-appsec” switch.
2. Select the Enforcement Mode, “Prevent-Learn” or “Detect-Learn”
3. Select the minimum confidence level for open-appsec to prevent an attack
4. Click “Save”

![NPM](https://2130324589-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FNcZmX14M2KdTBrq9EOnI%2Fuploads%2FLGWLf247SI8sbVLO7Dvv%2Fimage.png?alt=media&token=5fd70980-3ce9-4c07-9a51-fc71060a922e)


### How to view the logs

Open Appsec added menu option "Security Log":
- Important Events
- All Events
- Notifications

![NPM](https://2130324589-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FNcZmX14M2KdTBrq9EOnI%2Fuploads%2FTC3cJ5E263fHZLXGWGJL%2Fimage.png?alt=media&token=dcf6ddf6-b1a4-4425-b16a-77f027a85302)

### Check Status

Run ```docker exec appsec-agent open-appsec-ctl -s```

```
---- open-appsec Nano Agent ----
Version: 1.1.3-open-source
Status: Running
Management mode: Local management
Policy files:
    /etc/cp/conf/local_policy.yaml
Policy load status: Success
Last policy update: 2023-12-04T11:33:46.083483
AI model version: Simple model V1.0

---- open-appsec Orchestration Nano Service ----
Type: Public, Version: 1.1.3-open-source, Created at: 2023-10-05T17:46:48+0000
Status: Running

---- open-appsec Attachment Registrator Nano Service ----
Type: Public, Version: 1.1.3-open-source, Created at: 2023-10-05T17:46:48+0000
Status: Running

---- open-appsec Http Transaction Handler Nano Service ----
Type: Public, Version: 1.1.3-open-source, Created at: 2023-10-05T17:46:48+0000
Registered Instances: 2
Status: Running
```

## Advanced model

https://docs.openappsec.io/getting-started/using-the-advanced-machine-learning-model

After installing the agent runs the 'Simple model'. An advanced model is available on [https://my.openappsec.io/](https://my.openappsec.io/)
Make sure to create a user account and not use the Github (federation) user. For me the user menu was not available with a Github account.

1. Download the advanced model from the user menu
2. Copy the 'open-appsec-advanced-model.tgz' to /opt/npm-oappsec/advanced-model on the docker host.

Add an extra line for the mapping of the line for the ppsec-agent container to make the file available for the agent

```- /opt/npm-oappsec/advanced-model:/advanced-model```


Full Example:

```yaml
version: '3'
services:
  appsec-npm:
    image: 'ghcr.io/openappsec/nginx-proxy-manager-attachment:latest'
    container_name: npm-attachment

    healthcheck:
      test: ["CMD", "/bin/check-health"]
      interval: 10s
      timeout: 3s
    restart: unless-stopped
    environment:
      PUID: 1000
      PGID: 1000
      DISABLE_IPV6: 'true'
    volumes:
      - /opt/npm-open-appsec/config.json:/app/config/production.json
      - /opt/npm-open-appsec/data:/data
      - /opt/npm-open-appsec/letsencrypt:/etc/letsencrypt
      - /opt/npm-open-appsec/appsec-logs:/ext/appsec-logs
      - /opt/npm-open-appsec/appsec-localconfig:/ext/appsec
    ports:
      - '80:80' 
      - '81:81'
      - '443:443'
    ipc: "shareable"
    
  appsec-agent:
    container_name: openappsec-agent
    network_mode: service:appsec-npm
    image: 'ghcr.io/openappsec/agent:latest'
    restart: unless-stopped
    environment:
      - user_email=kris@bogaerts.org
      - nginxproxymanager=true
      - autoPolicyLoad=true
    volumes:
      - /opt/npm-open-appsec/appsec-config:/etc/cp/conf
      - /opt/npm-open-appsec/appsec-data:/etc/cp/data
      - /opt/npm-open-appsec/appsec-logs:/var/log/nano_agent
      - /opt/npm-open-appsec/appsec-localconfig:/ext/appsec
      - /opt/npm-open-appsec/advanced-model:/advanced-model
    command: /cp-nano-agent --standalone
    ipc: "container:npm-attachment"
```


## Check the agent status

The AI model version should say "Advanced model"

```
user@dockerhost:~# docker exec appsec-agent open-appsec-ctl -s

---- open-appsec Nano Agent ----
Version: 1.1.3-open-source
Status: Running
AI model version: Advanced model V1.0
Management mode: Local management
Policy files: 
    /etc/cp/conf/local_policy.yaml
Policy load status: Success
Last policy update: 2024-01-14T17:50:02.462186

---- open-appsec Orchestration Nano Service ----
Type: Public, Version: 1.1.3-open-source, Created at: 2023-12-27T19:52:40+0000
Status: Running

---- open-appsec Attachment Registrator Nano Service ----
Type: Public, Version: 1.1.3-open-source, Created at: 2023-12-27T19:52:40+0000
Status: Running

---- open-appsec Http Transaction Handler Nano Service ----
Type: Public, Version: 1.1.3-open-source, Created at: 2023-12-27T19:52:40+0000
Registered Instances: 16
Status: Running


For release notes and known limitations check: https://docs.openappsec.io/release-notes
For troubleshooting and support: https://openappsec.io/support
```

## Problems


1. Open-appsec on Docker HTTP Transaction handler has status "ready" and not "Running"
   Make sure to define the 'ipc: host' or make it shareble as in the example above
   https://docs.openappsec.io/troubleshooting/troubleshooting-guides/open-appsec-on-docker-http-tranasction-handler-is-set-to-ready


2. Not able to access  the "Security Logs" from the NPM admin portal
   I change the NPM user from root to 'PUID: 1000 and PGID: 1000". This prevented access to the agent logs 
   Quickfix with 'chown -R 1000:1000 /opt/npm-open-appsec/appsec-logs/' to make the file available for the NPM user (1000)
   Need to do further research to fix this properly

3. Make sure to create a user account and not use the Github (federation) user. For me the advanced model option was not available with a Github account.

## Testing

You can append the following to your http(s) requests to simulate an attack which should be detected/prevented by open-appsec: '?shell_cmd=cat/etc/passwd'
Example: http://localhost/?shell_cmd=cat/etc/passwd



Hope to do some more testing in the coming weeks

In no time everything was migrated and up and running again.
Also make sure to check the docs for more advanced options or for troublshooting

https://docs.openappsec.io/troubleshooting/troubleshooting
https://docs.openappsec.io/troubleshooting/troubleshooting-guides

