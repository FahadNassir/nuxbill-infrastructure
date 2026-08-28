# NuxBill ISP Billing — Self-Hosted Deployment

## Overview

A self-hosted deployment of **NuxBill**, an ISP billing and network management application, on an Ubuntu Linux server.

The project focuses on the infrastructure and deployment side of the application: containerization, reverse proxying, networking, database persistence, firewall configuration, backups, and operational troubleshooting.

This is a **self-directed portfolio project** designed to demonstrate practical Linux server and application deployment skills.

## Architecture

The deployment follows this traffic and service flow:

**Internet → UFW → Nginx → Docker/NuxBill → MySQL → Persistent Storage**

Administrative access uses a separate SSH path:

**Administrator → SSH :22 → UFW → SSH service**

NuxBill is exposed to the host only through `127.0.0.1:8080`. Nginx acts as the public-facing reverse proxy, while MySQL remains accessible only within the Docker network.

See [`architecture.png`](architecture/architecture.png) for the deployment architecture.

## Infrastructure

* Ubuntu Linux
* Nginx
* Docker
* Docker Compose
* NuxBill
* Apache/PHP within the NuxBill container
* MySQL 8.0
* UFW
* systemd
* SSH
* Docker persistent volumes

## Implemented

* Deployed NuxBill using Docker Compose
* Configured MySQL as a separate container
* Created persistent MySQL storage using a Docker volume
* Configured Nginx as a reverse proxy
* Restricted NuxBill's host binding to `127.0.0.1:8080`
* Configured UFW for required inbound services
* Disabled SSH password authentication
* Enabled SSH public-key authentication
* Configured automatic container/service recovery where appropriate
* Created MySQL database backups
* Tested database backup restoration
* Configured a systemd timer for automated backups
* Investigated application, Nginx, Docker, MySQL, networking, and service logs during deployment
* Documented the deployment and operational procedures

## Network Exposure

Only the following inbound ports are permitted through UFW:

| Port | Purpose            |
| ---- | ------------------ |
| 22   | SSH administration |
| 80   | HTTP               |
| 443  | HTTPS              |

The application container itself is not directly exposed to the network. Nginx communicates with it through the local host binding.

MySQL is not published to the host and communicates with NuxBill through the Docker network.

## Backup and Recovery

The MySQL database is stored in a persistent Docker volume and backed up using `mysqldump`.

Automated backups are triggered by a systemd timer and stored under:

`/opt/ispb/backups/`

A separate database was used to verify that a generated SQL backup could be restored successfully.

See [`backup-restore.md`](docs/backup-restore.md) for the procedure.

## Troubleshooting

The deployment included investigation of several infrastructure layers:

* HTTP request and response behavior
* Nginx access and error logs
* Docker container status and configuration
* Docker networking
* MySQL container logs
* systemd services and timers
* UFW rules
* SSH configuration
* Database permissions
* Backup and restoration

The troubleshooting documentation records the relevant symptoms, investigation process, and resolution rather than presenting the deployment as a simple installation.
## Project Scope

The project intentionally focuses on **infrastructure and deployment rather than application development**.

The NuxBill application itself was treated as the application payload. The work concentrated on deploying, exposing, securing, operating, backing up, and troubleshooting the application environment.

## Limitations

This is a self-directed lab/portfolio deployment rather than a production deployment for a real ISP.

The project demonstrates the infrastructure concepts and operational procedures used during deployment, but it does not claim to reproduce the scale, redundancy, monitoring, compliance, or operational processes of a production ISP environment.

## Skills Demonstrated

**Primary**

* Linux server administration
* Docker and Docker Compose
* Nginx reverse proxy configuration
* Networking fundamentals
* Database deployment and persistence
* Firewall configuration
* Backup and restoration
* Service management with systemd
* Infrastructure troubleshooting

**Supporting**

* Git
* SSH
* HTTP/HTTPS
* MySQL
* Log analysis
* Configuration management
* Technical documentation
