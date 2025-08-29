# Kafka Cluster Deployment Instructions (Strimzi)

## Overview

This setup deploys a production-ready Apache Kafka cluster using **Strimzi** operator with:

- **3 Kafka brokers** in KRaft mode (no Zookeeper needed)
- **Kafka UI** for cluster management and monitoring
- **Cross-namespace service** for river-model access
- **Persistent storage** for data durability

## Architecture

```
River Model (default ns) → kafka-service → Kafka Cluster (kafka ns)
                                         ↓
                                    Kafka UI (kafka ns)
```

## Components

### 1. Strimzi Operator

- Manages Kafka cluster lifecycle
- Provides Custom Resource Definitions (CRDs)
- Handles rolling updates and scaling

### 2. Kafka Cluster (river-kafka)

- **3 replicas** for high availability
- **KRaft mode** (no Zookeeper dependency)
- **Internal + External listeners** for flexible access
- **Persistent volumes** for data storage
- **Metrics collection** for monitoring

### 3. Kafka Topic (river-images)

- **6 partitions** for parallel processing
- **3 replicas** for fault tolerance
- **7-day retention** for data lifecycle

### 4. Kafka UI

- Web-based cluster management
- Topic and consumer monitoring
- Message browsing capabilities

## Deployment Steps

### Step 1: Verify Strimzi Operator

```bash
# Check operator status
kubectl get pods -n kafka
kubectl logs deployment/strimzi-cluster-operator -n kafka
```

### Step 2: Deploy Kafka Service and UI

```bash
# Deploy the Kafka service (internal access)
kubectl apply -f kafka/kafka-service.yaml

# Deploy the Kafka UI
kubectl apply -f kafka/kafka-ui.yaml

# Check UI status
kubectl get pods -n kafka -l app=kafka-ui
# Verify service
kubectl get svc -n kafka kafka-service
```

## Configuration Details

### Kafka Cluster Specs

- **Version**: Apache Kafka 4.0.0
- **Mode**: KRaft (Kafka Raft consensus)
- **Brokers**: 3 replicas for HA
- **Storage**: 5Gi persistent volumes per broker (adjust as needed)
- **Resources**: 1Gi memory, 500m CPU per broker (adjust for your environment)

### Security & Access

- **Internal**: Plain + TLS listeners
- **External**: NodePort for external access
- **Cross-namespace**: ExternalName service

### Performance Settings

- **Replication Factor**: 3 (fault tolerance)
- **Min In-Sync Replicas**: 2 (consistency)
- **Auto-create Topics**: Enabled (development)
- **Compression**: Producer-controlled

## Access Information

### Kafka UI

- **URL**: `http://localhost:30808`
- **Features**: Cluster overview, topics, consumers, messages

### Kafka Endpoints

- **Internal**: `my-cluster-kafka-bootstrap.kafka.svc.cluster.local:9092`
- **From Default NS**: `kafka-service.kafka.svc.cluster.local:9092` (use this in backend/model configs)
- **External**: NodePort via Kafka UI (if exposed)

## Monitoring & Verification

### Check Cluster Health

```bash
# Cluster status
kubectl get kafka river-kafka -n kafka -o yaml

# Pod status
kubectl get pods -n kafka

# Logs
kubectl logs river-kafka-kafka-0 -n kafka
```

### Check Topic

```bash
# Topic details
kubectl describe kafkatopic river-images -n kafka

# Topic status
kubectl get kafkatopics -n kafka
```

### Test Connectivity

```bash
# From river-model pod
kubectl exec deployment/river-model -- nslookup kafka-service

# Check if topic exists (via Kafka UI or kubectl)
kubectl get kafkatopics -n kafka
```

## Troubleshooting

### Common Issues

1. **Operator not ready**: Wait for `strimzi-cluster-operator` pod to be Running
2. **Cluster creation timeout**: Check resource constraints and storage class
3. **Cross-namespace access**: Verify `kafka-service` ClusterIP configuration
4. **Topic not found**: Wait for Topic Operator to process KafkaTopic resource

### Debug Commands

```bash
# Operator logs
kubectl logs deployment/strimzi-cluster-operator -n kafka

# Cluster operator logs
kubectl logs river-kafka-entity-operator -c topic-operator -n kafka

# Kafka broker logs
kubectl logs river-kafka-kafka-0 -n kafka

# Service resolution test
kubectl run test-pod --image=busybox -it --rm -- nslookup kafka-service
```

## Performance Tuning

### Scaling Brokers

```bash
# Scale to 5 brokers
kubectl patch kafka river-kafka -n kafka --type='merge' -p='{"spec":{"kafka":{"replicas":5}}}'
```

### Topic Partitions

```bash
# Increase partitions for higher throughput
kubectl patch kafkatopic river-images -n kafka --type='merge' -p='{"spec":{"partitions":12}}'
```

### Resource Limits

- Adjust memory/CPU based on workload
- Monitor JVM heap usage via metrics
- Scale storage as needed

## Security Considerations

- **Network Policies**: Restrict cross-namespace access
- **TLS**: Enable for production workloads
- **Authentication**: Add SASL/SCRAM for user management
- **Authorization**: Implement ACLs for topic access control

## Next Steps

1. **Deploy Kafka cluster** using these instructions
2. **Verify river-model connectivity** to kafka-service
3. **Monitor throughput** via Kafka UI
4. **Set up Grafana dashboards** for Kafka metrics
