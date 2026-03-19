# Kubernetes Deployment for MCP Servers

Remote MCP servers running as HTTP services need production-grade Kubernetes deployment with rolling updates, horizontal scaling, and proper resource limits.

## Deployment Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mcp-server
  labels:
    app: mcp-server
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0          # Zero-downtime: never reduce below desired count
  selector:
    matchLabels:
      app: mcp-server
  template:
    metadata:
      labels:
        app: mcp-server
    spec:
      containers:
      - name: mcp-server
        image: your-registry/mcp-server:latest
        ports:
        - containerPort: 3000
        env:
        - name: MCP_SESSION_SECRET
          valueFrom:
            secretKeyRef:
              name: mcp-secrets
              key: session-secret
        resources:
          requests:
            cpu: "250m"
            memory: "256Mi"
          limits:
            cpu: "1000m"
            memory: "512Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 10
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 10
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: mcp-server-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: mcp-server
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

## Session Affinity

For stateful MCP servers (those maintaining session context in-memory), add session affinity to the Service:

```yaml
apiVersion: v1
kind: Service
spec:
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 3600  # 1 hour session stickiness
```

Alternatively, externalize session state to Redis and drop the affinity requirement — enabling true horizontal scalability.

## Critical: Graceful Shutdown

Handle `SIGTERM` to complete in-flight tool calls before the pod terminates:

```typescript
process.on("SIGTERM", async () => {
  await server.close();   // Stop accepting new connections
  await drainInflight();  // Wait for active tool calls to complete
  process.exit(0);
});
```

**Why it matters:** MCP servers under agentic load need autoscaling. Without proper rolling update config, deployments interrupt active agent sessions and corrupt in-progress tool calls.

**Source:** [Stacklok — Performance Testing MCP Servers in Kubernetes](https://dev.to/stacklok/performance-testing-mcp-servers-in-kubernetes-transport-choice-is-the-make-or-break-decision-for-1ffb); [modelcontextprotocol.io](https://modelcontextprotocol.io) deployment guide
