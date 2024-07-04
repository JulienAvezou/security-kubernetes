# security-kubernetes
Notes from DevSecOps ch.12 Security in Kubernetes

Important to secure infra same way in Cloud as you would on-premise

K8s security in layers:
- Underlying infra
- K8s platform
- Applications

Security should be redundant to increase security at every step 

Sec best practices:
- build secure images (see docker security best practices section for image sec)
- do image scanning before pushing to registry + scan regularly in registry
- run container as non-root
- RBAC (Role Based Access Control) - authentication & authorization at cluster level and within the cluster, keep permissions as restrictive as possible mapped to namespaces. NB: users need to be created via certificates within context of K8s. For non-human users - ServiceAccount resource which uses tokens
- Defined network rules at network level (NetworkPolicy) or at service level via Service Mesh using proxies (with Istio for ex) to determine how pods can talk to each other - least access allowed rules
- mTLS between Services for encryption
- Secure Secret Data using Secret resource - enable built-in encryption using EncryptionConfiguration with AWS KMS to manage the encryption keys or 3rd party solution like HashiCorp Vault
-  Secure etcd store (stores secrets and config data from updates) - put etcd behind firewall, encrypt etcd data
-  Protect your application data - automated backup and restore
-  Configure Security Policies - Policy as Code to avoid misconfigurations by automating the validation of security config via 3rd party tools (Open Policy Agent, Kyverno etc.)

-----

# Demo
1. Provision EKS cluster
