architecture: standalone
auth:
  enabled: true
  password: "redispass"
master:
  persistence:
    enabled: true
    existingClaim: "redis-pvc"
  resources:
    requests:
      memory: "256Mi"
      cpu: "0.2"
    limits:
      memory: "512Mi"
      cpu: "0.5"
  service:
    type: NodePort
    port: 6379
    nodePort: 30779