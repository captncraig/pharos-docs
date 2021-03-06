## Host System Requirements

### Supported Host Operating Systems

- Debian 9
    - architectures: x86-64, arm64
    - container runtimes: docker (x86-64, arm64), cri-o (x86-64)
- CentOS 7.4 - 7.5
    - architectures: x86-64
    - container runtimes: docker, cri-o
- Redhat Enterprise Linux 7.4 - 7.6
    - architectures: x86-64
    - container runtimes: docker, cri-o
- Ubuntu 16.04
    - architectures: x86-64, arm64
    - container runtimes: docker (x86-64, arm64), cri-o (x86-64)
- Ubuntu 18.04
    - architectures: x86-64, arm64
    - container runtimes: docker (x86-64, arm64), cri-o (x86-64)

### Other Requirements

- hosts need SSH access
- a user with passwordless sudo permission (`echo "$USER ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$USER`)
- host operating system specific requirements
    - CentOS 7.4 - 7.5
        - `extras` repository enabled for docker package
    - Redhat Enterprise Linux 7.4 - 7.6
        - `rhel-7-server-extras-rpms` repository enabled for docker package
    - Ubuntu 16.04
        - `universe` (xenial-updates) apt repository enabled for docker.io package
    - Ubuntu 18.04
        - `universe` apt repository enabled for docker.io package

### Required Open Ports

The following ports are used by the `pharos` management tool, as well as between nodes in the same cluster. These ports are all authenticated, and can safely be left open for public access if desired.

| Protocol    | Port        | Service         | Direction               | Notes
|-------------|-------------|-----------------|-------------------------|-------
| TCP         | 22          | SSH             | CLI => Host             | authenticated management channel for `pharos` operations using SSH keys
| TCP         | 2379        | etcd clients    | Master <=> Master       | authenticated etcd client API using TLS client certs
| TCP         | 2380        | etcd peers      | Master <=> Master       | authenticated etcd peers API using TLS client certs
| TCP         | 6443        | kube-apiserver  | Worker, CLI => Master   | authenticated kube API using kube TLS client certs, ServiceAccount tokens with RBAC
| TCP         | 6783        | weave control   | Host <=> Host           | authenticated Weave peer control connections using the shared weave secret
| UDP         | 6783        | weave dataplane | Host <=> Host           | authenticated Weave `sleeve` fallback using the shared weave secret
| UDP         | 6784        | weave dataplane | Host <=> Host (trusted) | unauthenticated Weave `fastdp` (VXLAN), only used for peers on `network.trusted_subnets` networks
| ESP (IPSec) |             | weave dataplane | Host <=> Host           | authenticated Weave `fastdp` (IPsec encapsulated UDP port 6784 VXLAN) using IPSec SAs established over the control channel
| TCP         | 10250       | kubelet         | Master, Worker => Host  | authenticated kubelet API for the master node `kube-apiserver` (and `heapster`/`metrics-server` addons) using TLS client certs

If using the `ingress-nginx` addon, then TCP ports 80/443 on the worker nodes (or nodes matching `addons.ingress-nginx.node_selector`) must also be opened for public access.

### Monitoring Ports

The following ports serve unauthenticated monitoring/debugging information, and are either disabled, limited to localhost-only or only expose relatively harmless information.

| Protocol    | Port        | Service               | Hosts   | Status          | Notes
|-------------|-------------|-----------------------|---------|-----------------|-------
| TCP         | 6781        | weave-npc metrics     | All     | **OPEN**        | unauthenticated `/metrics`
| TCP         | 6782        | weave metrics         | All     | **OPEN**        | unauthenticated `/metrics`
| TCP         | 10248       | kubelet               | All     | localhost-only  | ?
| TCP         | 10249       | kube-proxy metrics    | All     | localhost-only  | ?
| TCP         | 10251       | kube-scheduler        | Master  | localhost-only  | ?
| TCP         | 10252       | kube-controller       | Master  | localhost-only  | ?
| TCP         | 10255       | kubelet read-only     | All     | *disabled*      | unauthenticated read-only `/pods`, various stats metrics
| TCP         | 10256       | kube-proxy healthz    | All     | **OPEN**        | unauthenticated `/healthz`
| TCP         | 18080       | ingress-nginx status  | Workers | **OPEN**        | unauthenticated `/healthz`, `/nginx_status` and default backend

These ports should be restricted from external access to prevent information leaks.

### Restricted Ports

The following restricted services are only accessible via localhost the nodes, and must not be exposed to any untrusted access.

| Protocol    | Port        | Service               | Hosts   | Status          | Notes
|-------------|-------------|-----------------------|---------|-----------------|------
| TCP         | 6784        | weave control         | All     | localhost-only  | unauthenticated weave control API
