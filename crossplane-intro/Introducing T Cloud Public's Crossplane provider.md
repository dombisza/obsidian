![[Pasted image 20260617084139.png]]

# 🧩 What is Crossplane

Crossplane is an open-source control plane that extends Kubernetes to manage cloud infrastructure and services using Kubernetes APIs. It enables platform teams to provision and manage resources across providers such as AWS, Azure, GCP, **T Cloud Public** through declarative, Kubernetes-native configurations.

By turning Kubernetes into a universal control plane for infrastructure, Crossplane allows teams to define reusable platform abstractions and self-service APIs that encapsulate organizational standards, security policies, and operational best practices. Developers consume high-level resources, while Crossplane automatically provisions and manages the underlying cloud infrastructure.

By treating infrastructure as code within Kubernetes and integrating naturally with GitOps workflows, Crossplane helps automate deployments, improve consistency, reduce operational overhead, and simplify multi-cloud operations.

- 💡 Manage cloud services with:
    - ⎈ Kubernetes-style APIs
    - 🕹️ **Reconciliation** loops:
	    - Drives from observed to desired state automaticly
    - 💚 **GitOps** tools/workflows:
	    - `helm`
	    - `helmfile`
	    - `argocd`
- ⎈ Installed as a **control-plane/operator** inside a Kubernetes cluster
	- ⚡️ Runs on:
	    - `kind`
	    - `CCE`
	    - `OpenShift`
	    - Any Kubernetes flavor actually
- ☁️ Enables management of **T Cloud Public** services like:
	- `RDS`
	- `OBS`
	- `VPC`
	- `ECS`
	- `CCE`
	- many **more**


## 💡 Let Crossplane automate cloud infra

**Crossplane** brings cloud resource management into **Kubernetes**, enabling declarative provisioning and automated reconciliation of **T Cloud Public** services like *RDS*, *CCE*, *OBS*, *ECS*, etc...
![[Pasted image 20260223083055.png]]

# 👀 Terraform vs Crossplane operation  
  
| Aspect                  | Terraform-Based Operations                                       | Crossplane-Based Operations                                                       |
| ----------------------- | ---------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| **Primary Model**       | Infrastructure as Code (IaC) using Terraform hcl configurations  | Kubernetes-native infrastructure management using Custom Resources (CRDs)         |
| **Control Plane**       | External Terraform CLI, Terraform Cloud, or automation pipelines | Kubernetes acts as the control plane                                              |
| **State Management**    | Requires separate state files (local or remote backend)          | State stored in Kubernetes' etcd                                                  |
| **Resource Lifecycle**  | CI/CD pipelines or manual runs                                   | Continuously reconciled by Kubernetes controllers                                 |
| **Drift Detection**     | Periodic `terraform plan` required                               | Automatic and continuous reconciliation                                           |
| **Operational Model**   | Push-based execution                                             | Pull-based reconciliation                                                         |
| **Multi-Cloud Support** | Mature and extensive                                             | Supported through Crossplane providers                                            |
| **GitOps Integration**  | Indirect, usually through CI/CD runners                          | Native fit with GitOps tools like ArgoCD                                          |
| **Day-2 Operations**    | Changes require Terraform runs                                   | Continuous management and automated remediation                                   |
| **Learning Curve**      | Easier for infrastructure teams                                  | Easier for Kubernetes-centric platform teams, but can be more complex             |
| **Best Fit**            | Traditional infrastructure automation, broad cloud coverage      | Internal developer platforms, Kubernetes-first organizations, GitOps environments |

# ⎈ Crossplane providers

- [Providers](https://docs.crossplane.io/latest/packages/providers/) are responsible for all aspects of connecting to non-Kubernetes resources, like cloud APIs:
    - Define APIs -> [ManagedResource](https://docs.crossplane.io/latest/managed-resources/managed-resources/)
    - Authentication
    - Implement controllers
    - Manage external infrastructure resources
- Most providers are built on top of **Terraform providers** with upjet

🚀 Example `ManagedResource` to deploy an `OBS` bucket:
```yaml
apiVersion: obs.opentelekomcloud.m.crossplane.io/v1alpha1
kind: Bucket
metadata:
  annotations:
    meta.upbound.io/example-id: obs/v1alpha1/bucket
  labels:
    testing.upbound.io/example-name: b
  name: b
spec:
  forProvider:
    acl: private
    versioning: true
    region: eu-de
    bucket: crossplane-test
    tags:
      Env: Test
      foo: bar
      managed: xplane
```

- `forProvider` section:
    - Similar to Terraform configuration
    - Single [source of truth](https://docs.crossplane.io/latest/managed-resources/managed-resources/#forprovider)
    - **Desired** state definition
    - Protected against deletion by default
## 🕹️ provider-opentelekomcloud

- Provider built using **Upjet tooling**
- Upjet [generates](https://github.com/crossplane/upjet-provider-template) Crossplane providers from Terraform providers
- All Terraform-supported services are configurable
- Some services still lack dynamic value assignment support: [tracker](https://github.com/opentelekomcloud/provider-opentelekomcloud/issues/7)

# 🕹️ManagedResources (MR)

A _managed resource_ (`MR`) represents an external service in a Provider. When users create a new managed resource, the Provider reacts by creating an external resource inside the Provider’s environment.
## Managed resource fields

- The Provider defines the group, kind and version of a managed resource. The Provider also define the available settings of a managed resource.

### Group, Kind and Version

- Each managed resource is a unique API endpoint with their own group, kind and version.
- For example the [T Cloud Provider](LINK) defines the `Bucket` (OBS) kind from the group `obs.opentelekomcloud.m.crossplane.io`

```yaml
apiVersion: obs.opentelekomcloud.m.crossplane.io/v1alpha1
kind: Bucket
```
**TODO note: namespaced and clustered**
### forProvider

- The `spec.forProvider` of a managed resource maps to the parameters of the external resource.
- For example, when creating a `Bucket` instance, the Provider supports defining the `region`, `acl`, `versioning` and other fields here.

```yaml
spec:
  forProvider:
    acl: private
    versioning: true
    region: eu-de
    bucket: crossplane-test
```

# 🛠️ Installing and Configuring the Provider

## Install Crossplane core
Start by creating a namespace for Crossplane:

```bash
kubectl create namespace crossplane-system
```

Next, add the Crossplane Helm repository and update it:

```bash
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
```

Finally, install Crossplane using Helm:

```bash
helm install crossplane --namespace crossplane-system crossplane-stable/crossplane 
```

After installation, verify that Crossplane is running correctly:

```bash
kubectl -n crossplane-system wait --for=condition=Available deployment --all --timeout=5m
```

## Install the T Cloud Public Provider
```bash
cat <<EOF | kubectl apply -f -
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-opentelekomcloud
spec:
  package: xpkg.upbound.io/opentelekomcloud/provider-opentelekomcloud:v0.7.0
EOF
```

`ClusterProviderConfig` setup with secret:

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: provider-secret
  namespace: crossplane-system
type: Opaque
stringData:
  credentials: |
    {
      "user_name": "admin",
      "password": "t0ps3cr3t11",
      "auth_url": "https://iam.eu-de.otc.t-systems.com/v3",
      "domain_name": "OTCxxxxx",
      "tenant_name": "eu-de_project",
      "swauth": "false",
      "allow_reauth": "true",
      "max_retries": "2",
      "max_backoff_retries": "6",
      "backoff_retry_timeout": "60",
      "insecure": "false"
    }
---
apiVersion: opentelekomcloud.m.crossplane.io/v1beta1
kind: ClusterProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      name: provider-secret
      namespace: crossplane-system
      key: credentials
EOF
```

# 🚀 Composite Resources
A composite resource, or XR, represents a set of Kubernetes resources as a single Kubernetes object. Crossplane creates composite resources when users access a custom API, defined in the CompositeResourceDefinition.

![[Pasted image 20260624122146.png]]

- **Multi-cloud engineering** – Enables composing infrastructure APIs that work consistently across multiple cloud providers.
- **Standardized cloud resources** – Allows platform teams to define approved infrastructure patterns, ensuring consistency, security, and compliance across the organization.
- **Self-service infrastructure** – Gives developers simple, application-focused APIs to provision infrastructure without needing deep expertise in cloud platforms.
- **Infrastructure abstraction** – Hides cloud-provider-specific complexity behind higher-level APIs that align with business and platform requirements.
- **Reusable infrastructure patterns** – Packages common architectures (such as databases, Kubernetes clusters, or application environments) into reusable building blocks that can be deployed repeatedly and consistently.
Demo Ideas:
- keycloak
- standardized rds -- most likely
- APP api: deployment+obs+dns+ingress static stuff


## 🧩Multi-cloud Platform Engineering

- Enables [building](https://docs.crossplane.io/latest/composition/) **custom infrastructure APIs**
- No need to write controllers manually

- Use case
    - Team `A` uses **AWS**
    - Team `B` uses **T Cloud**
    
	- ❌ Without Crossplane
	    - Multiple Terraform modules OR one complex module
	    - Developers must understand deployment logic
	- ✅ With Crossplane
	    - Same API for **AWS** and **T Cloud**
		- Platform team abstracts provider complexity
	    - Developers consume simplified, unified APIs

🚀 Example multi-cloud custom API
```yaml
#NOTE not an actual working example
#Implementation dependent

# DEPLOY TO AWS
apiVersion: platform.example.com/v1alpha1
kind: Bucket
metadata:
  name: app-bucket-aws
  namespace: team-a
spec:
  region: us-east-1
  acl: private
  versioning: true
  bucketName: team-a-demo-bucket
  compositionSelector:
    matchLabels:
      provider: aws
---
# DEPLOY TO TCLOUD
apiVersion: platform.example.com/v1alpha1
kind: Bucket
metadata:
  name: app-bucket-tcloud
  namespace: team-b
spec:
  region: eu-de
  acl: private
  versioning: true
  bucketName: team-b-demo-bucket
  compositionSelector:
    matchLabels:
      provider: tcloud
```
[# Managing Resources Across Multiple Cloud Providers with Crossplane](https://oneuptime.com/blog/post/2026-02-09-crossplane-multiple-clouds/view)

# ✨Demo checklist
### 📦 Basic MR
- [ ] Run through install
> [!NOTE]- Install. 
> [install provider](https://github.com/opentelekomcloud/provider-opentelekomcloud?tab=readme-ov-file#install-provider-opentelekomcloud)


- [ ] Show `ManagedResouce:buckets`
> [!bucket]- bucket.yaml
> ```yaml
> apiVersion: obs.opentelekomcloud.m.crossplane.io/v1alpha1
> kind: Bucket
> metadata:
>   annotations:
>     meta.upbound.io/example-id: obs/v1alpha1/bucket
>   labels:
>     testing.upbound.io/example-name: b
>   name: b
>   namespace: test
> spec:
>   forProvider:
>     acl: private
>     bucket: crossplane-test999999
>     tags:
>       Env: Test
>       foo: bar
>       managed: xplane
> ```


- [ ] Deploy an `OBS` `bucket`
> [!NOTE]- Deploy 
> `kubectl apply -f bucket.yaml`


- [ ] Show reconcile
> [!NOTE]- Note
> Delete TAGS, after 1 minute it will reconcile.


### 🚢 MR with dynamic values
- [ ] Show `ManagedResource:rds`
> [!bucket]- rds.yaml
> ```yaml
> apiVersion: v1
> kind: Secret
> metadata:
>   name: example-rds-secret
>   namespace: test
> type: Opaque
> data:
>   example-rds-key: UG9zdGdyZXNAIzIwMjQ=
>
> ---
> apiVersion: vpc.opentelekomcloud.m.crossplane.io/v1alpha1
> kind: VpcV1
> metadata:
>   annotations:
>     meta.upbound.io/example-id: vpc/v1alpha1/v1
>   labels:
>     testing.upbound.io/example-name: sample-rds-instance
>   name: sample-rds-instance
>   namespace: test
> spec:
>   forProvider:
>     cidr: "192.168.0.0/16"
>     name: crossplane-rds-vpc
>     tags:
>       managed-by: crossplane
>
> ---
> apiVersion: vpc.opentelekomcloud.m.crossplane.io/v1alpha1
> kind: SubnetV1
> metadata:
>   annotations:
>     meta.upbound.io/example-id: vpc/v1alpha1/subnetv1
>   labels:
>     testing.upbound.io/example-name: sample-rds-instance
>   name: sample-rds-instance
>   namespace: test
> spec:
>   forProvider:
>     cidr: "192.168.0.0/16"
>     gatewayIp: "192.168.0.1"
>     name: crossplane-rds-subnet
>     ntpAddresses: "10.100.0.33,10.100.0.34"
>     vpcIdSelector:
>       matchLabels:
>         testing.upbound.io/example-name: sample-rds-instance
>     tags:
>       managed-by: crossplane
>
> ---
> apiVersion: compute.opentelekomcloud.m.crossplane.io/v1alpha1
> kind: SecgroupV2
> metadata:
>   annotations:
>     meta.upbound.io/example-id: compute/v1alpha1/secgroupv2
>   labels:
>     testing.upbound.io/example-name: sample-rds-instance
>   name: sample-rds-instance
>   namespace: test
> spec:
>   forProvider:
>     description: crossplane security group
>     name: crossplane-rds-sg
>     rule:
>       - cidr: 0.0.0.0/0
>         fromPort: 8635
>         ipProtocol: tcp
>         toPort: 8635
>       - cidr: 0.0.0.0/0
>         fromPort: 8080
>         ipProtocol: tcp
>         toPort: 8080
>       - cidr: 0.0.0.0/0
>         fromPort: 443
>         ipProtocol: tcp
>         toPort: 443
>
> ---
> apiVersion: vpc.opentelekomcloud.m.crossplane.io/v1alpha1
> kind: EIPV1
> metadata:
>   annotations:
>     meta.upbound.io/example-id: vpc/v1alpha1/eipv1
>   labels:
>     testing.upbound.io/example-name: sample-rds-instance
>   name: sample-rds-instance
>   namespace: test
> spec:
>   forProvider:
>     bandwidth:
>       - chargeMode: traffic
>         name: crossplane-rds-instance
>         shareType: PER
>         size: 8
>     publicip:
>       - type: 5_bgp
>
> ---
> apiVersion: rds.opentelekomcloud.m.crossplane.io/v1alpha1
> kind: InstanceV3
> metadata:
>   annotations:
>     meta.upbound.io/example-id: rds/v1alpha1/instancev3
>   labels:
>     testing.upbound.io/example-name: sample-rds-instance
>   name: sample-rds-instance
>   namespace: test
> spec:
>   forProvider:
>     availabilityZone:
>       - eu-de-03
>     backupStrategy:
>       - keepDays: 1
>         startTime: 08:00-09:00
>     db:
>       - passwordSecretRef:
>           key: example-rds-key
>           name: example-rds-secret
>         port: 8635
>         type: PostgreSQL
>         version: "15"
>     flavor: rds.pg.n1.large.2
>     name: crossplane-rds-instance
>     computeSecurityGroupIdSelector:
>       matchLabels:
>         testing.upbound.io/example-name: sample-rds-instance
>     subnetIdSelector:
>       matchLabels:
>         testing.upbound.io/example-name: sample-rds-instance
>     tags:
>       managed-by: crossplane
>     volume:
>       - size: 100
>         type: CLOUDSSD
>     vpcIdSelector:
>       matchLabels:
>         testing.upbound.io/example-name: sample-rds-instance
>     publicIpsSelector:
>       matchLabels:
>         testing.upbound.io/example-name: sample-rds-instance
> ```


- [ ] Highlight dynamic value assignment
> [!NOTE]- Info 
> IDs from other resources(dependencies) can be populated dynamically using the Kubernetes labeling system. For example, you can assign the label `testing.upbound.io/example-name: sample-rds-instance` to the `VpcV1` resource and then use the `vpcIdSelector` field to dynamically reference the VPC ID.


- [ ] Deploy `RDS`
> [!NOTE]- Deploy 
> `kubectl apply -f rds.yaml`


