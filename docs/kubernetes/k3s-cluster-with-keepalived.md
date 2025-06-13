# k3s Cluster with keepalived

# Setting up Debian
```bash
sudo apt update
sudo apt upgrade -y
sudo apt install git tmux curl openssh-server qemu-guest-agent
```

# Install and configure keepalived
In my case, I have 3 nodes here, all using `enp6s18.10` as the interface. On all nodes, edit `/etc/keepalived/keepalived.conf`.

## "Prinary" node
```bash
vrrp_instance VI_4 {
	state MASTER
	interface enp6s18.10
	virtual_router_id 51
	priority 100
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass 2639
	}
	virtual_ipaddress {
		10.0.10.30/24
	}
}
vrrp_instance VI_6 {
	state MASTER
	interface enp6s18.10
	virtual_router_id 52
	priority 100
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass 2639
	}
	virtual_ipaddress {
		2a0b:6b86:b32:10::30/64
	}
}
```

## Second node

```bash
vrrp_instance VI_4 {
	state BACKUP
	interface enp6s18.10
	virtual_router_id 51
	priority 90
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass 2639
	}
	virtual_ipaddress {
		10.0.10.30/24
	}
}
vrrp_instance VI_6 {
	state BACKUP
	interface enp6s18.10
	virtual_router_id 52
	priority 90
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass 2639
	}
	virtual_ipaddress {
		2a0b:6b86:b32:10::30/64
	}
}
```

## Third node
```bash
vrrp_instance VI_4 {
	state BACKUP
	interface enp6s18.10
	virtual_router_id 51
	priority 80
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass 2639
	}
	virtual_ipaddress {
		10.0.10.30/24
	}
}
vrrp_instance VI_6 {
	state BACKUP
	interface enp6s18.10
	virtual_router_id 52
	priority 80
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass 2639
	}
	virtual_ipaddress {
		2a0b:6b86:b32:10::30/64
	}
}
```

Once these are edited on all nodes, on each node, run `sudo service keepalived restart`.

# Install k3s
## First "master" server
```bash
curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" sh -s - server \
    --tls-san=10.0.10.30 \
    --cluster-init
```

## Other servers
Get the token from the first server in the cluster via `cat /var/lib/rancher/k3s/server/node-token`. On the other instances, just like the first one, run the following:
```bash
curl -sfL https://get.k3s.io | K3S_TOKEN=[token] K3S_KUBECONFIG_MODE="644" sh -s - server \
    --server https://10.0.10.31:6443 \
    --tls-san=10.0.10.30
```

# Edit `.bashrc`
```bash
# Kubernetes stuff
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
export KUBE_EDITOR=nano
```
followed by `source .bashrc`.

# Install Helm
On the primary server, or any server you want to install Helm on,
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

# Install cert-manager
```bash
helm repo add jetstack https://charts.jetstack.io --force-update

helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.17.2 \
  --set crds.enabled=true
```

# Configure cert-manager to issue certs
After each of these, issue `kubectl apply -f [filename]`.

## Cloudflare API key secret `cloudflare-secret.yaml`
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-token-secret
  namespace: cert-manager
type: Opaque
stringData:
  api-token: [Cloudflare API token]
```

## Traefik TLS Store `default-tlsstore.yaml`
```yaml
apiVersion: traefik.io/v1alpha1
kind: TLSStore
metadata:
  name: default
  namespace: kube-system
spec:
  defaultCertificate:
    secretName: wildcard-example-com-tls
```

## ClusterIssuer `letsencrypt-prod.yaml`
```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: [LetsEncrypt email]
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - dns01:
        cloudflare:
          apiTokenSecretRef:
            name: cloudflare-api-token-secret
            key: api-token
```

## Wildcard certificate `wildcard-example-com.yaml`
```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: wildcard-example-com
  namespace: kube-system
spec:
  secretName: wildcard-example-com-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - '*.example.com'
```
