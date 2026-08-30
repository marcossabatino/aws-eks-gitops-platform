# Initial Platform Design

## Status

Proposed

## Purpose

Build a cost-conscious AWS EKS platform for hands-on DevOps and
Platform Engineering learning. This design is not production-grade.

## Requirements

- Use a sandbox AWS account.
- Deploy resources in `us-east-1`.
- Use one shared EKS cluster.
- Create three Kubernetes namespaces:
  - `development`
  - `staging`
  - `production`
- Treat `production` as a simulated environment.
- Manage infrastructure with Terraform.
- Deliver Kubernetes resources through GitOps.

## Compute

- Use one small On-Demand managed node group for Karpenter and critical
  system components.
- Keep one system node as the sandbox baseline.
- Use Karpenter to provision application nodes.
- Use Graviton (`arm64`) application nodes.
- Use Spot capacity for application nodes.
- Maintain three baseline application nodes.
- Allow application capacity to grow to six nodes.
- Use multiple compatible Graviton instance types when possible.
- Validate that all application and add-on images support `arm64`.

The expected cluster size is four to seven EC2 nodes:

- One On-Demand system node.
- Three to six Spot application nodes.

## Networking

- Deploy the VPC across three Availability Zones.
- Create one private node subnet in each Availability Zone.
- Do not assign public IP addresses to EKS nodes.
- Distribute the three baseline application nodes across the three
  Availability Zones when possible.
- Use one NAT gateway to reduce sandbox costs.
- Enable both public and private EKS API endpoints.
- Restrict the public EKS API endpoint to approved CIDR ranges.

## Environment isolation

Each environment must have:

- A dedicated Kubernetes namespace.
- Separate configuration and secrets.
- Resource quotas and limit ranges.
- Namespace-level RBAC.
- Network policies.
- Independent GitOps deployment configuration.
- Workload resource requests and limits.
- Readiness, liveness, and startup probes where appropriate.

Namespaces provide logical isolation only. A cluster failure or
cluster-wide change can affect all environments.

## Application access

For Version 1:

- Use Kubernetes `ClusterIP` services.
- Access applications with `kubectl port-forward`.
- Do not install AWS Load Balancer Controller.
- Do not create public Ingress resources or load balancers.

Example:

```bash
kubectl port-forward \
  --namespace development \
  service/example-app \
  8080:80
```