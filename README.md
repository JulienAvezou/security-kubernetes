# security-kubernetes
Notes from DevSecOps ch.12 onwards for Security topics in Kubernetes

## Concepts

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

### Demo/ setting up EKS cluster
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

-----

## Access management in K8s

Least privilege rule

Authentication + Authorization layers of security in K8s

Restrict access to devs by namespace - RBAC (Role Based Access Control)
-> RBAC makes use of Role component bound to namespace with specific permissions
-> to attach User or Group to Role usd RoleBinding component
-> for admins doing cluster wide ops, can use ClusterRole & ClusterRoleBinding
-> K8s doesn't manage human users natively, so can use certificates or static token file or 3rd party (ie. LDAP) to manage users via API Server for authentication
-> For application users (non human) - ServiceAccount component to represent user and then use same Role, RoleBinding, ClusterRole, ClusterRoleBinding

checking API access: auth can-i subcommand

For Managed K8s (ie. EKS via mapping AWS IAM users & roles to K8s roles) these config can be abstracted 
How does this mapping work?
- everything goes through release pipeline and shouldn`t be modifiable outside of the release process
- admins only have READ ONLY access via ClusterRole
- devs have Role with limited access to namespace with READ ONLY permission

### Demo/ configure K8s access in terraform
1. create aws-auth ConfigMap - connector between AWS resources & K8s components
![Capture d’écran 2024-07-05 à 17 47 33](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/97083947-e41b-4b46-b2cb-8c3833ce5ffe)

2. configure aws IAM roles
![Capture d’écran 2024-07-05 à 18 14 07](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/3f10f5b3-0d32-4815-a344-c5f4433fb248)
![Capture d’écran 2024-07-05 à 18 14 15](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/43eb9b40-6a95-4ccf-9a34-fa332bdd3e37)

3. configure mapping between AWS roles & k8s roles
![Capture d’écran 2024-07-05 à 18 14 22](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/e7f1eba2-3cb4-421d-aef8-8175020387a9)

4. Create AWS users and set arn in tfvars
![Capture d’écran 2024-07-05 à 19 58 09](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/19b3843d-0890-494d-a878-85615516a381)
![Capture d’écran 2024-07-05 à 20 03 12](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/7bfeb56e-e1e6-4a0a-bcdd-004b1b71bf38)

5. Setup k8s resources
- setup kubernetes provider
![Capture d’écran 2024-07-05 à 20 07 57](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/ec7e6557-3fa0-462f-9abb-b36ff0a57dfa)

- connect to eks cluster with kubernetes provider (needs aws cli installed locally + aws user to execute the cmd)
![Capture d’écran 2024-07-05 à 20 12 34](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/59321da9-559c-4c2b-9eb1-7ef5d430353f)

- configure k8s namespace
![Capture d’écran 2024-07-05 à 20 28 03](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/2dfe9c90-9685-4b27-8647-6017a18f7200)

- configure k8s Role
![Capture d’écran 2024-07-05 à 20 28 09](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/c7034b70-4542-439e-82e1-22f6d37041c1)

- configure k8s RoleBinding
![Capture d’écran 2024-07-06 à 11 18 36](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/5f14d9bc-b7fe-4622-8c5c-4993b50f8d26)

- configure k8s ClusterRole & ClusterRoleBinding
![Capture d’écran 2024-07-06 à 11 22 36](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/7ce6e70a-3492-4431-927c-8e15ac31c200)

6. Provision resources with tf + check that resources are provisioned + test access
![Capture d’écran 2024-07-06 à 15 35 38](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/f0b0d7cb-224a-47fb-ad92-f0d89306ec90)
![Capture d’écran 2024-07-06 à 15 37 55](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/e1416a38-c8aa-422f-910c-f5e77b36dc66)
![Capture d’écran 2024-07-06 à 15 38 56](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/ef2cf2a6-e78c-48a3-b298-92672646bd20)
- test permissions with k8-admin user, while not having assumed the role to read access for Kubernetes resources
  aws sts get-caller-identity:
![Capture d’écran 2024-07-06 à 15 48 36](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/e0b6c9db-dc37-4d9c-a9cd-1c333b3c47ad)
![Capture d’écran 2024-07-06 à 15 49 32](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/eece26d2-87cb-4819-9f3f-7609ed03c7d8)
  then assume the role with that user:
![Capture d’écran 2024-07-06 à 16 36 38](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/42fc56ad-900b-45c7-8f1c-49b4ea229d84)
![Capture d’écran 2024-07-06 à 16 06 10](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/ddddc163-a3fa-4034-9273-6af15b734a6c)
![Capture d’écran 2024-07-06 à 16 06 25](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/69324c1f-2a08-415f-89dc-ad4b0b0ba116)
- test permissions with k8-developer user, while not having assumed the role to read access for Kubernetes resources
![Capture d’écran 2024-07-06 à 16 48 19](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/ed759318-e6c5-416a-bb4d-0739518ad2f0)
![Capture d’écran 2024-07-06 à 16 48 08](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/e4f93562-999a-487c-be45-6803728a729b)
then assume the role with that user:
![Capture d’écran 2024-07-06 à 16 50 18](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/440e8040-0bfa-4457-b025-153915829251)
![Capture d’écran 2024-07-06 à 16 49 45](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/80d09f73-a626-4907-bd3e-c1e5f68925e7)
![Capture d’écran 2024-07-06 à 16 52 09](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/5dc7465a-da46-47a8-a8dc-028979587743)
![Capture d’écran 2024-07-06 à 16 50 48](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/fafc89a5-8588-4c21-a247-e14b2d299f1f)

-----

## Secure IaC Pipeline for EKS provisioning
SSM limited to only certain aws resources (ie. EC2, RDS), and also needed to run on EC2 instance using Gitlab runner
-> more secure and generic approach = AWS STS (Security Token Service)

Set up a Trust Policy between AWS & external entity, in this case Gitlab, using Gitlab OIDC (OpenID Connect)
Create a Role that makes use of this web trust policy (AssumeRoleWithWebIdentity), and store this role in Gitlab variables for referencing in the pipeline

![Capture d’écran 2024-07-06 à 18 22 22](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/185c398d-a998-4b3a-b581-3d6e0aea0bfc)
![Capture d’écran 2024-07-06 à 18 24 02](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/fb5422c9-f3cc-41a6-9f40-2221326f8abe)
![Capture d’écran 2024-07-06 à 18 25 10](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/5a71c8e6-e4a7-4b7c-8022-70f23f994e57)
![Capture d’écran 2024-07-06 à 18 28 01](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/46251337-ff26-4543-a8c8-74a780106a84)

Gitlab makes request to AWS by sending a OIDC token that will be verified by AWS using the Role setup with web identity (ie. proving that Gitlab is the principal that should be trusted by AWS to assume that Role)
![Capture d’écran 2024-07-06 à 19 17 22](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/080b9c44-29de-4600-8c14-b3e16ecfbfbf)
![Capture d’écran 2024-07-06 à 19 25 39](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/13bfafb5-3bdb-4532-bdc9-c8b42d74c543)

To summarise the workflow above in steps:
1. Configure GitLab Identity Provider on AWS
2. Configure Trustpolicy for IAM Role
3. Configure ID token generation in GitLab CI job
4. Configure Role Assumption in CI job

5. Configure remote terraform state
![Capture d’écran 2024-07-07 à 15 36 58](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/07cbbed5-1e1d-4893-859d-0b1610064887)
![Capture d’écran 2024-07-07 à 15 38 34](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/5cdaaa1f-66a9-4866-9bc7-653f4b28e8d0)

6. Add terraform config to pipeline
![Capture d’écran 2024-07-07 à 15 58 51](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/e6a22230-288a-4182-9913-6c957bd33d88)
![Capture d’écran 2024-07-07 à 15 58 46](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/8255a1cc-8cc0-4b5e-a271-019879021c21)
![Capture d’écran 2024-07-07 à 15 57 37](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/5b644c9b-e5d2-441a-920c-ad6bccb940f0)
![Capture d’écran 2024-07-07 à 15 58 01](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/1e9d8daf-6fbf-4843-99d2-3ac86d15e497)
![Capture d’écran 2024-07-07 à 15 59 49](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/1f13a0c2-5668-4794-92bf-8e1c3b5bdb7f)
![Capture d’écran 2024-07-07 à 16 00 14](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/5d51c26d-32e6-468b-ad71-9d77067fb345)
![Capture d’écran 2024-07-07 à 16 14 15](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/1cbb38d8-f072-4fcb-addc-747c429edfd1)
![Capture d’écran 2024-07-07 à 16 14 33](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/e4564085-8402-4044-b108-e3c781d998f8)

------

## EKS Blueprints
