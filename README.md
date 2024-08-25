## Create rancher manager

1. Start colima with a decent spec

```
colima start --cpu 5 --memory 16
```

2. Have a cluster working with ingress nginx.

```bash
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  image: kindest/node:v1.28.13
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

3. Install ingress-nginx

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

4. wait a bit until ingress is healthy

5. install Rancher helm chart

```bash
$ kubectl create namespace cattle-system

## Generate ssl certs using minica
$ minica --domains 'localhost'
$ k create secret tls tls-rancher-ingress -n cattle-system --cert=localhost/cert.pem --key=localhost/key.pem
$ k -n cattle-system create secret generic tls-ca \
  --from-file=cacerts.pem=./minica.pem

## Install rancher
$ helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
$ helm repo update

$ helm upgrade --install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --set hostname=localhost \
  --set bootstrapPassword=admin \
  --set replicas=1 \
  --set ingress.tls.source=secret

# watch for rancher pods
$ k -n cattle-system rollout status deploy/rancher

```

6. Deploy grafana
```bash
kubectl apply -f ./grafana.yaml
```
