# Walk along

This document serves as a guide that will walk you through the process of migrating a Java application to Azure using GitHub Copilot modernization. The workshop covers assessment, Java/framework upgrades, migration to Azure services, containerization, and deployment.

**What the modernization Process Will Do**: The modernization will transform your application from the outdated technologies to a modern Azure-native solution. This includes upgrading from Java 8 to Java 21, migrating from Spring Boot 2.x to 3.x, replacing AWS S3 with Azure Blob Storage, switching from RabbitMQ to Azure Service Bus, migrating to Azure Database for PostgreSQL, implementing managed identity authentication, adding health checks, containerizing the applications, and preparing them for cloud deployment with proper monitoring.

## Overview

In this demo, you will use the GitHub Copilot app modernization extension to assess, upgrade, migrate, and finally deploy the project to Azure

## Current Architecture

flowchart TD

%% Applications
WebApp[Web Application]
Worker[Worker Service]

%% Storage Components
S3[(AWS S3)]
LocalFS[("Local File System<br/>dev only")]

%% Message Broker
RabbitMQ(RabbitMQ)

%% Database
PostgreSQL[(PostgreSQL)]

%% Queues
Queue[image-processing queue]
RetryQueue[image-processing.retry queue]

%% User
User([User])

%% User Flow
User -->|Upload Image| WebApp
User -->|View Images| WebApp

%% Web App Flows
WebApp -->|Store Original Image| S3
WebApp -->|Store Original Image| LocalFS
WebApp -->|Send Processing Message| RabbitMQ
WebApp -->|Store Metadata| PostgreSQL
WebApp -->|Retrieve Images| S3
WebApp -->|Retrieve Images| LocalFS
WebApp -->|Retrieve Metadata| PostgreSQL

%% RabbitMQ Flow
RabbitMQ -->|Push Message| Queue
Queue -->|Processing Failed| RetryQueue
RetryQueue -->|After 1 min delay| Queue
Queue -->|Consume Message| Worker

%% Worker Flow
Worker -->|Download Original| S3
Worker -->|Download Original| LocalFS
Worker -->|Upload Thumbnail| S3
Worker -->|Upload Thumbnail| LocalFS
Worker -->|Store Metadata| PostgreSQL
Worker -->|Retrieve Metadata| PostgreSQL

%% Styling
classDef app fill:#90caf9,stroke:#0d47a1,color:#0d47a1
classDef storage fill:#a5d6a7,stroke:#1b5e20,color:#1b5e20
classDef broker fill:#ffcc80,stroke:#e65100,color:#e65100
classDef db fill:#ce93d8,stroke:#4a148c,color:#4a148c
classDef queue fill:#fff59d,stroke:#f57f17,color:#f57f17
classDef user fill:#ef9a9a,stroke:#b71c1c,color:#b71c1c

class WebApp,Worker app
class S3,LocalFS storage
class RabbitMQ broker
class PostgreSQL db
class Queue,RetryQueue queue
class User user

## Run locally

Clone the repository and run set up script

Prerequisites:

- JDK 8: Required for running the initial application locally.
- Maven 3.6.0+: Required to build the application locally.
- Docker: Required for running the application locally.

Run the following commands to set up the enviroment .
Windows:
```./setup.ps1```