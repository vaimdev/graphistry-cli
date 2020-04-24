# Welcome to Graphistry: Admin Guide

Graphistry is the most scalable graph-based visual analysis and investigation automation platform. It supports both cloud and on-prem deployment options. Big graphs are tons of fun!

#### Docs

This documentation cover system administration. See the **Further reading** section at the bottom of this page for the list of top-level administration guides. For analyst and developer guides, see the main docs accessible from the Graphistry web UI: `your_graphistry.acme.ngo/docs`. 

#### Support

Email, Zoom, Slack, phone, tickets -- we encourage using the [Graphistry support channel](https://www.graphistry.com/support) that works best for you. We want you and your users to succeed! 


#### Quick launch: Managed

The fastest way to start using Graphistry is to quick launch a private preconfigured Graphistry instance from the [AWS and Azure marketplaces](https://www.graphistry.com/get-started). They run within your cloud provider account and therefore facilitate experimentation with real data. These docs will still be helpful for advanced configuration and maintenance. See [AWS launch walkthrough tutorial & videos](https://www.graphistry.com/blog/marketplace-tutorial). 

#### Quick launch: Manual

1. Install if not already available from the folder with `containers.tar`, and likely using `sudo`:

```
/var/graphistry $ docker load -i containers.tar
```

2. Launch from the folder with `docker-compose.yml` if not already up, and likely using `sudo`:

```
/var/graphistry $ docker-compose up -d
```

3. Go to `http://localhost`


## Advanced administration

The admin guide covers:
 
* Deployment planning: 
  * Architecture selection: Hosted, cloud marketplaces, cloud, on-prem, hybrid, and air-gapped
  * Capacity planning and software requirements
* Installation & testing
* Configuration
* Operation
* Backup, maintenance, and upgrades


**Cloud Marketplace Admins**: You can focus exclusively on sections on capacity planning, optional configurations, and optional maintenance, . Congrats!

**Manul Install Admins**: The use of GPU computing makes administration a bit different than other software. Graphistry ships batteries-included to make operations close to what you'd expect of modern containerized software. However, this includes setup of Nvidia drivers, Docker, docker-compose, and the nvidia-docker runtime.


## Top commands

Graphistry supports advanced command-line administration via standard `docker-compose`, `.yml` / `.env` files, and `caddy` reverse-proxy configuration.

### Login to server

| Image | Command |
|--: |:-- |
| **AWS** | `ssh -i [your_key].pem ubuntu@[your_public_ip]` |
| **Azure** | `ssh -i [your_key].pem [your_user]@[your_public_ip]` <br> `ssh [your_user]@[your_public_ip]` (pwd-based) |
| **On-prem / BYOL** | Contact your admin |

### CLI commands

All likely require `sudo`. Run from where your `docker-compose.yml` file is located:  `/home/ubuntu/graphistry` (AWS), `/var/graphistry` (Azure), or `/var/graphistry/<releases>/<version>` (recommended on-prem).

|  TASK	| COMMAND 	| NOTES 	|
|--: |:---	|:---	|
| **Install** 	| `docker load -i containers.tar` 	| Install the `containers.tar` Graphistry release from the current folder. You may need to first run `tar -xvvf my-graphistry-release.tar.gz`.	|
| **Start <br>interactive** 	| `docker-compose up` 	| Starts Graphistry, close with ctrl-c 	|
| **Start <br>daemon** 	| `docker-compose up -d` 	| Starts Graphistry as background process 	|
| **Start <br>namespaced** (experimental) 	| `docker-compose -p my_namespace up` 	| Starts Graphistry with a unique name (in case of multiple versions). NOTE: must modify volume names in `docker-compose.yml`. 	|
| **Stop** 	| `docker-compose stop` 	| Stops Graphistry 	|
| **Restart** 	| `docker restart <CONTAINER>` 	|  	|
|  **Status** 	| `docker-compose ps`, `docker ps`, and `docker status` 	|  Status: Uptime, healthchecks, ...	|
| **GPU Status** | `nvidia-smi` | See GPU processes, compute/memory consumption, and driver.  Ex: `watch -n 1.5 nvidia-smi` |
|  **API Key** 	| docker-compose exec streamgl-vgraph-etl curl "http://0.0.0.0:8080/api/internal/provision?text=MYUSERNAME" 	|  Generates API key for a developer or notebook user	|
| **Logs** 	|  `docker-compose logs <CONTAINER>` 	|  Ex: Watch all logs, starting with the 20 most recent lines:  `docker-compose logs -f -t --tail=20 forge-etl-python`	. You likely need to switch Docker to use the local json logging driver by  deleting the two default managed Splunk log driver options in `/etc/docker/daemon.json` and then restarting the `docker` daemon (see below). |
| **Reset**     | `docker-compose down -v && docker-compose up` | Stop Graphistry, remove all internal state (including the user account dataabase!), and start fresh .  |
| **Create Users** | Use Admin Panel (see [Create Users](docs/user-creation.md)) |
| **Restart Docker Daemon** | `sudo service docker restart` | Use when changing `/etc/docker/daemon.json`, ... |


## Manual enterprise install

NOTE: Managed Graphistry instances do not require any of these steps.

The Graphistry environnment depends soley on [Nvidia RAPIDS](https://rapids.ai) and [Nvidia Docker](https://github.com/NVIDIA/nvidia-docker) via `Docker Compose 3`, and ships with all other dependencies built in.

### Test your environment setup


You can test your GPU environment via Graphistry's [base RAPIDS Docker image on DockerHub](https://hub.docker.com/r/graphistry/graphistry-blazing):

```
 sudo docker run --rm -it graphistry/graphistry-blazing:v2.29.2 /bin/bash -c "source activate rapids && python3 -c \"import cudf; print(cudf.DataFrame({'x': [0,1,2]})['x'].sum())\""
```
=>
```
3
```

### Manual environment setup

See sample [Ubuntu 18.04 TLS](./docs/ubuntu_18_04_lts_setup.md) and [RHEL 7.6](./docs/rhel_7_6_setup.md) environment setup scripts for Nvidia drivers, Docker, nvidia-docker runtime, and docker-compose. 

Please contact Graphistry staff for environment automation options. 

### Manual Graphistry container download

Download the latest enterprise distribution from the [enterprise release list](https://graphistry.zendesk.com/hc/en-us/articles/360033184174-Enterprise-Releases) on the support site.  Please contact your Graphistry support staff for access if not available.

### Manual Graphistry container installation

If `nvidia` is already your `docker info | grep Default` runtime:

```
############ Install & Launch
wget -O release.tar.gz "https://..."
tar -xvvf release.tar.gz
docker load -i containers.tar
docker-compose up -d
```

## Further reading

* Plan a deployment
  * Architecture: [Deployment architecture planning guide](docs/deployment-planning.md)
  * Capacity: [Hardware/software requirements](hardware-software.md)
  * Security: [Recommended hardening](docs/configure-security.md) and [threat model](docs/threatmodel.md)
* Install
  * [Manual Graphistry Installation](docs/manual.md) for non-marketplace / non-hosted: 
  <br>AWS BYOL, Azure BYOL, On-Prem (RHEL/Ubuntu), & Air-Gapped
* Configure
  * [System settings](docs/configure.md): 
    <br/>TLS/SSL/HTTPS, performance, logging, backups to disk, multiple proxy layers, and more
      * [Security: Enable auto-TLS and restrict network access](docs/configure-security.md)
      * [Add users](docs/user-creation.md)
      * [AWS Marketplace quickstart](docs/aws_marketplace.md)
      * [Azure Marketplace quickstart](docs/azure_marketplace.md)
  * Content: 
      * [Investigations connectors](docs/configure-investigation.md): Splunk, Neo4j, and more
      * [Investigation templates](docs/templates.md): Save, reuse, link, and embed workflows
      * [Custom pivots](docs/configure-custom-pivots.md): Streamline common investigation steps with simplified UIs
      * [Ontology](docs/configure-ontology.md): Add new types and customize colors, icons, sizes, and more 
      * The [Graphistry Data Bridge](docs/bridge.md): Go between cloud <> on-prem
  * [PyGraphistry and notebooks](docs/configure-pygraphistry.md)

* Maintain
  * On unconfigured instance reboots, you may need to first run `sudo systemctl start docker`
  * [Update, backup, and migrate](docs/update-backup-migrate.md)
* Test:
  * [Testing an install](docs/testing-an-install.md)
