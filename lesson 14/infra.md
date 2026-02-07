```mermaid
C4Deployment
    title Infrastructure: HA & Geo-Redundancy (Fixed)

    %% Внешний балансировщик (Global Traffic)
    Container_Ext(cdn, "Global Traffic Manager", "Cloudflare / Route53", "DNS Balancing & DDoS Protection")

    %% === ZONE 1 (PRIMARY) ===
    Deployment_Node(az1, "Зона 1", "Регион 1") {

        Deployment_Node(compute_1, "Compute Layer", "K8s Cluster A") {
            Container(waf_1, "WAF Ingress", "Nginx", "Entry Point")
            Container(apps_1, "Critical Apps", "Pods", "Auth, Catalog, Order")
        }

        Deployment_Node(data_1, "Data Layer", "High IOPS SSD") {
            ContainerDb(db_master, "Postgres Master", "Primary", "Read/Write")
        }
    }

    %% === ZONE 2 (STANDBY / FAILOVER) ===
    Deployment_Node(az2, "Зона 2", "Регион 2") {

        Deployment_Node(compute_2, "Compute Layer", "K8s Cluster B") {
            Container(waf_2, "WAF Ingress", "Nginx", "Entry Point")
            Container(apps_2, "Critical Apps", "Pods", "Standby Replicas")
        }

        Deployment_Node(data_2, "Data Layer", "High IOPS SSD") {
            ContainerDb(db_sync, "Postgres Sync", "Replica", "Failover Candidate")
        }
    }

    %% === ZONE 3 (QUORUM / DRP) ===
    Deployment_Node(az3, "Зона 3", "Регион 3") {

        Deployment_Node(compute_3, "Compute Layer", "K8s Cluster C") {
            Container(apps_3, "Non-Critical Apps", "Pods", "Analytics, Quality UI")
        }

        Deployment_Node(data_3, "Data Layer", "Cold Storage") {
            ContainerDb(db_async, "Postgres Async", "Replica", "DRP Backup")
            ContainerDb(s3, "S3 Backup", "Object Store", "WAL Archive / Dumps")
        }
    }

    %% === СВЯЗИ ===

    %% Traffic Distribution
    Rel(cdn, waf_1, "Route Traffic", "Active")
    Rel(cdn, waf_2, "Route Traffic", "Passive/Active")

    %% App to DB
    Rel(apps_1, db_master, "Write/Read", "SQL")
    Rel(apps_2, db_master, "Write/Read", "SQL (Cross-Zone)")

    %% Database Replication (The Core of Reliability)
    Rel(db_master, db_sync, "Sync Replication", "Zero Data Loss")
    Rel(db_master, db_async, "Async Replication", "Eventual Consistency")
    Rel(db_master, s3, "WAL Archiving", "Point-in-Time Recovery")

    UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")