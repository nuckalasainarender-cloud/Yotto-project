# Multi-tenant Hosting Platform — Quick Start

## Prereqs
- Kubernetes cluster (1.25+)
- Helm 3
- DNS provider with API token (for ExternalDNS), or wildcard DNS pointing to LB
- GitHub repo + GHCR access (or DockerHub)
- GitHub Actions secrets:
  - KUBECONFIG_BASE64
  - KAFKA_BOOTSTRAP
  - DOCKER_REGISTRY creds if using DockerHub

## Install infra
1. Install ingress controller:
   `helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace`
2. Install cert-manager:
   `helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true`
3. Install ExternalDNS (configure provider):
   `helm install external-dns bitnami/external-dns --set provider=cloudflare --set cloudflare.apiToken=<TOKEN>`

## Deploy tenants
1. Create namespaces: `kubectl apply -f k8s/base/namespace-user1.yaml` (repeat for multiple namespace)
2. Build & push images or let GH Actions do it.
3. Apply tenant manifests and check ingress/cert:
   `kubectl apply -f k8s/tenant -n user1`
   `kubectl get ingress -n user1`
   `kubectl describe certificate user1-tls -n user1`

## Kafka (local)
`docker-compose -f kafka/docker-compose.yml up -d`
Create topic: `docker exec -it kafka kafka-topics.sh --create --topic websites --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1`

## Testing autoscale
Run `hey`:
`hey -n 50000 -c 200 https://user1.example.com/`
Watch `kubectl get hpa -n user1 -w` and Grafana.

