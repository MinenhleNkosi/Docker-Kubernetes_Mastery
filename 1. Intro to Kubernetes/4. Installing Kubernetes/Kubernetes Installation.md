# Kubernetes Configuration — Choosing How to Set Up Your Cluster

### Not One Size Fits All

Just like you wouldn't use a Formula 1 race car to do your grocery shopping, you wouldn't set up a full production-grade Kubernetes cluster just to learn the basics. Kubernetes offers several configuration types — from a single laptop setup all the way to a fully redundant, multi-region production cluster.

The key variables are:

- How many **control plane nodes** do you need?
- How many **worker nodes** do you need?
- Is **etcd stacked** (on the control plane) or **etcd external** (on its own hosts)?

```mermaid
graph TD
    START[🤔 What kind of cluster\ndo you need?] --> Q1{Purpose?}

    Q1 -->|Learning / Dev / Testing| SIMPLE[Simpler configs\nSingle node or basic setups]
    Q1 -->|Production| COMPLEX[Advanced configs\nHA with multiple nodes]

    style SIMPLE fill:#2471a3,color:#fff
    style COMPLEX fill:#1a3c5e,color:#fff
```

## The 5 Configuration Types

### 1. All-in-One Single-Node

**Everything** — control plane AND worker components — runs on a **single machine**. Think of it as a **studio apartment** — kitchen, bedroom, and living room all in one space. Convenient for one person, not liveable for a family.

```mermaid
graph TD
    subgraph SINGLE[🖥️ Single Node]
        CP[🧠 Control Plane\napiserver · scheduler · controller-manager · etcd]
        W[💪 Worker\nkubelet · kube-proxy · runtime]
        PODS[📦 Your Pods]
        CP --> W --> PODS
    end

    style SINGLE fill:#2471a3,color:#fff
```

- **Great for**: learning, local dev, testing
- **Not for**: production

### 2. Single Control Plane + Multi-Worker (Stacked etcd)

One control plane node (with etcd running on it), managing multiple worker nodes. Like a manager with a team — **one brain, multiple hands**.

```mermaid
graph TD
    CP[🧠 Single Control Plane\n+ stacked etcd]

    CP --> W1[🖥️ Worker Node 1]
    CP --> W2[🖥️ Worker Node 2]
    CP --> W3[🖥️ Worker Node 3]

    style CP fill:#1a3c5e,color:#fff
    style W1 fill:#1a6b3c,color:#fff
    style W2 fill:#1a6b3c,color:#fff
    style W3 fill:#1a6b3c,color:#fff
```

- Better than single-node — workloads are distributed.
- Control plane is still a single point of failure.

### 3. Single Control Plane + External etcd + Multi-Worker

Same as above, but etcd is moved off the control plane onto its own dedicated host. This protects the data store from being affected if the control plane has issues.

```mermaid
graph TD
    CP[🧠 Single Control Plane\nNo etcd here]
    ETCD[🗄️ External etcd\nDedicated Host]

    CP <-->|Reads & writes state| ETCD
    CP --> W1[🖥️ Worker Node 1]
    CP --> W2[🖥️ Worker Node 2]
    CP --> W3[🖥️ Worker Node 3]

    style CP fill:#1a3c5e,color:#fff
    style ETCD fill:#7d6608,color:#fff
    style W1 fill:#1a6b3c,color:#fff
    style W2 fill:#1a6b3c,color:#fff
    style W3 fill:#1a6b3c,color:#fff
```

- etcd is protected from control plane failures.
- Control plane itself is still a single point of failure.

### 4. Multi-Control Plane + Stacked etcd + Multi-Worker (HA)

Now we're talking proper production. Multiple control plane nodes — each with its own stacked etcd instance — forming a HA cluster. If one control plane node fails, the others keep the cluster running.

```mermaid
graph TD
    LB[⚖️ Load Balancer]

    LB --> CP1[🧠 Control Plane 1\n+ etcd ✅]
    LB --> CP2[🧠 Control Plane 2\n+ etcd ✅]
    LB --> CP3[🧠 Control Plane 3\n+ etcd ✅]

    CP1 <-->|etcd sync| CP2
    CP2 <-->|etcd sync| CP3

    CP1 --> W1[🖥️ Worker 1]
    CP2 --> W2[🖥️ Worker 2]
    CP3 --> W3[🖥️ Worker 3]

    style CP1 fill:#1a3c5e,color:#fff
    style CP2 fill:#1a3c5e,color:#fff
    style CP3 fill:#1a3c5e,color:#fff
    style LB fill:#e67e22,color:#fff
    style W1 fill:#1a6b3c,color:#fff
    style W2 fill:#1a6b3c,color:#fff
    style W3 fill:#1a6b3c,color:#fff
```

- Control plane is fault-tolerant
- etcd is replicated across control plane nodes
- etcd and control plane still share the same nodes — a node failure affects both

### 5. Multi-Control Plane + External etcd + Multi-Worker (Full HA)

The gold standard for production. Control plane and etcd are fully decoupled and both run in HA mode on their own dedicated hosts. This is the most resilient, most complex, and most expensive configuration.

```mermaid
graph TD
    LB[⚖️ Load Balancer]

    LB --> CP1[🧠 Control Plane 1]
    LB --> CP2[🧠 Control Plane 2]
    LB --> CP3[🧠 Control Plane 3]

    CP1 --> ETCD_CLUSTER
    CP2 --> ETCD_CLUSTER
    CP3 --> ETCD_CLUSTER

    subgraph ETCD_CLUSTER[🗄️ External HA etcd Cluster]
        E1[etcd Host 1]
        E2[etcd Host 2]
        E3[etcd Host 3]
    end

    CP1 --> W1[🖥️ Worker 1]
    CP2 --> W2[🖥️ Worker 2]
    CP3 --> W3[🖥️ Worker 3]

    style CP1 fill:#1a3c5e,color:#fff
    style CP2 fill:#1a3c5e,color:#fff
    style CP3 fill:#1a3c5e,color:#fff
    style ETCD_CLUSTER fill:#7d6608,color:#fff
    style LB fill:#e67e22,color:#fff
    style W1 fill:#1a6b3c,color:#fff
    style W2 fill:#1a6b3c,color:#fff
    style W3 fill:#1a6b3c,color:#fff
```

- Maximum resilience — control plane and etcd failures are independent
- Recommended for production at scale
- Highest hardware cost and operational complexity

## All 5 Configurations — Side by Side

```mermaid
graph LR
    subgraph CONFIG1[1 - All-in-One]
        N1[🖥️ One node\nEverything here]
    end

    subgraph CONFIG2[2 - Single CP + Workers]
        CP2A[🧠 CP + etcd]
        W2A[🖥️🖥️🖥️ Workers]
        CP2A --> W2A
    end

    subgraph CONFIG3[3 - Single CP + Ext etcd]
        CP3A[🧠 CP]
        E3A[🗄️ etcd]
        W3A[🖥️🖥️🖥️ Workers]
        CP3A --- E3A
        CP3A --> W3A
    end

    subgraph CONFIG4[4 - HA CP + Stacked etcd]
        CP4A[🧠🧠🧠 HA CPs\n each with etcd]
        W4A[🖥️🖥️🖥️ Workers]
        CP4A --> W4A
    end

    subgraph CONFIG5[5 - Full HA]
        CP5A[🧠🧠🧠 HA CPs]
        E5A[🗄️🗄️🗄️ HA etcd]
        W5A[🖥️🖥️🖥️ Workers]
        CP5A --- E5A
        CP5A --> W5A
    end
```

| Config | Control Plane       | etcd        | Workers   | Use Case            |
| ------ | ------------------- | ----------- | --------- | ------------------- |
| 1      | Single (all-in-one) | Stacked     | Same node | Learning only       |
| 2      | Single              | Stacked     | Multiple  | Dev / Small teams   |
| 3      | Single              | External    | Multiple  | Better dev setup    |
| 4      | HA (multiple)       | Stacked     | Multiple  | Production          |
| 5      | HA (multiple)       | External HA | Multiple  | Production at scale |

## Infrastructure Decisions

Before installing, you need to answer three key questions:

```mermaid
mindmap
  root((Infrastructure\nDecisions))
    Where will it run?
      Bare metal servers
      Public cloud\nAWS · Azure · GCP
      Private cloud
      Hybrid cloud
    Which OS?
      Linux\nRed Hat-based\nDebian-based
      Windows\nWorker nodes only
    Which networking CNI?
      Flannel
      Calico
      Cilium
      Weave
      Others
```

These choices affect performance, cost, compatibility, and how you manage the cluster long-term. The environment you're targeting — learning vs production — is usually the clearest guide.

## Local Learning Clusters — Getting Started Fast

For learning and development, you don't need a full cluster. These tools let you spin up a Kubernetes environment on your own laptop or workstation in minutes:

```mermaid
graph TD
    subgraph LOCAL[🖥️ Local Learning Tools]
        MK[🟢 Minikube\nSingle & multi-node\nBest for beginners\n⭐ Used in this course]
        KIND[🐳 Kind\nK8s inside Docker containers\nGreat for multi-node testing]
        DD[🐋 Docker Desktop\nBuilt-in K8s\nFor Docker users]
        PD[📦 Podman Desktop\nK8s integration\nFor Podman users]
        MK8S[⚡ MicroK8s\nFrom Canonical\nLocal + cloud]
        K3S[🪶 K3S\nLightweight K8s\nEdge · IoT · Local\nCNCF project]
    end

    style MK fill:#1a6b3c,color:#fff
    style KIND fill:#2471a3,color:#fff
    style DD fill:#2980b9,color:#fff
    style PD fill:#6e2f8b,color:#fff
    style MK8S fill:#e67e22,color:#fff
    style K3S fill:#7d6608,color:#fff
```

_**This course uses Minikube** — it's the most beginner-friendly, highly flexible, and built with features specifically designed to simplify your interaction with Kubernetes during learning_.

| Tool               | Best For                        | Nodes                |
| ------------------ | ------------------------------- | -------------------- |
| **Minikube**       | Beginners, this course          | Single & multi       |
| **Kind**           | Multi-node testing in CI        | Multi (Docker-based) |
| **Docker Desktop** | Docker users wanting quick K8s  | Single               |
| **Podman Desktop** | Podman users                    | Single               |
| **MicroK8s**       | Lightweight local + small cloud | Single & multi       |
| **K3S**            | Edge, IoT, resource-constrained | Single & multi       |

## Production Cluster Deployment Tools

When you're ready for a real production environment, these tools handle the heavy lifting of bootstrapping a full cluster:

### Kubeadm

![kubeadm](image.png)

**kubeadm** is a first-class citizen of the Kubernetes ecosystem. It is a secure and recommended method to bootstrap a multi-node production ready Highly Available Kubernetes cluster, on-premises or in the cloud. kubeadm can also bootstrap a single-node cluster for learning. It has a set of building blocks to set up the cluster, but it is easily extendable to add more features. Please note that kubeadm does not support the provisioning of hosts - they should be provisioned separately with a tool of our choice.

### Kubespray

![kubespray](image1.png)

**kubespray** (formerly known as kargo) allows us to install Highly Available production ready Kubernetes clusters on AWS, GCP, Azure, OpenStack, vSphere, or bare metal. kubespray is based on Ansible, and is available on most Linux distributions.

### Kops

![kops](image2.png)

**kops** enables us to create, upgrade, and maintain production-grade, Highly Available Kubernetes clusters from the command line. It can provision the required infrastructure as well. Currently, AWS and GCE are officially supported. Support for DigitalOcean and OpenStack is in beta, while Azure is in alpha support, and other platforms are planned for the future.

```mermaid
graph TD
    subgraph PROD[🏭 Production Deployment Tools]
        KA[🔧 kubeadm\nFirst-class Kubernetes tool\nSecure · HA · On-premises or cloud\nDoes NOT provision hosts]
        KS[⚙️ kubespray\nAnsible-based\nAWS · GCP · Azure · OpenStack\nvSphere · Bare metal\nMost Linux distros]
        KO[☁️ kops\nCLI-based cluster management\nCreate · Upgrade · Maintain\nProvisions infrastructure too\nAWS + GCE officially supported]
    end

    style KA fill:#1a3c5e,color:#fff
    style KS fill:#1a6b3c,color:#fff
    style KO fill:#2471a3,color:#fff
```

| Tool          | Provisions Hosts?                      | Platforms                                             | Best For                                   |
| ------------- | -------------------------------------- | ----------------------------------------------------- | ------------------------------------------ |
| **kubeadm**   | ❌ No — you handle hosts               | On-prem, any cloud                                    | Standard HA cluster setup                  |
| **kubespray** | ❌ No — uses Ansible on existing hosts | AWS, GCP, Azure, OpenStack, vSphere, bare metal       | Ansible-based teams, wide platform support |
| **kops**      | ✅ Yes — provisions infra too          | AWS (full), GCE (full), DigitalOcean/OpenStack (beta) | Full lifecycle management from CLI         |

## Kubernetes on Windows

Windows plays a major role in enterprise environments, and Kubernetes has progressively added Windows support over the years. Here's what you need to know:

```mermaid
graph TD
    subgraph K8S_WINDOWS[☸️ Kubernetes Windows Support]
        WW[🪟 Windows Worker Nodes\n✅ Supported since v1.14\nWindows Server 2019 & 2022]
        WCP[🧠 Windows Control Plane Nodes\n❌ NOT supported\nControl plane must run Linux]
    end

    subgraph HYBRID[🔀 Hybrid Cluster]
        LCP[🐧 Linux Control Plane]
        LW[🐧 Linux Worker Node\nRuns Linux containers]
        WN[🪟 Windows Worker Node\nRuns Windows containers]

        LCP --> LW
        LCP --> WN
    end

    style WW fill:#1a6b3c,color:#fff
    style WCP fill:#8B0000,color:#fff
    style LCP fill:#1a3c5e,color:#fff
    style LW fill:#2471a3,color:#fff
    style WN fill:#e67e22,color:#fff
```

Key points to remember:

| Aspect                     | Detail                                                     |
| -------------------------- | ---------------------------------------------------------- |
| Windows worker nodes       | ✅ Supported since Kubernetes v1.14                        |
| Windows control plane      | ❌ Not supported — must use Linux                          |
| Supported Windows versions | Windows Server 2019 and Windows Server 2022                |
| Cluster type               | Can be Windows-only or hybrid (Linux + Windows nodes)      |
| Workload scheduling        | You must configure workloads to target the correct OS node |

When running a hybrid cluster, you are responsible for making sure Linux containers are scheduled on Linux nodes and Windows containers are scheduled on Windows nodes. Kubernetes won't do this automatically — you configure it through node selectors or affinity rules.

## The Full Installation Decision Tree

```mermaid
graph TD
    START[🚦 Starting Out with Kubernetes] --> PURPOSE{What's the purpose?}

    PURPOSE -->|Learning / Dev| LOCAL[💻 Local Cluster\nMinikube · Kind · K3S]
    PURPOSE -->|Production| PROD_Q{How critical\nis availability?}

    PROD_Q -->|Moderate| SIMPLE_PROD[🔧 kubeadm\nSingle or basic HA]
    PROD_Q -->|High| HA_Q{Manage infra\nyourself?}

    HA_Q -->|Yes - Ansible| KS2[⚙️ kubespray]
    HA_Q -->|Yes - CLI| KO2[☁️ kops]
    HA_Q -->|No - use cloud| KAAS[☁️ Managed KaaS\nEKS · AKS · GKE]

    style LOCAL fill:#2471a3,color:#fff
    style SIMPLE_PROD fill:#1a5e38,color:#fff
    style KS2 fill:#1a6b3c,color:#fff
    style KO2 fill:#1a6b3c,color:#fff
    style KAAS fill:#e67e22,color:#fff
```

**Key Takeaway**: Kubernetes configuration is not one-size-fits-all. Start with **Minikube** to learn, graduate to **kubeadm** or **kubespray** for production, and aim for a **multi-control-plane with external etcd** setup when reliability truly matters. Always match your cluster design to your actual needs — over-engineering early is as costly as under-engineering later.
