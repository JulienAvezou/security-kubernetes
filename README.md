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
- Build secure images (see docker security best practices section for image sec)
- Do image scanning before pushing to registry + scan regularly in registry
- Run container as non-root
- RBAC (Role Based Access Control) - authentication & authorization at cluster level and within the cluster, keep permissions as restrictive as possible mapped to namespaces. NB: users need to be created via certificates within context of K8s. For non-human users - ServiceAccount resource which uses tokens
- Defined network rules at network level (NetworkPolicy) or at service level via Service Mesh using proxies (with Istio for ex) to determine how pods can talk to each other - least access allowed rules
- mTLS between Services for encryption
- Secure Secret Data using Secret resource - enable built-in encryption using EncryptionConfiguration with AWS KMS to manage the encryption keys or 3rd party solution like HashiCorp Vault
- Secure etcd store (stores secrets and config data from updates) - put etcd behind firewall, encrypt etcd data
- Protect your application data - automated backup and restore
- Configure Security Policies - Policy as Code to avoid misconfigurations by automating the validation of security config via 3rd party tools (Open Policy Agent, Kyverno etc.)

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
Some examples of add-ons via EKS Blueprints: auto scaling; metrics server, load balancer controller
![Capture d’écran 2024-07-07 à 16 47 02](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/567b591c-b967-472f-ba7c-1d1b53e5f5aa)

Add-ons use Helm in background:
- need to define Helm provider in terraform and authenticate
![Capture d’écran 2024-07-07 à 16 55 45](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/6e0f8737-a92c-43b9-a66b-2e2b914794f1)

For troubleshooting:
1. Find original Helm Chart, EKS Blueprints are using
2. In Helm Chart, search in values.yaml for config options
3. Override default config inside EKS Blueprint terraform module
![Capture d’écran 2024-07-07 à 17 24 40](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/f2acc359-a586-4a7a-9eb0-f7b66a97107c)

------

## ArgoCD

ArgoCD -> gitops continuous delivery tool used for Kubernetes
pull-based: 
1. Deploy ArgoCD in K8s cluster
2. Configure ArgoCD to track git repo
3. ArgoCD monitors for any changes & applies automatically

Unlike Jenkins or other CI/CD tools:
- no need to install and setup tools, like kubectl
- no need to configure access to K8s and cloud platforms (like AWS)
- no visibility of deployment status

best practice to have separate git repo for app source doe & app configuration to avoid triggering full CI pipeline if only app configuration changes (ie. K8s manifest file) but not app source code

ArgoCD supports:
- K8s YAML files
- Helm charts
- Kustomize

Splitting CI & CD thanks to ArgoCD

Benefits of using ArgoCD setup based on GitOps principles:
- git as single source of truth: ArgoCD will always try to sync with the desired state in the git repo containing the app configuration (can also configure to not sync automatically and alert on manual changes)
- versioning via change history
- easy rollback
- cluster disaster recovery
- K8s access control using git repo
- No need to give external access to K8s cluster (like Jenkins)
- ArgoCD uses existing K8s functionalities (like using etcd for storage), improving visibility in the cluster
- Same ArgoCD instance can be used to sync a fleet of K8s clusters distributed across regions for the same environment
- For different environments can use overlays with kustomize 

### Demo
Pre-step:
- create new repo in Gitlab + setup empty folder structure overlays/dev
![Capture d’écran 2024-07-09 à 17 07 01](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/4ca16252-1829-485c-8cf3-24cf4686eeb0)

1. Deploy Argo CD in K8s cluster
- configure Argo CD via helm chart instance
![Capture d’écran 2024-07-09 à 15 41 36](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/22231ff6-9e87-455b-869e-c3c0d36de31f)

- configure K8s secret to connect Argo CD to git repo
![Capture d’écran 2024-07-09 à 15 41 44](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/e305cda5-9ee0-480e-9b0a-e344ccff1057)
![Capture d’écran 2024-07-09 à 15 42 05](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/d6564889-3b08-4e32-942a-f5efc314d248)

- give way for admin to access CLuster via port forwarding
![Capture d’écran 2024-07-09 à 15 42 14](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/7948b03c-9123-43d1-bdbc-ba3bac0d702b)

- configure CRD resource for Argo CD application
![Capture d’écran 2024-07-09 à 15 42 31](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/9650a3ac-a6ab-4ffe-abc6-9ec962990a18)

- add stage in pipeline to deploy the Argo CD application
![Capture d’écran 2024-07-09 à 17 05 53](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/42b6e535-97f6-4e56-9a31-50b339b043ed)


2. Connect Argo to Gitops repo
- generate developer token from gitops repo and save credentials in settings of infra repo as variables + also the url of the gitops repo
![Capture d’écran 2024-07-09 à 17 05 26](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/af5314b6-395e-4b89-9ada-696c2579415d)

- run pipeline that deploys Argo CD

- once pipeline has run successfully, check that Argo CD Application is deployed and running
  assume role with admin user
  connect to cluster with kubectl
![Capture d’écran 2024-07-09 à 18 35 26](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/1d3fd185-c100-4051-ad57-26c916f31caa)
  portforward with kubectl to connect to Argo CD UI on localhost
![Capture d’écran 2024-07-09 à 18 58 38](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/d0ed75f0-70f9-426a-8d7d-3f96a7f6ea60)
 get secret to login to Argo CD UI
![Capture d’écran 2024-07-09 à 18 49 46](https://github.com/JulienAvezou/security-kubernetes/assets/62488871/e9f950bc-5634-4020-94fb-52138958333e)
login to Argo CD UI and check state & connection to the gitops repo
<img width="1049" alt="Capture d’écran 2024-07-11 à 19 33 55" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/cb2a5b4e-60d2-47a1-a63a-a1c6e0d67db5">
<img width="1020" alt="Capture d’écran 2024-07-11 à 19 33 36" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/f584dabc-bfa7-4bdd-8da2-27cadd4dae43">
<img width="585" alt="Capture d’écran 2024-07-11 à 19 33 17" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/67720471-2d3c-4ab8-8ed5-7802ce7cb44c">

3. Write K8s manifest files
make use of Kustomize to define base and then add overlays on top of manifest -> ie. similar apps across multiple environments
<img width="831" alt="Capture d’écran 2024-07-11 à 20 09 05" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/5d5041c4-f3af-4c4f-aa9c-b00ca6a650e3">

need one kustomization file per folder wherever used
<img width="389" alt="Capture d’écran 2024-07-11 à 19 56 40" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/c942d552-b192-41cd-8f46-92b3961188bc">
<img width="707" alt="Capture d’écran 2024-07-11 à 19 59 45" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/bd63321f-fd72-4aa9-9458-5efc88a4361c">

also allows to dynamically customise variables such as image names and tags
<img width="596" alt="Capture d’écran 2024-07-11 à 20 05 25" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/757161dd-464d-499c-bf54-5fadd1d58dff">

create gitops pipeline to increment version of manifest files via centralized kustomize file using yq tool to update YAML files, this will trigger a deployment automatically only if trigger by another pipeline:
<img width="1167" alt="Capture d’écran 2024-07-11 à 21 47 35" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/2b34b1da-61f2-469a-9b7d-ddfd3c05ea7c">
For security, every repo should have its own credentials for access (create access token per project)

create the CI pipeline that triggers the gitops pipeline:
<img width="689" alt="Capture d’écran 2024-07-11 à 22 20 56" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/d84c9924-3ab7-465c-8000-4de82751603f">

5. Argo deploys application in cluster
<img width="831" alt="Capture d’écran 2024-07-11 à 20 09 05" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/232008a9-b4fe-41e1-9ec5-9b96ac787ddb">
<img width="1136" alt="Capture d’écran 2024-07-11 à 20 09 42" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/ec88f9f1-cea2-4dcf-8966-d848b15ea69e">
<img width="1069" alt="Capture d’écran 2024-07-11 à 20 09 55" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/e6b2ced8-4997-46b9-bf45-30c0633e097c">
<img width="1371" alt="Capture d’écran 2024-07-11 à 20 27 03" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/5f43bc01-55ea-45a5-900f-1141fad87d59">

Final E2E CI/CD flow:
<img width="1005" alt="Capture d’écran 2024-07-11 à 22 24 05" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/972d2075-ad68-4434-8381-a8c1d1bb8196">
<img width="745" alt="Capture d’écran 2024-07-11 à 22 24 18" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/fcbb1d6e-1a20-4d48-8edf-9bad9bbab96c">
<img width="351" alt="Capture d’écran 2024-07-11 à 22 24 45" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/5d1feb0a-26e8-4617-8ff7-3e0e0f62896e">
<img width="992" alt="Capture d’écran 2024-07-11 à 22 26 01" src="https://github.com/JulienAvezou/security-kubernetes/assets/62488871/24bfe75f-bf60-4df3-b834-226f36a4e94c">

------

## Policy as Code

- validate manifests before deployment based on policies set by an organisation to enforce best practices on top of RBAC
- implemented by Open Policy Agent (OPA) or Gatekeeper, OPA is more general purpose & flexible, Gatekeeper is built on top of OPA for K8s specifically making it simpler to use
- policy language is called Rego

How Gatekeeper works?
- controller manager & audit installed in K8s cluster
- on each API request to CRUD K8s resource in cluster, an admission validation webhook is triggered with Gatekeeper
- Gatekeeper applies several CRD in K8s cluster including: Constraint Template (sets the desired state) CRD & Constraint (objects of desired state) CRD
- can use Gatekeeper library to find pre-defined Constraint Templates

### Demo

1. install Gatekeeper in K8s cluster within infra gitops repo
<img width="666" alt="Capture d’écran 2024-07-12 à 17 27 14" src="https://github.com/user-attachments/assets/3d02e5a1-7c23-4f62-a3cd-71b1439dd0f3">

2. create Constraint Templates & Constraints in gitops repo -> separate infra from policies

can find pre-defined templates in Gatekeeper library
<img width="896" alt="Capture d’écran 2024-07-12 à 18 14 29" src="https://github.com/user-attachments/assets/9d6319ed-9e9e-44c0-a75b-e55b11d26d9e">

<img width="996" alt="Capture d’écran 2024-07-13 à 11 32 31" src="https://github.com/user-attachments/assets/e1e0d08f-e53d-4967-8256-4af7d4a7e634">

<img width="623" alt="Capture d’écran 2024-07-13 à 11 41 45" src="https://github.com/user-attachments/assets/669d38c6-e21d-4fc0-8d5c-2e3abcaddc7d">
<img width="698" alt="Capture d’écran 2024-07-13 à 11 41 37" src="https://github.com/user-attachments/assets/e1508234-1545-490f-a6bc-83a242fe7292">

3. Create Argo CD application to deploy the OPA resources
![Capture d’écran 2024-07-13 à 11 59 00](https://github.com/user-attachments/assets/9cbe7d25-38dd-4212-b03c-bd5c9c48388d)
![Capture d’écran 2024-07-13 à 11 59 08](https://github.com/user-attachments/assets/bc19e777-1305-41df-bb74-2814b07d4ae3)

NB: you should deploy the Constraint Template before deploying the Constraint, to avoid race conditions

4. Try adding another policy constraint, such as prevent privileged containers from being deployed
<img width="1250" alt="Capture d’écran 2024-07-13 à 12 40 02" src="https://github.com/user-attachments/assets/7f63b8c5-d707-4b39-a6bc-5d14e7a308c4">
<img width="451" alt="Capture d’écran 2024-07-13 à 12 40 12" src="https://github.com/user-attachments/assets/dd96dbdc-a555-4773-a7a4-15f5ad65fc2e">

------

## Secrets management

-> Secret Object K8s uses base64 which is not secure

Alternatives:
- could reference secret as env var from pipeline however CI/CD are not secret stores, increases risk of being exposed
- store locally, however the local device can be compromised
- **secrets manager is the best option**

Secrets Manager providers:
Hashicorp Vault, AWS Secrets Manager etc.
- encrypted at rest & in transit
- secrets centrally stored
- fine-grained access control
- audit & compliance - detailed logs of access

Vault provides 2 specific features:
- dynamic secrets with short expiration period, using a secrets engine for each tool under the hood in a storage backend via authentication methods (trusted 3rd party identities) using path routing - every client has own short lived token
- encrypt as service

AWS Secrets Manager uses KMS for encryption secrets & IAM for access control & auditing via CloudTrail

AWS Secrets Manager doesn't require operational effort, unlike Vault 

External Secrets Operator K8s component allows to connect to any external secrets manager, via following CRDs:
- Cluster Secret Store - allows to connect with external secrets manager
- External Secret - references external secret and creates native K8s secret

### Demo

1. Install External Secrets Controller in EKS cluster
![Capture d’écran 2024-07-15 à 15 46 13](https://github.com/user-attachments/assets/2fbecb13-8eb7-4790-b6da-c3e590bbabc5)

2. Create secret in AWS Secrets Manager
<img width="1326" alt="Capture d’écran 2024-07-15 à 15 51 04" src="https://github.com/user-attachments/assets/80d8b585-dcfe-44c7-9556-4ca3a59433cf">

3. Create IAM role
![Capture d’écran 2024-07-15 à 16 04 23](https://github.com/user-attachments/assets/36f97053-9232-49b9-9480-2c241fc20168)

4. Create Service Account to map to IAM role
![Capture d’écran 2024-07-15 à 16 07 55](https://github.com/user-attachments/assets/1eeb765b-940c-4d05-9538-551f1798758f)

5. Create ClusterSecretStore in gitops manifest file
<img width="667" alt="Capture d’écran 2024-07-15 à 16 28 45" src="https://github.com/user-attachments/assets/2c6578f0-2b88-41c4-9f10-3fa4405c0ad1">

6. Create ExternalSecret in gitops manifest file
<img width="376" alt="Capture d’écran 2024-07-15 à 16 52 42" src="https://github.com/user-attachments/assets/b7b24c8e-77bd-430b-ab16-c5fcb3fa61f2">

7. Use secret in microservices app
<img width="635" alt="Capture d’écran 2024-07-15 à 16 55 33" src="https://github.com/user-attachments/assets/cfad74be-ce27-4a4a-b1f2-ab9e619811b6">
