# NuxBill Deployment

## Objective

Deploy NuxBill as a self-hosted ISP billing application on an Ubuntu Linux server using Docker Compose, with MySQL for database storage and Nginx as the reverse proxy.

## Environment

* Ubuntu Linux
* Docker
* Docker Compose
* NuxBill
* PHP/Apache
* MySQL 8.0
* Nginx
* UFW
* SSH
* systemd

## Architecture
See ../architecture.png

## Docker Deployment

NuxBill and MySQL run as separate Docker containers.

The NuxBill container provides the web application, while MySQL provides the database service.

The application communicates with MySQL through the Docker network using the MySQL container's service name rather than exposing MySQL to the host network.

The NuxBill web service is bound to the host through: 127.0.0.1:8080

This prevents direct external access to the application port.

## Database

MySQL is deployed as a separate container with persistent storage.

The database uses:

* Database: 'nuxbill'
* Database user: 'nuxbill'
* MySQL 8.0

The MySQL container is not directly published to the host.

NuxBill connects to MySQL through the internal Docker network.

## Nginx Reverse Proxy

Nginx acts as the public-facing web server and reverse proxy.

Instead of exposing the application container directly, external HTTP/HTTPS traffic is handled by Nginx and forwarded to the local NuxBill service.

This provides a single controlled entry point for web traffic and allows TLS/HTTPS configuration to remain at the reverse-proxy layer.

## Firewall

UFW is used to restrict inbound traffic.

Required public ports:

| Port | Purpose            |
| ---- | ------------------ |
| 22   | SSH administration |
| 80   | HTTP               |
| 443  | HTTPS              |

The application port '8080' and MySQL port '3306' are not exposed publicly.

## SSH

SSH is used for server administration.

SSH access is restricted through the firewall and configured for key-based authentication.

## Configuration and Secrets

Deployment-specific credentials are kept outside version-controlled source code.

The repository provides '.env.example' as a configuration template.

The actual '.env' file is excluded through '.gitignore'.

NuxBill's generated 'config.php' is also excluded from version control because it contains database connection configuration.

## Persistence

MySQL data is stored in a Docker persistent volume.

This separates database data from the lifecycle of the MySQL container itself.

Removing and recreating the container therefore does not inherently remove the database contents stored in the persistent volume.

## Service Management

The deployment uses Docker Compose to define and manage the application and database services.

A systemd timer is also used for scheduled database backups.

## Verification

The deployment was verified by:

* Starting the Docker services successfully
* Confirming the containers were running
* Accessing NuxBill through the web interface
* Confirming NuxBill could communicate with MySQL
* Confirming MySQL data persisted
* Testing database backup and restoration
* Verifying the restored database contained the expected NuxBill tables
* Verifying the scheduled backup timer
