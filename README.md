# GitOps Automation

This repository contains the GitOps configurations for managing Kubernetes clusters across multiple retail store locations. Its primary purpose is to automate ring-based deployments of applications across store clusters, allowing controlled rollout of different application versions to specific groups of stores. Rings can be defined either through cluster metadata, enabling flexible deployment strategies. The GitOps Automation tooling combines configuration data with application templates to generate store-specific Kubernetes manifests, with built-in support for rapid rollback in case of deployment issues.

## Configuration Management

### Custom Resource Definitions (CRDs)

The repository uses Custom Resource Definitions (CRDs) to define the structure and validation rules for clusters. CRDs extend the Kubernetes API by defining new, custom resource types that can be managed like built-in Kubernetes objects.

For more information on CRDs, see the [Kubernetes documentation on Custom Resources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/).

The following CRDs are defined in `config/crds.yaml`:

- **Cluster CRD**: Defines the structure for cluster configurations in `clusters.yaml`
  - Specifies name and metadata details for each retail store cluster
  - Validates cluster-specific configuration parameters

Sample configurations in these YAML files conform to their respective CRD specifications, ensuring consistent and validated configurations across the GitOps workflow.

### Cluster and Application Versioning

The repository uses GitOps Automation (goa), a powerful command-line tool that streamlines the process of managing Kubernetes configurations in a GitOps workflow. It merges configuration data with K8s templates to populate the GitOps repository efficiently.

#### Key Features

- **Configuration Management**: Merges configuration data with Kubernetes templates
- **Cluster Management**: Lists and manages Kubernetes clusters
- **Application Management**: Lists and manages applications across clusters
- **Expression Evaluation**: Supports complex expression evaluation in configurations
- **Dry Run Support**: Tests changes before applying them
- **Multi-threading**: Optimized performance for large-scale deployments
- **YAML Support**: Native YAML input/output support

### Application Templates

The `templates/` directory contains Kubernetes manifest templates that are used to deploy applications to each cluster. These templates leverage Kustomize, a powerful Kubernetes native configuration management tool, allowing multiple versions of applications to coexist and be deployed to different customers. The templates are processed by the GitOps automation process based on the `expression:` for each overlay version to generate the final manifests for each cluster.

Using Kustomize with Flux is considered a best practice in GitOps workflows, as it provides powerful capabilities like:
- Base and overlay configurations for different environments
- Strategic merge patches for customization
- ConfigMap and Secret generators
- Resource transformers for consistent modifications

For more information on using Kustomize with Flux, see the [Flux documentation on Kustomize integration](https://fluxcd.io/flux/components/kustomize/kustomization/).

Key applications include:
- **POS**: Point of Sale application for retail operations
- **Heartbeat**: Health monitoring and status reporting
- **Make-Line**: Robotic burrito bowl make line

### Deployment Rules

Applications are mapped to clusters using expressions in the `kustomization.yaml` files:

Expressions are formulas to determine which clusters apply to that expression

a token beginning with /c matches the cluster. /c/cluster.Name matches the one cluster with that name. /c/* matches all clusters.

The logical operators and, or, not are valid tokens. Parenthesis are supported for evaluation ordering.

A token beginning with /m is a metadata token. The full format is /m/key/value where key exists in the clusters metadata and the value matches. Key and value matching are case sensitive.

Examples
- `/c/*`: Deploys to all clusters
- `/m/ring/0`: Deploys to clusters with metadata `ring: 0`
- `/c/tx-austin`: Deploys to the cluster with name `tx-austin`
- Combining expressions
  - `/m/ring/0 or /m/ring/1`
  - `/m/ring/0 or /c/tx-lockhart`
  - `/m/ring/0 and /m/oven/ACME5000`
- Additional expressions can be defined for specific deployment patterns

#### Process Flow

1. The GitOps automation process reads the configuration files
2. Evaluates the expressions against cluster metadata
3. Generates the appropriate Kubernetes manifests
4. Saves the results in the `/clusters` directory
5. Each store gets its own subdirectory with its specific configuration

```mermaid
graph LR
    subgraph "Configuration"
        A[Clusters]
        B[Applications]
    end

    subgraph "Deployment"
        J[Deployment Rules Examples]
        J --> |"/c/*"| K[All Clusters]
        J --> |"/c/tx-austin"| N[Cluster by Name]
        J --> |"/m/ring/0"| L[Ring 0 Metadata]
    end

    subgraph "GitOps Automation"
        O[Configuration] --> E[Deployment Rules Evaluation]
        E --> F[Manifest Generation]
    end

    subgraph "Generated Configurations"
        F --> G[clusters/tx-austin/]
        F --> H[clusters/tx-cedar-park/]
        F --> I[clusters/tx-round-rock/]
    end

    Configuration --> O
    Deployment --> E

    style A fill:#f9f,stroke:#333,stroke-width:2px,color:#000
    style B fill:#f9f,stroke:#333,stroke-width:2px,color:#000
    style E fill:#dfd,stroke:#333,stroke-width:2px,color:#000
    style J fill:#dfd,stroke:#333,stroke-width:2px,color:#000
    style O fill:#f9f,stroke:#333,stroke-width:2px,color:#000
```

## Architecture

The following diagram illustrates how the GitOps repository is deployed to retail stores in the Austin area:

```mermaid
graph TB
    subgraph "GitOps Repository"
        A[GitOps Repo] --> B[Clusters Config]
        A --> C[Apps Config]
        B --> D[Store Clusters]
        C --> E[Applications]
    end

    subgraph "Store Clusters"
        D --> F[tx-austin]
        D --> G[tx-round-rock]
        D --> H[tx-pflugerville]
        D --> I[tx-cedar-park]
        D --> J[tx-leander]
    end

    subgraph "Applications"
        E --> K[POS System]
        E --> O[Heartbeat]
    end

    subgraph "Stores"
        P[Austin] --> F
        Q[Round Rock] --> G
        R[Pflugerville] --> H
        S[Cedar Park] --> I
        T[Leander] --> J
    end

    style A fill:#f9f,stroke:#333,stroke-width:2px,color:#000
    style F fill:#dfd,stroke:#333,stroke-width:2px,color:#000
    style G fill:#dfd,stroke:#333,stroke-width:2px,color:#000
    style H fill:#dfd,stroke:#333,stroke-width:2px,color:#000
    style I fill:#dfd,stroke:#333,stroke-width:2px,color:#000
    style J fill:#dfd,stroke:#333,stroke-width:2px,color:#000
    style P fill:#bbf,stroke:#333,stroke-width:2px,color:#000
    style Q fill:#bbf,stroke:#333,stroke-width:2px,color:#000
    style R fill:#bbf,stroke:#333,stroke-width:2px,color:#000
    style S fill:#bbf,stroke:#333,stroke-width:2px,color:#000
    style T fill:#bbf,stroke:#333,stroke-width:2px,color:#000
```

## Components

### Applications

- **POS System**: Point of Sale application for retail operations
- **Ingress Nginx**: Kubernetes ingress controller for routing traffic
- **Cert Manager**: SSL/TLS certificate management
- **Heartbeat**: Health monitoring and status reporting

### Store Clusters

Each retail store location has its own Kubernetes cluster, managed through GitOps. The clusters are named according to their location (e.g., `tx-austin`, `tx-round-rock`).

### GitOps Implementation

This repository uses Arc enabled GitOps (Flux) for GitOps implementation, which:
- Continuously monitors the repository for changes
- Automatically syncs configurations to the appropriate clusters
- Maintains the desired state of all store environments
- Provides audit trail of all changes

## Getting Started

### Prerequisites
- Kubernetes cluster
- Flux CD installed
- kubectl configured

### Deployment
1. Clone this repository
2. Configure Flux CD for your cluster
3. Apply the cluster-specific configurations
4. Monitor the deployment through Flux CD

## Contributing
1. Create a new branch for your changes
2. Make your changes
3. Submit a pull request
4. Ensure all tests pass
5. Get approval from maintainers

## Security
- Access to the repository is restricted
- All changes require review and approval

## Support
For support, please create an issue in this repository
