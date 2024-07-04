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
- terraform init
<img width="602" alt="Capture d’écran 2024-07-04 à 21 08 42" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/e5690d7d-8b64-4c74-a6ae-3b708a14ab2f">

- terraform plan
<img width="614" alt="Capture d’écran 2024-07-04 à 21 11 22" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/ec52df48-9166-485e-8a67-d11c911c19eb">

- terraform apply
<img width="703" alt="Capture d’écran 2024-07-04 à 21 28 29" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/e9ee4e4f-abde-440a-b6e1-61e0ca683864">

- check setup + connection to cluster
<img width="655" alt="Capture d’écran 2024-07-04 à 21 41 46" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/7c5bf9ef-e5b3-4c61-aed4-de600e37824a">
<img width="1074" alt="Capture d’écran 2024-07-04 à 21 36 38" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/4f143e75-e933-422d-a13d-f84836826215">
<img width="1053" alt="Capture d’écran 2024-07-04 à 21 36 14" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/34709cae-c8a3-4c17-81a4-3b05012be7d2">

- terraform destroy (cleanup)
<img width="421" alt="Capture d’écran 2024-07-04 à 21 50 09" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/2d43a2a7-c53f-401f-bded-0ca0a45603d8">

user that created cluster gets automatically added to system:masters group

get current user info logged in with: aws sts get-caller-identity

improve security by upgrading version
<img width="228" alt="Capture d’écran 2024-07-04 à 21 31 41" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/3a993b4e-624b-43fd-b2c1-349fbd6491c4">
