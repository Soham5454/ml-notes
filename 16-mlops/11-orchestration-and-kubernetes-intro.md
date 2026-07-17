# 11 — Orchestration & Kubernetes Intro

Recap file 10's closing note and MLOps file 03's Docker Compose scaling preview directly — extending Compose into the genuinely industry-standard tool for running and scaling containerized applications at real production scale.

## Why Docker Compose genuinely stops being enough

```yaml
# Recap MLOps file 03's docker-compose.yml directly -- genuinely excellent for
# LOCAL development and small deployments, but Compose runs on a SINGLE machine
# Real production ML systems (recap file 10's canary deployment discussion --
# needing MULTIPLE instances, load balancing, automatic recovery from failures)
# genuinely need to run across MULTIPLE machines, with automatic scaling and
# self-healing -- this is precisely the gap Kubernetes fills
```

## What Kubernetes actually is — genuinely a container ORCHESTRATOR

```
Kubernetes (K8s) manages containers across a CLUSTER of machines (nodes),
handling automatically:
- SCHEDULING: deciding which machine runs which container
- SCALING: running MORE instances under high load, fewer when idle
- SELF-HEALING: automatically restarting a container that crashes
- LOAD BALANCING: distributing traffic across multiple running instances
- ROLLING UPDATES: deploying new versions gradually (recap file 10's canary
  deployment concept directly -- Kubernetes has native support for this)
```

Genuinely worth understanding K8s as the tool that AUTOMATES everything that would otherwise need to be done manually — recap the general "why automation matters" theme running through this whole MLOps folder, now applied at the infrastructure/deployment layer specifically.

## Core Kubernetes concepts — genuinely the essential vocabulary

```yaml
# A Pod -- the SMALLEST deployable unit, genuinely wraps one or more containers
# (usually just ONE container per pod for simple cases, recap Docker file 02)

# A Deployment -- manages a set of IDENTICAL pod replicas, handles updates/scaling
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sentiment-classifier
spec:
  replicas: 3      # genuinely runs 3 identical instances simultaneously
  selector:
    matchLabels:
      app: sentiment-classifier
  template:
    metadata:
      labels:
        app: sentiment-classifier
    spec:
      containers:
        - name: sentiment-classifier
          image: my-registry/sentiment-classifier:v5      # recap MLOps file 02's Docker image directly
          ports:
            - containerPort: 8000
          resources:
            requests:
              memory: "512Mi"
              cpu: "500m"
            limits:
              memory: "1Gi"
              cpu: "1"
```

Genuinely worth recognizing `replicas: 3` as the FORMALIZED, production-grade version of MLOps file 03's `docker-compose up --scale web=3` preview — Kubernetes doesn't just start 3 instances once, it CONTINUOUSLY monitors and maintains exactly that count, automatically replacing any instance that crashes.

## Service — genuinely how traffic reaches the pods

```yaml
apiVersion: v1
kind: Service
metadata:
  name: sentiment-classifier-service
spec:
  selector:
    app: sentiment-classifier
  ports:
    - port: 80
      targetPort: 8000
  type: LoadBalancer      # genuinely exposes this to external traffic, distributing across all matching pods
```

Genuinely important, worth understanding the split: Pods are EPHEMERAL (recap the general "containers are disposable" framing from MLOps file 03) — they can be created, destroyed, and rescheduled onto different machines constantly. A SERVICE provides a STABLE network identity/address that automatically routes to whichever pods currently exist matching its `selector`, regardless of how many times individual pods get recreated.

## Horizontal Pod Autoscaling — genuinely important for ML workloads with variable traffic

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: sentiment-classifier-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: sentiment-classifier
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70      # genuinely scales UP when average CPU exceeds 70%
```

Genuinely a real, practical benefit for ML serving specifically — recap PyTorch file 13's GPU/compute cost awareness directly: automatically scaling DOWN during low-traffic periods (genuinely saving real compute cost) and scaling UP during high-traffic periods (maintaining acceptable response times) without requiring manual intervention.

## Rolling updates — recap file 10's deployment strategies, natively in Kubernetes

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # genuinely allows 1 EXTRA pod during the update
      maxUnavailable: 0      # genuinely never takes an existing pod down before its replacement is ready
```

Genuinely worth recognizing this as Kubernetes' NATIVE implementation of file 10's blue-green/canary philosophy — `maxUnavailable: 0` ensures ZERO downtime during a deployment: new pods (the new model version) are brought up and verified healthy BEFORE old pods are removed, directly automating the safety principle from that entire file.

## Health checks — recap Flask file 06's /health endpoint, now formalized

```yaml
containers:
  - name: sentiment-classifier
    livenessProbe:
      httpGet:
        path: /health      # recap Flask file 06 directly
        port: 8000
      periodSeconds: 10
    readinessProbe:
      httpGet:
        path: /health
        port: 8000
      periodSeconds: 5
```

Genuinely important distinction worth being precise about: a **liveness probe** checks "is this container still ALIVE" (if it fails repeatedly, Kubernetes RESTARTS the container); a **readiness probe** checks "is this container READY to receive traffic" (if it fails, Kubernetes STOPS routing traffic to it, WITHOUT restarting it — genuinely useful for a container that's alive but temporarily busy, e.g. still loading a large model file, recap Flask file 06's "load model once at startup" discussion — the readiness probe can genuinely prevent traffic from arriving before that loading completes).

## ConfigMaps and Secrets — recap Flask/Docker's environment variable discipline

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  secret-key: <base64-encoded-value>      # recap the "never hardcode secrets" principle,
                                               # now Kubernetes' own dedicated mechanism for it
```

Genuinely worth recognizing this as Kubernetes' formalized version of the SAME environment-variable/secrets discipline from Flask files 08/10/11 and MLOps file 02 — `ConfigMap` for non-sensitive configuration, `Secret` for genuinely sensitive values, both injected into pods as environment variables at RUNTIME, never baked into the container image itself.

## kubectl — genuinely the primary tool for interacting with a cluster

```bash
kubectl apply -f deployment.yaml      # genuinely applies/updates the desired state described in the YAML
kubectl get pods                        # lists running pods
kubectl logs <pod-name>                   # recap Docker file 02's docker logs directly, same underlying idea
kubectl scale deployment sentiment-classifier --replicas=5      # manual scaling override
kubectl rollout undo deployment sentiment-classifier      # genuinely instant rollback, recap file 10 directly
```

## The honest, real complexity tradeoff — worth stating directly

```
Genuinely important, honest note: Kubernetes has a REAL, steep learning curve
and genuinely adds significant operational complexity -- it's NOT the right
choice for a small portfolio project or a single-instance demo (recap Flask
file 11's Render/Railway recommendation directly, which remains genuinely correct
for that scale)
Kubernetes earns its complexity at REAL scale: multiple services, genuine need
for auto-scaling, high-availability requirements, or a team managing many
different ML models/services simultaneously
```

Genuinely the same honest engineering-tradeoff framework echoed throughout this ENTIRE roadmap, one more time — knowing Kubernetes exists and understanding its core concepts is genuinely valuable interview/career knowledge, but reaching for it prematurely on a small project would be a real, unnecessary complexity cost.

## Try this

```yaml
# Genuinely worth doing this with a LOCAL Kubernetes tool (minikube or Docker
# Desktop's built-in Kubernetes) rather than a real cloud cluster, to practice
# safely and for free:
# Take the Dockerized Flask ML app from MLOps file 02's exercise and write
# a Deployment (3 replicas) and a Service (LoadBalancer type) for it
# Apply it with kubectl apply, confirm 3 pods are running with kubectl get pods
# Deliberately kill one pod manually (kubectl delete pod <name>) and observe
# Kubernetes automatically create a replacement -- confirming self-healing
# Then scale to 5 replicas with kubectl scale, and back down to 2
```

Actually watching Kubernetes automatically replace a manually-killed pod is genuinely the clearest, most concrete demonstration of "self-healing" — a real, tangible proof of the automation this whole file has been describing conceptually.

## What's next
File 12 — the final MLOps → Cloud Bridge, pulling every concept from files 01-11 together and connecting them to the managed cloud services (AWS SageMaker, GCP Vertex AI, Azure ML) covered properly in the upcoming Cloud folder.
