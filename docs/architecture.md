# TradeFlow Architecture

## Overview
TradeFlow is a microservices-based system consisting of 5 main services, an API gateway, a load balancer, and a distributed MongoDB database. It is orchestrated via Kubernetes and deployed using Helm and GitOps practices.

## Traffic Flow
1. **External Traffic**: Internet traffic hits **HAProxy**, acting as the external ingress and load balancer.
2. **Frontend Routing**: HAProxy routes traffic to the **Frontend (React/Vite)** service.
3. **Internal Routing**: The Frontend application makes API requests to **KGateway** (the internal API gateway).
4. **Microservices**: KGateway routes the requests to the appropriate backend microservice (`user-service`, `stock-service`, `trading-service`, `portfolio-service`).
5. **Database Layer**: Microservices communicate with the **MongoDB ReplicaSet**, each using its dedicated database (`user-db`, `stock-db`, `trading-db`, `portfolio-db`).

## Storage
- **NFS Storage**: Persistent volumes are backed by an EC2-hosted NFS server. The `nfs-storage` StorageClass is used across the cluster.
- **MongoDB**: Runs as a StatefulSet with 3 replicas (1 Primary, 2 Secondaries) to ensure high availability and data redundancy.

## Security & Reliability
- **Admission Controller**: Kyverno enforces security policies preventing the use of `latest` tags, requiring resource limits, and denying privileged containers.
- **Scaling**: All microservices are equipped with HorizontalPodAutoscalers (HPA) scaling between 2 and 5 replicas based on CPU utilization.
- **Probes**: All deployments implement Liveness, Readiness, and Startup probes to ensure traffic is only routed to healthy pods and slow-starting containers are accommodated.
