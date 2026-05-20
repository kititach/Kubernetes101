# Kubernetes 101 Lab Repository

Welcome to the Kubernetes 101 Labs! This repository contains a structured series of Kubernetes configurations and manifests designed to help you learn Kubernetes concepts step by step.

## Repository Structure

The repository is organized into the following modules:

- **00-setup**: Initial environment setup and cluster configurations.
- **01-basics**: Basic Kubernetes resources, including simple Pods.
- **02-workloads**: Managing application workloads with Deployments and ReplicaSets.
- **03-networking**: Services, Ingress, and cluster networking (ClusterIP, NodePort).
- **04-storage**: Persistent Volumes (PV) and Persistent Volume Claims (PVC).
- **05-config**: Configuration management using ConfigMaps and Secrets.
- **06-scheduling**: Controlling pod placement using NodeSelectors, Taints, and Tolerations.
- **07-security**: Security contexts, ServiceAccounts, and RBAC configurations.
- **08-helm**: Introduction to package management in Kubernetes using Helm charts.

## Additional Resources

- **[K8S-GUIDE.md](./K8S-GUIDE.md)**: Comprehensive guide and references for the concepts covered in this lab.
- **[STATUS.md](./STATUS.md)**: Current lab status and progress tracking.

## Getting Started

1. Clone this repository to your local machine.
2. Ensure you have a running Kubernetes cluster (e.g., Minikube, kind, or a managed cloud K8s).
3. Navigate into any module directory to explore the manifest files (`.yaml`).
4. Apply the configurations using `kubectl apply -f <filename.yaml>`.

---
*Happy Learning!*
