---
layout: post
title: Scaling Postgres Databases with Liquibase on Kubernetes
date: 2023-09-06
categories: [Kubernetes, Liquibase, Postgres]
---

I recently dived into scaling stateful applications in Kubernetes, focusing particularly on Postgres databases. My current project also integrates Liquibase for database migrations, which led me to explore its impact on scaling. Here are the consolidated notes from my research.

## Postgres Helm Charts

### Configuration

We have been using the [Bitnami Postgres Chart](https://github.com/bitnami/charts/tree/main/bitnami/postgresql/).

- **Setting Replica Count**: Utilize the 'replicaCount' chart parameter to scale the cluster horizontally. This is the normal field for scaling in Kubernetes.
  - Ensure the replica count is an odd number.
  - To enable replication, use: '--set replication.enabled=true'.
  - The default architecture is 'architecture=standalone'; switch to 'replication'.
  - The default update strategy is 'primary.updateStrategy.type=RollingUpdate'.

### Replication

Postgres doesn't inherently support multi-master clustering. Employ a single master node and complement it with multiple read replicas. If needed, the master node can fail over to the read replicas.

- **Connection Pooling and Routing**: PGBouncer is a reliable option for connection pooling.
- **Failover**: The Helm chart is designed to automatically manage failover if the primary node becomes unresponsive. Nonetheless, testing this behavior is essential.
- **Node Anti-Affinity**: Adopt soft node anti-affinity to ensure that replicas are distributed across Kubernetes cluster nodes.

### High Availability Version

I think we should switch to [Bitnami HA version](https://github.com/bitnami/charts/tree/main/bitnami/postgresql-ha) to enhance replication and overall availability.

- **Replica Count**: By default, 'postgresql.replicaCount=3' (ensure it's an odd number). This configuration results in one write replica master node and two read replicas.
- **Witness Nodes**: These can be configured to manage network partitions.
- **Pgpool**: This tool manages read/write connections and offers features like read replica promotions, query parallelization, caching, connection limits, and failover mechanisms.
- **Repmgr**: Simplifies the setup, provides failover support, monitoring, switchover functionality, WAL shipping, and integrates smoothly with other tools. It also facilitates timely notifications.

## Liquibase

Using Liquibase, Spring Boot, and Kubernetes in tandem presents a challenge. The Liveness and Readiness Probes might inadvertently terminate an application pod during a migration, resulting in migration lock issues.

### Options

1. **Extend Probe Deadlines**:
   - Pros: Offers simplicity.
   - Cons: Might introduce interruptions, extended wait durations, and potential readiness challenges.
2. **Lifecycle Hooks**:
   - Pros: Guarantees lock release before any termination.
   - Cons: Reliability of the script is crucial, and there's a risk of applications getting stuck.
3. **Separate Container (Job)**:
   - Pros: Minimizes the risk of probe-induced interruptions.
   - Cons: Needs coordination within the pipeline and appropriate resource allocation.
4. **Use Init Container**:
   - Pros: Unaffected by probes and avoids additional pods or coordination overhead.
   - Cons: The migration process remains closely tied to the application.

### Recommended Migration Approach

- **Separation**: Use Kubernetes Jobs or Init Containers to decouple the migration from the application startup.
- **Probes**: Set up Kubernetes readiness and liveness probes accurately.
- **Monitoring and Alerts**: Monitoring logs and setting up alerts are imperative.
- **Backup**: Ensure the database is backed up before initiating any migrations.
- **Testing**: It's crucial to test migrations in an environment that mirrors the production setup.
- **Version Control and Documentation**: Maintain a comprehensive history of all changes for clarity.

#### Steps

1. **Preparation**:
   - Secure a backup of the primary database.
   - Test in a staging environment.
   - Implement monitoring and alerts.
2. **Handling Replicas**:
   - Pause or manage replication while the migration is ongoing.
3. **Migration Execution**:
   - Opt for a maintenance window during low-traffic periods.
   - Remain cognizant of Liquibase's locking
