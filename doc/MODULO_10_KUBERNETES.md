# ☸️ Módulo 10: Kubernetes para Desenvolvedores Java

> **Objetivo:** Dominar orquestração de containers com Kubernetes, essencial para ambientes cloud.

---

## 📌 Por que Kubernetes?

- **Padrão de mercado** para orquestração de containers
- **Portabilidade:** Funciona em qualquer cloud (AWS EKS, Azure AKS, GCP GKE)
- **Auto-scaling:** Escala aplicações automaticamente
- **Self-healing:** Reinicia containers que falham
- **Exigido em ~70%** das vagas Java Senior

---

## 🎯 O que você vai aprender

| Seção | Tópico | Importância |
|-------|--------|-------------|
| 10.1 | Conceitos Fundamentais | 🔴 Crítico |
| 10.2 | Pods e Containers | 🔴 Crítico |
| 10.3 | Deployments | 🔴 Crítico |
| 10.4 | Services e Networking | 🔴 Crítico |
| 10.5 | ConfigMaps e Secrets | 🔴 Crítico |
| 10.6 | Ingress | 🟡 Importante |
| 10.7 | Health Checks | 🔴 Crítico |
| 10.8 | Horizontal Pod Autoscaler | 🟡 Importante |
| 10.9 | Helm Charts | 🟡 Importante |
| 10.10 | Ambiente Local (Minikube) | 🔴 Crítico |

---

## 10.1 Conceitos Fundamentais

### Arquitetura Kubernetes

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            KUBERNETES CLUSTER                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌────────────────────────────────────────────────────────────────────────┐ │
│  │                         CONTROL PLANE                                   │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐      │ │
│  │  │  API    │  │  etcd   │  │Scheduler│  │Controller│  │  Cloud  │      │ │
│  │  │ Server  │  │         │  │         │  │ Manager │  │Controller│      │ │
│  │  └─────────┘  └─────────┘  └─────────┘  └─────────┘  └─────────┘      │ │
│  └────────────────────────────────────────────────────────────────────────┘ │
│                                     │                                        │
│                                     ▼                                        │
│  ┌──────────────────────┐  ┌──────────────────────┐  ┌──────────────────┐  │
│  │      WORKER NODE     │  │      WORKER NODE     │  │    WORKER NODE   │  │
│  │  ┌────────────────┐  │  │  ┌────────────────┐  │  │  ┌────────────┐  │  │
│  │  │    kubelet     │  │  │  │    kubelet     │  │  │  │  kubelet   │  │  │
│  │  └────────────────┘  │  │  └────────────────┘  │  │  └────────────┘  │  │
│  │  ┌────────────────┐  │  │  ┌────────────────┐  │  │  ┌────────────┐  │  │
│  │  │   kube-proxy   │  │  │  │   kube-proxy   │  │  │  │ kube-proxy │  │  │
│  │  └────────────────┘  │  │  └────────────────┘  │  │  └────────────┘  │  │
│  │  ┌─────┐ ┌─────┐     │  │  ┌─────┐ ┌─────┐     │  │  ┌─────┐        │  │
│  │  │ Pod │ │ Pod │     │  │  │ Pod │ │ Pod │     │  │  │ Pod │        │  │
│  │  └─────┘ └─────┘     │  │  └─────┘ └─────┘     │  │  └─────┘        │  │
│  └──────────────────────┘  └──────────────────────┘  └──────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Objetos Principais

| Objeto | Descrição | Analogia |
|--------|-----------|----------|
| **Pod** | Menor unidade, 1+ containers | Container wrapper |
| **Deployment** | Gerencia réplicas de Pods | Auto Scaling Group |
| **Service** | Expõe Pods na rede | Load Balancer interno |
| **Ingress** | Entrada HTTP externa | Reverse Proxy |
| **ConfigMap** | Configurações não-sensíveis | Properties file |
| **Secret** | Dados sensíveis | Vault (básico) |
| **Namespace** | Isolamento lógico | Ambiente/Team |

---

## 10.2 Pods e Containers

### Pod Básico

```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: account-service
  labels:
    app: account-service
    version: v1
spec:
  containers:
    - name: account-service
      image: pix-account-service:1.0.0
      ports:
        - containerPort: 8080
      env:
        - name: SPRING_PROFILES_ACTIVE
          value: "kubernetes"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secret
              key: url
      resources:
        requests:
          memory: "256Mi"
          cpu: "250m"
        limits:
          memory: "512Mi"
          cpu: "500m"
```

### Resources: Requests vs Limits

```yaml
resources:
  # Garantido - usado para scheduling
  requests:
    memory: "256Mi"    # Mínimo garantido
    cpu: "250m"        # 0.25 CPU cores
  
  # Máximo - pod é terminado se exceder
  limits:
    memory: "512Mi"    # OOMKilled se exceder
    cpu: "500m"        # Throttled se exceder
```

---

## 10.3 Deployments

### Deployment Completo

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: account-service
  namespace: pix-system
  labels:
    app: account-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: account-service
  
  # Estratégia de atualização
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Pods extras durante update
      maxUnavailable: 0  # Nenhum pod indisponível
  
  template:
    metadata:
      labels:
        app: account-service
        version: v1
    spec:
      containers:
        - name: account-service
          image: 123456789.dkr.ecr.sa-east-1.amazonaws.com/pix-account-service:1.0.0
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
              name: http
            - containerPort: 8081
              name: management
          
          envFrom:
            - configMapRef:
                name: account-service-config
            - secretRef:
                name: account-service-secrets
          
          resources:
            requests:
              memory: "512Mi"
              cpu: "500m"
            limits:
              memory: "1Gi"
              cpu: "1000m"
          
          # Health checks
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: management
            initialDelaySeconds: 30
            periodSeconds: 10
          
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: management
            initialDelaySeconds: 10
            periodSeconds: 5
      
      # Tolerations e Affinity (opcional)
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app: account-service
                topologyKey: kubernetes.io/hostname
```

### Comandos Úteis

```bash
# Aplicar deployment
kubectl apply -f deployment.yaml

# Ver status
kubectl get deployments -n pix-system
kubectl get pods -n pix-system

# Escalar
kubectl scale deployment account-service --replicas=5 -n pix-system

# Rollout status
kubectl rollout status deployment/account-service -n pix-system

# Rollback
kubectl rollout undo deployment/account-service -n pix-system

# Logs
kubectl logs -f deployment/account-service -n pix-system

# Entrar no pod
kubectl exec -it <pod-name> -n pix-system -- /bin/sh
```

---

## 10.4 Services e Networking

### Tipos de Service

| Tipo | Descrição | Uso |
|------|-----------|-----|
| **ClusterIP** | IP interno do cluster | Comunicação entre services |
| **NodePort** | Porta em cada node | Desenvolvimento/debug |
| **LoadBalancer** | Load balancer externo | Produção (cloud) |
| **ExternalName** | DNS CNAME | Serviços externos |

### Service ClusterIP

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: account-service
  namespace: pix-system
spec:
  type: ClusterIP
  selector:
    app: account-service
  ports:
    - name: http
      port: 80           # Porta do service
      targetPort: 8080   # Porta do container
      protocol: TCP
```

### Comunicação entre Services

```yaml
# No Transfer Service, para chamar Account Service:
spring:
  application:
    name: transfer-service

# URL interna do Kubernetes
account-service:
  url: http://account-service.pix-system.svc.cluster.local
  # Formato: <service-name>.<namespace>.svc.cluster.local
```

```java
@FeignClient(
    name = "account-service",
    url = "${account-service.url}"
)
public interface AccountClient {
    @GetMapping("/api/v1/accounts/{id}")
    AccountResponse getAccount(@PathVariable UUID id);
}
```

---

## 10.5 ConfigMaps e Secrets

### ConfigMap

```yaml
# configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: account-service-config
  namespace: pix-system
data:
  SPRING_PROFILES_ACTIVE: "kubernetes,prod"
  SERVER_PORT: "8080"
  MANAGEMENT_SERVER_PORT: "8081"
  LOGGING_LEVEL_ROOT: "INFO"
  LOGGING_LEVEL_COM_BANK: "DEBUG"
  
  # Arquivo de configuração completo
  application.yml: |
    spring:
      datasource:
        hikari:
          maximum-pool-size: 10
          minimum-idle: 2
    management:
      endpoints:
        web:
          exposure:
            include: health,info,metrics,prometheus
```

### Secret

```yaml
# secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: account-service-secrets
  namespace: pix-system
type: Opaque
stringData:  # Texto plano (será encodado automaticamente)
  DATABASE_URL: "jdbc:postgresql://rds-pix.xxx.sa-east-1.rds.amazonaws.com:5432/pix"
  DATABASE_USERNAME: "pix_user"
  DATABASE_PASSWORD: "super-secret-password"
  AWS_ACCESS_KEY_ID: "AKIAXXXXXXX"
  AWS_SECRET_ACCESS_KEY: "xxxxxxxxxxxxx"
```

### Montando como Volume

```yaml
spec:
  containers:
    - name: account-service
      volumeMounts:
        - name: config-volume
          mountPath: /config
          readOnly: true
  volumes:
    - name: config-volume
      configMap:
        name: account-service-config
        items:
          - key: application.yml
            path: application.yml
```

---

## 10.6 Ingress

### NGINX Ingress Controller

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: pix-ingress
  namespace: pix-system
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/rewrite-target: /$2
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  tls:
    - hosts:
        - api.pix.bank.com
      secretName: pix-tls-secret
  rules:
    - host: api.pix.bank.com
      http:
        paths:
          - path: /accounts(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: account-service
                port:
                  number: 80
          
          - path: /transfers(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: transfer-service
                port:
                  number: 80
          
          - path: /recurring(/|$)(.*)
            pathType: Prefix
            backend:
              service:
                name: recurring-service
                port:
                  number: 80
```

### Fluxo de Requisição

```
┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│  Client  │────►│ Ingress  │────►│ Service  │────►│   Pod    │
│          │     │Controller│     │          │     │          │
└──────────┘     └──────────┘     └──────────┘     └──────────┘
                      │
                      │ /accounts/* → account-service
                      │ /transfers/* → transfer-service
                      │ /recurring/* → recurring-service
```

---

## 10.7 Health Checks

### Tipos de Probes

| Probe | Propósito | Falha = |
|-------|-----------|---------|
| **livenessProbe** | Aplicação está viva? | Restart container |
| **readinessProbe** | Pronta para tráfego? | Remove do Service |
| **startupProbe** | Inicializou? | Delay liveness/readiness |

### Spring Boot Actuator

```yaml
# application.yml
management:
  server:
    port: 8081
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: always
      probes:
        enabled: true
  health:
    livenessState:
      enabled: true
    readinessState:
      enabled: true
```

### Kubernetes Probes

```yaml
containers:
  - name: account-service
    ports:
      - containerPort: 8080
        name: http
      - containerPort: 8081
        name: management
    
    # Aplicação travou? Restart!
    livenessProbe:
      httpGet:
        path: /actuator/health/liveness
        port: management
      initialDelaySeconds: 60    # Esperar inicialização
      periodSeconds: 10          # Verificar a cada 10s
      timeoutSeconds: 5          # Timeout de 5s
      failureThreshold: 3        # 3 falhas = restart
    
    # Pronta para receber tráfego?
    readinessProbe:
      httpGet:
        path: /actuator/health/readiness
        port: management
      initialDelaySeconds: 10
      periodSeconds: 5
      failureThreshold: 3
    
    # Para apps que demoram a inicializar
    startupProbe:
      httpGet:
        path: /actuator/health
        port: management
      initialDelaySeconds: 10
      periodSeconds: 10
      failureThreshold: 30       # Até 5 minutos para inicializar
```

---

## 10.8 Horizontal Pod Autoscaler (HPA)

### HPA Baseado em CPU

```yaml
# hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: account-service-hpa
  namespace: pix-system
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: account-service
  
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
  
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # 5 min antes de scale down
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
```

---

## 10.9 Helm Charts

### Estrutura de um Chart

```
pix-account-service/
├── Chart.yaml              # Metadata do chart
├── values.yaml             # Valores padrão
├── values-dev.yaml         # Valores para dev
├── values-prod.yaml        # Valores para prod
├── templates/
│   ├── _helpers.tpl        # Template helpers
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── ingress.yaml
│   ├── hpa.yaml
│   └── NOTES.txt           # Instruções pós-install
└── charts/                 # Dependências
```

### Chart.yaml

```yaml
apiVersion: v2
name: pix-account-service
description: Pix Account Service Helm Chart
type: application
version: 1.0.0
appVersion: "1.0.0"

dependencies:
  - name: postgresql
    version: "12.1.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
```

### values.yaml

```yaml
# values.yaml
replicaCount: 2

image:
  repository: 123456789.dkr.ecr.sa-east-1.amazonaws.com/pix-account-service
  tag: "1.0.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: api.pix.bank.com
      paths:
        - path: /accounts
          pathType: Prefix

resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1000m"

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70

env:
  SPRING_PROFILES_ACTIVE: "kubernetes"

secrets:
  DATABASE_PASSWORD: ""  # Override in values-prod.yaml
```

### Template com Helpers

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "pix-account-service.fullname" . }}
  labels:
    {{- include "pix-account-service.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "pix-account-service.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "pix-account-service.selectorLabels" . | nindent 8 }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 8080
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

### Comandos Helm

```bash
# Instalar
helm install account-service ./pix-account-service \
  -n pix-system \
  -f values-prod.yaml

# Upgrade
helm upgrade account-service ./pix-account-service \
  -n pix-system \
  -f values-prod.yaml

# Listar releases
helm list -n pix-system

# Rollback
helm rollback account-service 1 -n pix-system

# Template local (debug)
helm template account-service ./pix-account-service -f values-prod.yaml
```

---

## 10.10 Ambiente Local (Minikube)

### Instalação

```bash
# Windows (com Chocolatey)
choco install minikube

# Iniciar cluster
minikube start --driver=docker --memory=4096 --cpus=2

# Status
minikube status

# Dashboard
minikube dashboard

# Parar
minikube stop
```

### Desenvolvimento Local

```bash
# Usar Docker do Minikube
eval $(minikube docker-env)

# Build direto no Minikube
docker build -t pix-account-service:local .

# Aplicar manifests
kubectl apply -f k8s/

# Port-forward para testar
kubectl port-forward svc/account-service 8080:80 -n pix-system

# Acessar: http://localhost:8080
```

### Skaffold (Hot Reload)

```yaml
# skaffold.yaml
apiVersion: skaffold/v4beta6
kind: Config
metadata:
  name: pix-system
build:
  artifacts:
    - image: pix-account-service
      context: pix-account-service
      docker:
        dockerfile: Dockerfile
deploy:
  kubectl:
    manifests:
      - k8s/*.yaml
```

```bash
# Desenvolvimento com hot reload
skaffold dev
```

---

## 🎓 Perguntas de Entrevista

### Nível Pleno

1. **"Qual a diferença entre Pod e Container?"**
   - Pod é a menor unidade do K8s, pode ter 1+ containers
   - Containers no mesmo Pod compartilham network e storage

2. **"Para que serve um Service?"**
   - Expõe Pods na rede com IP estável
   - Load balancing entre Pods
   - Service discovery via DNS

3. **"Como você faz deploy sem downtime?"**
   - RollingUpdate strategy
   - maxSurge e maxUnavailable
   - Readiness probes

### Nível Senior

4. **"Explique liveness vs readiness probe"**
   - Liveness: aplicação travou? Restart!
   - Readiness: pronta para tráfego? Remove do Service

5. **"Como você escala aplicações automaticamente?"**
   - HPA baseado em CPU/Memory
   - Custom metrics com Prometheus
   - Behavior para controlar velocidade

---

## 📚 Recursos Adicionais

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Helm Documentation](https://helm.sh/docs/)
- [Minikube Documentation](https://minikube.sigs.k8s.io/docs/)
- [Kubernetes Patterns](https://www.redhat.com/en/resources/oreilly-kubernetes-patterns-cloud-native-apps-ebook)

---

**Próximo:** [MODULO_11_TERRAFORM.md](MODULO_11_TERRAFORM.md) - Infrastructure as Code
