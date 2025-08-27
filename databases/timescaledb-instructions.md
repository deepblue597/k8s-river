# TimescaleDB Distributed Setup Instructions

This setup provides a distributed TimescaleDB cluster with automatic synchronization between replicas, optimized for the river segmentation use case.

## Architecture Overview

- **2 TimescaleDB replicas** in a StatefulSet with automatic failover via Patroni
- **Headless service** for cluster communication
- **Regular service** for application access
- **Continuous aggregates** for real-time statistics
- **Compression and retention policies** for data management

## Deployment Steps

### 1. Create the secrets from the environment file

```bash
# Create Kubernetes secret from the environment file
kubectl create secret generic timescaledb-secret \
  --from-env-file=timescaledb-secrets.env \
  --namespace=your-namespace
```

### 2. Deploy the TimescaleDB cluster

```bash
# Deploy the StatefulSet and services
kubectl apply -f timescaledb-app.yaml
```

### 3. Wait for the cluster to be ready

```bash
# Check pod status
kubectl get pods -l app=timescaledb-app

# Check logs
kubectl logs -f timescaledb-app-0
kubectl logs -f timescaledb-app-1
```

### 4. Initialize the database

```bash
# Run the initialization job
kubectl apply -f timescaledb-init-job.yaml

# Check job status
kubectl get jobs
kubectl logs -f job/timescaledb-init-job
```

## Features Configured

### Database Schema

- **river_segmentation** hypertable with the following columns:
  - `timestamp` (TIMESTAMP, primary key)
  - `model_name` (VARCHAR)
  - `filename` (VARCHAR)
  - `water_coverage` (FLOAT)
  - `avg_confidence` (FLOAT)
  - `overflow_detected` (BOOLEAN)
  - `location` (VARCHAR)

### Continuous Aggregates

- **Daily statistics**: Daily summaries with standard deviation
- **Weekly statistics**: Weekly trends and patterns

### Performance Optimizations

- **Indexes**: On model_name, location, timestamp+location, overflow_detected
- **Compression**: Data older than 3 months is automatically compressed
- **Retention**: Data is kept for 2 years
- **Auto-refresh**: Continuous aggregates refresh automatically

## Connection Information

### Internal cluster access:

- **Service**: `timescaledb-service:5432`
- **Database**: `river`
- **Username**: `postgres`

### Direct pod access:

- **Primary**: `timescaledb-app-0.timescaledb-headless:5432`
- **Replica**: `timescaledb-app-1.timescaledb-headless:5432`

## Example Queries

### Insert data:

```sql
INSERT INTO river_segmentation (timestamp, model_name, filename, water_coverage, avg_confidence, overflow_detected, location)
VALUES (NOW(), 'segmentation_v1', 'river_001.jpg', 0.75, 0.89, false, 'River Point A');
```

### View daily trends:

```sql
SELECT bucket, location, avg_water_coverage, overflow_count
FROM river_segmentation_daily
WHERE bucket >= NOW() - INTERVAL '7 days'
ORDER BY bucket DESC;
```

### View weekly trends:

```sql
SELECT bucket, location, avg_water_coverage, overflow_count, stddev_water_coverage
FROM river_segmentation_weekly
WHERE bucket >= NOW() - INTERVAL '30 days'
ORDER BY bucket DESC;
```

## Monitoring

### Check cluster health:

```bash
kubectl exec -it timescaledb-app-0 -- psql -U postgres -d river -c "SELECT * FROM pg_stat_replication;"
```

### View continuous aggregate status:

```bash
kubectl exec -it timescaledb-app-0 -- psql -U postgres -d river -c "SELECT * FROM timescaledb_information.continuous_aggregates;"
```

## Troubleshooting

### Pod not starting:

1. Check logs: `kubectl logs timescaledb-app-0`
2. Verify secrets: `kubectl get secret timescaledb-secret -o yaml`
3. Check storage: `kubectl get pvc`

### Replication issues:

1. Check Patroni status: `kubectl exec -it timescaledb-app-0 -- patronictl list`
2. Verify network connectivity between pods
3. Check replication lag: Query `pg_stat_replication` table

### Job failed:

1. Check job logs: `kubectl logs job/timescaledb-init-job`
2. Verify database connectivity
3. Re-run job: `kubectl delete job timescaledb-init-job && kubectl apply -f timescaledb-init-job.yaml`
