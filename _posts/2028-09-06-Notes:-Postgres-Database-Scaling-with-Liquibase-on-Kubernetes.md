---
layout: post
title: Postgres Database Scaling with Liquibase on Kubernetes
date: 2023-09-06
catagories: Kubernetes, Liquibase, Postgres, Database
---
Did some research this last week on scaling stateful applications in Kubernetes, specifically with Postgres databases. The project I'm working at work also involves Liquibase for database migrations, so researched how that impacts scaling as well. The following is the cleaned up notes I took.

Scaling

Configuration
- Configuration Options: https://github.com/bitnami/charts/tree/main/bitnami/postgresql/
- Setting Replica Count: Use the 'replicaCount' chart parameter to scale the cluster horizontally. After setting the password, perform a regular chart upgrade.
  - Replica count should be an odd number
  - Enable Replication: '--set replication.enabled=true'
  - Architecture: Default is 'architecture=standalone', change to 'replication'
  - Update Strategy: Default is 'primary.updateStrategy.type=RollingUpdate'

Replication
- Postgres doesn't natively support multi-master clustering. Employ a single master node with multiple read replicas. The master node can fail over to the read replicas.
- Connection Pooling and Routing: Consider PGBouncer for connection pooling.
- Failover: The Helm chart should automatically handle failover if the primary node goes down. Testing this functionality is recommended.
- Node Anti-Affinity: Utilize soft node anti-affinity to ensure replicas distribute across different Kubernetes nodes.

High Availability Version
- Switching to HA: Consider the HA version for better replication and availability. https://github.com/bitnami/charts/tree/main/bitnami/postgresql-ha
- Replica Count: Default is 'postgresql.replicaCount=3' (always an odd number). This provides one write replica master node and two read replicas.
- Witness Nodes: Can be configured for handling network partitions.
- Pgpool: Manages read/write connections and offers features like promoting read replicas, parallelizing queries, caching, connection limiting, and failover handling.
- Repmgr: Streamlines setup, offers failover support, monitoring, switchover functionality, WAL shipping, integration with other tools, and notifications.

Liquibase

Liquibase, Spring Boot, and Kubernetes
- Problem Statement: Liveness and Readiness Probes may inadvertently kill an application pod during a migration, leading to migration lock issues.

Options:
1. Extend Probe Deadlines:
   - Pros: Simplicity
   - Cons: Possible interruptions, longer wait times, potential readiness issues
2. Lifecycle Hooks:
   - Pros: Ensures lock release before termination
   - Cons: Script reliability, potential stuck applications
3. Separate Container (Job):
   - Pros: Reduces risk of probe-induced interruptions
   - Cons: Requires pipeline coordination, needs resource allocation
4. Use Init Container:
   - Pros: Not probe-affected, no extra pods or coordination overhead
   - Cons: Migration remains tied to the application

Recommended Migration Approach
- Separation: Use Kubernetes Jobs or Init Containers to separate migration from application startup.
- Probes: Configure Kubernetes readiness and liveness probes.
- Monitoring and Alerts: Log monitoring and alert setup is crucial.
- Backup: Always backup the database before migrations.
- Testing: Test migrations in a mirrored environment.
- Version Control and Documentation: Maintain a history of changes.

Steps

1. Preparation:
   - Backup the primary database.
   - Test in a staging environment.
   - Set up monitoring and alerts.
2. Handling Replicas:
   - Pause or manage replication during migration.
3. Migration Execution:
   - Use a low-traffic maintenance window.
   - Be aware of Liquibase locking.
   - Consider batching changesets.
4. Post-Migration:
   - Ensure replication consistency.
   - Test the system post-migration.
   - Monitor system performance.
5. Rollback Strategy:
   - Use Liquibase rollback or backup restoration.
6. Documentation:
   - Document all migration steps and issues.
7. Communication:
   - Keep stakeholders informed.

Note on locking during migrations
When Liquibase mentions "locking", it refers to a lock on the DATABASECHANGELOGLOCK table, ensuring only one Liquibase instance updates the schema at once.

- Liquibase Lock Mechanism: Only one instance can acquire the lock and proceed with the migration.
- Impact on Applications: This lock doesn't affect regular application operations but ensures non-concurrent migrations.
- Actual Database Locks: Operations like 'ALTER TABLE' might temporarily lock specific parts of the database.