

## TITLE: Introducing T Cloud Public's Crossplane provider
![[crossplane-image.png]]

## Summary

[Crossplane](https://www.crossplane.io/) brings cloud resource management into **Kubernetes**, enabling declarative provisioning and automated reconciliation of **T Cloud Public** services like _RDS_, _CCE_, _OBS_, _ECS_... These cloud resources in Crossplane are called [Managed Resources](https://docs.crossplane.io/latest/managed-resources/managed-resources/), which map Kubernetes custom resources(CRD's) to external infrastructure components.
It extends the Kubernetes API to manage infrastructure the same way teams manage applications, using familiar YAML, GitOps workflows, and CI/CD pipelines.  
With [Composite Resources](https://docs.crossplane.io/latest/composition/composite-resources/), platform teams can create their own higher-level APIs that abstract complex cloud architectures into simple, reusable building blocks.  
Crossplane enables self-service infrastructure provisioning with built-in policy control, standardization, and multi-environment consistency.  
Its reconciliation engine continuously monitors and corrects drift, ensuring infrastructure remains aligned with the declared desired state.  
By combining Kubernetes-native operations with reusable platform APIs, Crossplane accelerates cloud automation while improving governance, scalability, and developer experience.

## Target groups
Primary target audiences

- **Platform Engineers / Platform Teams** 
- **DevOps Engineers** 
- **Cloud Architects** 
- **Site Reliability Engineers** 
- **Infrastructure / Operations Teams** 

Secondary target audiences

- **Developers / Application Teams** 
- **CTOs / Engineering Managers** 
- **Security & Compliance Teams** 

## Agenda
- Introducing Crossplane
- Crossplane basics and fundamentals
- Crossplane Providers
- Installing Crossplane and the provider
- ManagedResources demonstration
- Basic composite resources and self-service (if time allows)
- Q&A

## Level of complexity 
Intermediate (kubernetes experience recommended)

## About the speakers
**Szabolcs Dombi – Expert Solution Engineer**  
Szabolcs is an active maintainer of the **T Cloud Public Crossplane Provider** and has extensive experience delivering Kubernetes-based solutions across the T Cloud ecosystem. Over the years, he has contributed to a wide range of projects, including the CCE backend, images, cloud consulting, and platform integrations.

**Anton Sidelnikov – Principal Cloud Engineer**  
Anton is the lead developer behind many well-known open-source projects within the **T Cloud Public** ecosystem, including the Crossplane provider. He originally started the provider's development and played the key role in designing and implementing its core functionality. With strong expertise in cloud platforms, systems design, and open-source engineering, he continues to drive innovation across the T Cloud Public ecosystem. He is joining the the Q&A section to answer all your questions about the Crossplane provider.