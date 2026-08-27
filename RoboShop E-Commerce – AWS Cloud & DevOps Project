# 🛒 RoboShop E-Commerce – AWS Cloud & DevOps Project

## 📌 Project Overview

RoboShop is a microservices-based e-commerce application deployed on **AWS EC2** using Linux administration, DNS, systemd, Nginx, databases, Redis, RabbitMQ, and multiple application runtimes.

The project demonstrates how to deploy, configure, connect, troubleshoot, and validate a complete microservices application in an AWS environment.

### Technologies Used

* AWS EC2
* AWS VPC
* Amazon Route 53
* Internet Gateway
* Linux / RHEL
* Nginx
* Node.js
* Python
* Java / Spring Boot
* Go
* MongoDB
* MySQL
* Redis
* RabbitMQ
* systemd
* Git / GitHub
* curl
* DNS
* REST APIs

---

# 🏗️ Architecture

```text
                         INTERNET
                            │
                            ▼
                    Amazon Route 53
                  roboshop.shashi5devops.fun
                            │
                            ▼
                     Internet Gateway
                            │
                            ▼
                    ┌───────────────┐
                    │ Frontend EC2  │
                    │ Nginx :80     │
                    └───────┬───────┘
                            │
             ┌──────────────┼──────────────┐
             │              │              │
             ▼              ▼              ▼
       Catalogue          User           Cart
       Node.js            Node.js        Node.js
       :8080              :8080          :8082
             │              │              │
             └──────────────┼──────────────┘
                            │
             ┌──────────────┼──────────────┐
             ▼              ▼              ▼
          MongoDB         Redis          Shipping
          :27017          :6379          Java
                                           :8081
                                             │
                                             ▼
                                           MySQL
                                           :3306

                     Payment
                     Python
                     :8081
                        │
                        ▼
                    RabbitMQ
                     :5672
                        │
                        ▼
                    Dispatch
                       Go
                    Consumer
```

> **Note:** Because of the AWS EC2 quota limitation, multiple application services were configured to run on different ports where required instead of assuming one EC2 instance per service.

---

# ☁️ AWS Infrastructure

## EC2

EC2 instances were used to host the RoboShop microservices and supporting components.

During deployment, the AWS account had an EC2 quota limitation. The available quota was initially insufficient for deploying every microservice on a separate instance.

Therefore, services were consolidated where necessary.

For example:

```text
Catalogue → 8080
User      → 8081
Cart      → 8082
```

This avoided port conflicts while working within the EC2 quota.

---

# 🌐 DNS Configuration

Private service communication was configured using DNS names such as:

```text
cart.shashi5devops.fun
catalogue.shashi5devops.fun
user.shashi5devops.fun
rabbitmq.shashi5devops.fun
redis.shashi5devops.fun
mongodb.shashi5devops.fun
mysql.shashi5devops.fun
```

DNS resolution was tested using:

```bash
getent hosts cart.shashi5devops.fun
```

Example result:

```text
172.31.15.67    cart.shashi5devops.fun
```

This confirmed that the hostname resolved to the expected private IP.

---

# 🐧 Linux Administration

The application servers were managed using Linux.

Important commands used throughout the project:

```bash
systemctl status <service>
systemctl start <service>
systemctl stop <service>
systemctl restart <service>
systemctl enable <service>
systemctl daemon-reload
systemctl cat <service>
systemctl show <service> -p Environment
journalctl -u <service>
journalctl -u <service> -n 50 -o cat
ps -ef
grep
grep -Rni
find
ls -lh
curl
getent hosts
vim
```

---

# ⚙️ systemd Service Management

Instead of manually running applications, systemd was used to manage services.

Advantages:

* Automatic startup
* Process management
* Centralized logs
* Service restart
* Environment variable configuration
* Integration with Linux boot process

---

# 🛒 Cart Service

The Cart service is a Node.js application.

Because another service was already using port `8080`, the Cart service was configured to use:

```text
8082
```

The systemd configuration contained:

```ini
[Unit]
Description=Cart Service

[Service]
User=roboshop
Environment=REDIS_HOST=redis.shashi5devops.fun
Environment=CATALOGUE_HOST=catalogue.shashi5devops.fun
Environment=CATALOGUE_PORT=8080
Environment=CART_SERVER_PORT=8082
ExecStart=/bin/node /app/cart-server.js
SyslogIdentifier=cart

[Install]
WantedBy=multi-user.target
```

After changing the configuration:

```bash
systemctl daemon-reload
systemctl restart cart
systemctl status cart --no-pager
```

---

# 🔍 Cart API Troubleshooting

Initially, the following request was tested:

```bash
curl -i -X POST http://cart.shashi5devops.fun:8080/shipping/anonymous-43 \
-H "Content-Type: application/json" \
-d '{"distance":9201,"cost":460.05,"location":"Australia"}'
```

It returned:

```text
HTTP/1.1 404 Not Found

Cannot POST /shipping/anonymous-43
```

The reason was simple:

**Cart was not running on port 8080.**

The actual Cart port was `8082`.

The correct test was:

```bash
curl -i -X POST http://localhost:8082/shipping/anonymous-43 \
-H "Content-Type: application/json" \
-d '{"distance":9201,"cost":460.05,"location":"Australia"}'
```

Result:

```text
HTTP/1.1 200 OK
```

Response:

```json
{
  "total": 2461.05,
  "tax": 410.1749999999997,
  "items": [
    {
      "qty": 1,
      "sku": "Watson",
      "name": "Watson",
      "price": 2001,
      "subtotal": 2001
    },
    {
      "qty": 1,
      "sku": "SHIP",
      "name": "shipping to Australia",
      "price": 460.05,
      "subtotal": 460.05
    }
  ]
}
```

This proved that the Cart shipping API was working.

---

# ❤️ Cart Health Check

Cart was also tested remotely:

```bash
curl -i http://cart.shashi5devops.fun:8082/health
```

Successful response:

```json
{
  "app": "OK",
  "redis": true
}
```

This confirmed:

```text
Cart Application → Working
Redis Connection  → Working
DNS               → Working
Network           → Working
Port 8082         → Working
```

---

# 🚚 Shipping Service

The Shipping service is a Java Spring Boot application.

The application was originally built into:

```text
/app/target/shipping-1.0.jar
```

The generated JAR was checked using:

```bash
ls -lh /app/target/*.jar
```

Result:

```text
shipping-1.0.jar
```

The deployed JAR was:

```text
/app/shipping.jar
```

A backup was created before replacement:

```bash
cp /app/shipping.jar /app/shipping.jar.bak
```

The service was configured using systemd.

---

# 🔧 Shipping systemd Configuration

```bash
systemctl cat shipping
```

Configuration:

```ini
[Unit]
Description=Shipping Service

[Service]
User=roboshop
Environment=CART_ENDPOINT=cart.shashi5devops.fun:8080
Environment=DB_HOST=mysql.shashi5devops.fun
ExecStart=/bin/java -jar /app/shipping.jar
SyslogIdentifier=shipping

[Install]
WantedBy=multi-user.target
```

The application was successfully started:

```bash
systemctl daemon-reload
systemctl restart shipping
systemctl status shipping --no-pager
```

---

# ❤️ Shipping Health Check

The Shipping service was listening on port `8081`.

Test:

```bash
curl -i http://localhost:8081/health
```

Successful response:

```text
HTTP/1.1 200 OK

OK
```

This confirmed that the Java application was running.

---

# 🧩 Shipping Controller Issue

Initially, the Shipping application expected a route involving `/shipping`.

The controller source was inspected:

```bash
vim /app/src/main/java/com/instana/robotshop/shipping/Controller.java
```

The route was changed from:

```text
shipping
```

to:

```text
order
```

The application was then rebuilt.

The important lesson was:

> Changing Java source code does not change the running application until the application is rebuilt and the new JAR is deployed.

---

# 💳 Payment Service

Payment is implemented using Python and uWSGI.

The application files included:

```text
payment.py
payment.ini
requirements.txt
rabbitmq.py
```

The Payment service was managed using systemd.

Check service:

```bash
systemctl cat payment
```

Environment variables:

```ini
Environment=CART_HOST=cart.shashi5devops.fun
Environment=CART_PORT=8082
Environment=USER_HOST=user.shashi5devops.fun
Environment=USER_PORT=8080
Environment=AMQP_HOST=rabbitmq.shashi5devops.fun
Environment=AMQP_USER=roboshop
Environment=AMQP_PASS=roboshop123
```

---

# 🔎 Verifying Payment Environment Variables

The active environment variables were verified using:

```bash
systemctl show payment -p Environment
```

The important configuration was:

```text
CART_HOST=cart.shashi5devops.fun
CART_PORT=8082
```

This was critical because Cart was running on port `8082`.

---

# 🐛 Payment → Cart 404 Issue

Initially, Payment was configured to contact Cart on:

```text
CART_PORT=8080
```

But Cart was actually listening on:

```text
8082
```

Payment logs showed:

```text
cart delete returned 404
```

The Payment source was inspected:

```bash
grep -nEi "cart|delete|CART" /app/payment.py
```

The relevant code showed:

```python
CART = os.getenv('CART_HOST', 'cart')
CART_PORT = os.getenv('CART_PORT', 8080)
```

And the delete request:

```python
requests.delete(
    'http://{cart}:{cartPort}/cart/{id}'
)
```

Therefore, the correct solution was to configure:

```text
CART_PORT=8082
```

After updating the systemd configuration:

```bash
systemctl daemon-reload
systemctl restart payment
```

Then verify:

```bash
systemctl show payment -p Environment
```

---

# 🐇 RabbitMQ

RabbitMQ was used as the message broker between Payment and Dispatch.

Payment successfully connected to RabbitMQ.

From the Payment logs:

```text
queue order
connected to broker
message sent
```

This proved that the order message was successfully published.

---

# 🚀 Order Flow

The final order flow worked like this:

```text
User
 │
 ▼
Frontend
 │
 ▼
Cart
 │
 ├── Catalogue
 ├── Redis
 │
 ▼
Shipping
 │
 ▼
Payment
 │
 ▼
RabbitMQ
 │
 ▼
Dispatch
```

---

# 📦 Successful Order

After fixing the Cart port configuration, the order was successfully placed.

The application displayed:

```text
Order placed 171d4f8a-6ace-482e-8243-6b48f8a84a3a

Thank you for your order
```

This confirmed that the complete order-processing flow was functioning.

---

# 🚚 Dispatch Service

Dispatch is written in Go.

Go was installed using:

```bash
dnf install golang -y
```

The application user was created:

```bash
useradd --system --home /app --shell /sbin/nologin \
--comment "roboshop system user" roboshop
```

Application directory:

```bash
mkdir /app
```

---

# 📥 Dispatch Application

The Dispatch source was downloaded and extracted:

```bash
curl -L -o /tmp/dispatch.zip \
https://roboshop-artifacts.s3.amazonaws.com/dispatch-v3.zip

cd /app

unzip /tmp/dispatch.zip
```

Dependencies were downloaded:

```bash
go mod init dispatch
go get
```

The application was compiled:

```bash
go build
```

This generated:

```text
/app/dispatch
```

---

# ❌ Dispatch 203/EXEC Error

Initially, systemd returned:

```text
status=203/EXEC
```

with:

```text
Failed to locate executable /app/dispatch
```

The problem was that the binary did not exist yet.

After building the Go application:

```bash
go build
```

the executable became available:

```text
/app/dispatch
```

---

# ⚙️ Dispatch systemd Service

The service was configured as:

```ini
[Unit]
Description=Dispatch Service

[Service]
User=roboshop
Environment=AMQP_HOST=rabbitmq.shashi5devops.fun
Environment=AMQP_USER=roboshop
Environment=AMQP_PASS=roboshop123
ExecStart=/app/dispatch
SyslogIdentifier=dispatch

[Install]
WantedBy=multi-user.target
```

Then:

```bash
systemctl daemon-reload
systemctl enable dispatch
systemctl start dispatch
```

Check:

```bash
systemctl status dispatch --no-pager
```

---

# 🐇 Dispatch RabbitMQ Connection Issue

When the binary was manually executed initially:

```bash
/app/dispatch
```

the application attempted:

```text
amqp://:@rabbitmq:5672/
```

and failed:

```text
dial tcp: lookup rabbitmq ... no such host
```

The problem was:

```text
rabbitmq
```

was not resolvable.

The actual DNS hostname was:

```text
rabbitmq.shashi5devops.fun
```

Therefore, systemd environment variables were configured:

```ini
Environment=AMQP_HOST=rabbitmq.shashi5devops.fun
Environment=AMQP_USER=roboshop
Environment=AMQP_PASS=roboshop123
```

After starting through systemd, the logs showed:

```text
Connecting to amqp://roboshop:roboshop123@rabbitmq.shashi5devops.fun:5672/
Waiting for messages
Rabbit MQ ready true
```

This confirmed:

```text
Dispatch → RabbitMQ = Connected
```

---

# 📊 Dispatch Verification

Logs were checked using:

```bash
journalctl -u dispatch -n 30 -o cat
```

Successful output included:

```text
Rabbit MQ ready true
```

The process was therefore running correctly as a systemd-managed service.

---

# 🔍 Troubleshooting Methodology

One of the most important parts of this project was troubleshooting.

Instead of randomly changing configuration, the following process was used.

## 1. Check service status

```bash
systemctl status <service> --no-pager
```

## 2. Check logs

```bash
journalctl -u <service> -n 50 -o cat
```

## 3. Check configuration

```bash
systemctl cat <service>
```

## 4. Check active environment variables

```bash
systemctl show <service> -p Environment
```

## 5. Check application source

```bash
grep -RniE "shipping|app.post|router.post|POST" /app
```

## 6. Check files

```bash
ls -lh /app
find /app -maxdepth 2 -type f
```

## 7. Check DNS

```bash
getent hosts cart.shashi5devops.fun
```

## 8. Test APIs directly

```bash
curl -i http://localhost:<PORT>/health
```

## 9. Test microservice communication

```bash
curl -i -X POST http://localhost:<PORT>/shipping/<id>
```

This methodology helped identify:

* Wrong port
* Wrong endpoint
* Missing executable
* Wrong RabbitMQ hostname
* Incorrect environment variable
* Application routing problems

---

# 🧠 Major Problems Solved

## Problem 1 – AWS EC2 Quota

The AWS account had a limited EC2 quota.

### Solution

Instead of requiring one EC2 instance for every service, services were consolidated and configured to use different ports.

---

## Problem 2 – Cart Port Conflict

Cart initially attempted to use:

```text
8080
```

But another service was already using that port.

### Solution

Cart was moved to:

```text
8082
```

Environment:

```ini
Environment=CART_SERVER_PORT=8082
```

---

## Problem 3 – 404 on Shipping API

Incorrect request:

```bash
curl -i -X POST http://cart.shashi5devops.fun:8080/shipping/anonymous-43
```

Result:

```text
Cannot POST /shipping/anonymous-43
```

### Root Cause

The request was sent to the wrong port.

### Solution

```bash
curl -i -X POST http://localhost:8082/shipping/anonymous-43
```

Result:

```text
HTTP/1.1 200 OK
```

---

## Problem 4 – Payment Could Not Delete Cart

Payment logs showed:

```text
cart delete returned 404
```

### Root Cause

Payment was using:

```text
CART_PORT=8080
```

while Cart was actually running on:

```text
8082
```

### Solution

```ini
Environment=CART_PORT=8082
```

Then:

```bash
systemctl daemon-reload
systemctl restart payment
```

---

## Problem 5 – Dispatch 203/EXEC

Systemd reported:

```text
status=203/EXEC
```

### Root Cause

The `/app/dispatch` executable did not exist.

### Solution

Build the Go application:

```bash
cd /app
go mod init dispatch
go get
go build
```

Then:

```bash
systemctl restart dispatch
```

---

## Problem 6 – Dispatch Could Not Find RabbitMQ

Error:

```text
dial tcp: lookup rabbitmq: no such host
```

### Root Cause

The application was trying to resolve:

```text
rabbitmq
```

instead of the configured DNS hostname.

### Solution

```ini
Environment=AMQP_HOST=rabbitmq.shashi5devops.fun
```

After restart:

```text
Rabbit MQ ready true
```

---

# 🧪 Final Validation

## Cart

```bash
curl -i http://cart.shashi5devops.fun:8082/health
```

Expected:

```json
{"app":"OK","redis":true}
```

---

## Shipping

```bash
curl -i http://localhost:8081/health
```

Expected:

```text
OK
```

---

## Dispatch

```bash
journalctl -u dispatch -n 30 -o cat
```

Expected:

```text
Rabbit MQ ready true
```

---

## Payment

```bash
journalctl -u payment -n 50 -o cat
```

Expected flow:

```text
payment for anonymous-43
queue order
connected to broker
message sent
```

---

# 🔄 Complete End-to-End Flow

```text
                    USER
                     │
                     ▼
              ┌─────────────┐
              │   Route 53  │
              └──────┬──────┘
                     │
                     ▼
              ┌─────────────┐
              │   Nginx     │
              │  Frontend   │
              └──────┬──────┘
                     │
                     ▼
             ┌───────────────┐
             │     Cart      │
             │    :8082      │
             └───────┬───────┘
                     │
           ┌─────────┼─────────┐
           ▼         ▼         ▼
      Catalogue    Redis    Shipping
        :8080       :6379     :8081
                              │
                              ▼
                            MySQL
                             :3306
                              │
                              ▼
                          ┌─────────┐
                          │ Payment │
                          │ Python  │
                          └────┬────┘
                               │
                               ▼
                           RabbitMQ
                             :5672
                               │
                               ▼
                          ┌──────────┐
                          │ Dispatch │
                          │   Go     │
                          └──────────┘
```

---

# 🎯 DevOps Skills Demonstrated

This project demonstrates practical experience with:

### Linux

* User management
* File permissions
* Process management
* systemd
* journalctl
* Vim
* grep
* curl
* DNS troubleshooting

### AWS

* EC2
* VPC
* Subnets
* Internet Gateway
* Security Groups
* Route 53
* EC2 quota management

### Microservices

* Service-to-service communication
* REST APIs
* Port management
* DNS-based service discovery
* Environment configuration

### Databases

* MongoDB
* MySQL
* Redis

### Messaging

* RabbitMQ
* Producer / Consumer architecture

### Application Deployment

* Node.js
* Python
* Java Spring Boot
* Go
* JAR deployment
* Go binary compilation

### Troubleshooting

* HTTP 404
* systemd 203/EXEC
* DNS resolution failure
* Wrong application port
* Environment variable mismatch
* RabbitMQ connectivity
* Microservice communication failures

---

# 🏆 Project Result

The RoboShop application was successfully deployed and tested on AWS.

The final validation confirmed:

```text
✅ Frontend accessible
✅ DNS resolution working
✅ Catalogue service working
✅ User service working
✅ Cart service working on port 8082
✅ Redis connection working
✅ Shipping service working
✅ MySQL connectivity working
✅ Payment service working
✅ RabbitMQ connection working
✅ Dispatch service working
✅ End-to-end order successfully placed
```

A successful order generated:

```text
Order placed
171d4f8a-6ace-482e-8243-6b48f8a84a3a
```

This demonstrates a complete working microservices deployment rather than just installing individual applications.

---

# 📌 Key Commands Cheat Sheet

```bash
# Service management
systemctl daemon-reload
systemctl enable <service>
systemctl start <service>
systemctl restart <service>
systemctl status <service> --no-pager

# Logs
journalctl -u <service> -n 50 -o cat

# Service configuration
systemctl cat <service>

# Environment variables
systemctl show <service> -p Environment

# DNS
getent hosts <hostname>

# Process
ps -ef

# Files
ls -lh /app
find /app -maxdepth 2 -type f

# Search source code
grep -RniE "shipping|app.post|router.post|POST" /app

# API testing
curl -i http://localhost:<PORT>/health

# POST API
curl -i -X POST http://localhost:<PORT>/<endpoint> \
-H "Content-Type: application/json" \
-d '<JSON>'

# Go application
dnf install golang -y
go mod init dispatch
go get
go build
```

---

# 🚀 Future Improvements

The current project is a strong manual AWS/Linux deployment project. A production-grade DevOps implementation could further add:

* Terraform for Infrastructure as Code
* Ansible for configuration management
* Jenkins/GitHub Actions for CI/CD
* Docker containers
* Kubernetes/EKS
* Application Load Balancer
* Auto Scaling
* CloudWatch monitoring
* Prometheus + Grafana
* Centralized logging
* AWS Secrets Manager
* HTTPS using ACM
* Private subnets with NAT Gateway
* High availability across Availability Zones

---

# 👨‍💻 Project Summary

**RoboShop – AWS Microservices E-Commerce Deployment**

A hands-on DevOps project involving deployment and troubleshooting of a multi-service e-commerce application on AWS EC2. The project covers Linux administration, DNS, systemd service management, Node.js, Python, Java Spring Boot, Go, MongoDB, MySQL, Redis, RabbitMQ, REST API testing, and end-to-end microservice communication.

The project particularly demonstrates real-world troubleshooting where application ports, DNS records, environment variables, systemd configuration, executable permissions, and service dependencies had to be identified and corrected.

**Status: ✅ Successfully Deployed and End-to-End Order Tested**
