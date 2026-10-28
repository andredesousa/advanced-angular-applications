# Web Infrastructure, Operations and Security

## Overview

Web infrastructure is the foundation that supports websites and web applications on the internet. It consists of servers, networks, databases, software, and security systems that work together to store, process, and deliver web content to users. In essence, web infrastructure provides the foundation that enables websites and web applications to operate efficiently, securely, and continuously for users around the world. Modern web infrastructure often relies on cloud computing, content delivery networks (CDNs), load balancers, monitoring tools, and cybersecurity measures to ensure reliability, scalability, performance, and protection against threats.

For Angular applications, infrastructure provisioning means preparing and configuring the resources required to host and run the application reliably. This includes setting up web servers, networking, and security controls so the application can be deployed safely and reached by users. At the same time, observability gives teams the ability to monitor, measure, and understand application behavior in real time. Also, security must be treated as an ongoing engineering practice, not a final checklist item.

This chapter focuses on the infrastructure decisions that matter most for DevOps teams, namely, hosting, traffic routing, observability, and operational security. It also introduces practical tools such as [Docker](https://www.docker.com/), [Terraform](https://developer.hashicorp.com/terraform), [Prometheus](https://prometheus.io/), [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/), [Grafana](https://grafana.com/), [Jaeger](https://www.jaegertracing.io/), and others.

## Cloud Providers

Cloud providers deliver on-demand infrastructure and platform services over the internet. Instead of buying and operating physical servers, networking equipment, storage arrays, and data-center facilities directly, teams rent those capabilities from providers such as [Amazon Web Services](https://aws.amazon.com/), [Google Cloud](https://cloud.google.com/), and [Microsoft Azure](https://azure.microsoft.com/). These platforms expose building blocks such as virtual machines, object storage, managed databases, load balancers, DNS, CDN services, observability tooling, and identity management through web consoles, APIs, and infrastructure-as-code workflows.

Cloud strategy also depends on the deployment model. In a **public cloud**, infrastructure is owned and operated by a third-party provider and shared across customers through tenant isolation. Public cloud is the default model for AWS, Azure, and Google Cloud and is usually the fastest path for hosting Angular applications. In a **private cloud**, cloud-style infrastructure is dedicated to a single organization, either in its own data center or in an isolated hosted environment, which can be useful for strict compliance, data residency, or hardware control requirements. In a **hybrid cloud**, an organization combines private infrastructure with public cloud services, for example by serving the Angular frontend from a public CDN while backend systems or sensitive databases remain in a private environment. Hybrid models are common during migration programs and in regulated industries where teams need cloud elasticity without moving every workload into a public platform.

For Angular and other web applications, cloud providers are commonly used to host static assets, run containerized workloads, terminate TLS, distribute traffic globally, and connect the frontend to managed backend services. A simple deployment might upload the build output to object storage and place a CDN in front of it, while a more advanced setup can run the application in containers, integrate with [WAF](https://en.wikipedia.org/wiki/Web_Application_Firewall) and [DDoS](https://en.wikipedia.org/wiki/Denial-of-service_attack) protection, and stream logs and metrics into a managed monitoring stack. The main advantage of cloud providers is that teams can provision capacity quickly, automate deployments, and scale traffic handling without owning the underlying hardware.

## Cloud Hosting

### Static Files Hosting

Serving an Angular application as static files is the simplest and most cost-effective hosting model available. The Angular `ng build` command produces a folder of HTML, JavaScript, CSS, and assets with no server-side runtime dependency. Those files can be stored in an object storage service and distributed through a CDN without managing any compute infrastructure. A recommended caching strategy for Angular deployments is to apply a long `Cache-Control: max-age` to versioned assets with hashed filenames while keeping the `index.html` entry point on `Cache-Control: no-cache` so that new deployments are picked up immediately.

**Amazon CloudFront**

On AWS, static hosting combines [Amazon S3](https://aws.amazon.com/s3/) for object storage with [Amazon CloudFront](https://aws.amazon.com/cloudfront/) as the global CDN layer. The typical setup uploads the Angular build output to an S3 bucket, associates that bucket as a CloudFront origin, configures custom error responses so that 403 and 404 responses from S3 are rewritten to `index.html` with an HTTP 200 status for SPA routing, and attaches an SSL/TLS certificate from [AWS Certificate Manager](https://aws.amazon.com/certificate-manager/) to serve the application over HTTPS on a custom domain. Traffic reaches the nearest CloudFront edge location, where cached responses are served directly to users.

**Azure Static Web Apps**

On Azure, [Azure Static Web Apps](https://azure.microsoft.com/products/app-service/static) is a fully managed service that combines static asset hosting with a globally distributed CDN, SSL certificates, and custom domain configuration. It includes a built-in [GitHub Actions](https://github.com/features/actions) or [Azure DevOps](https://azure.microsoft.com/products/devops) integration that deploys the Angular build output on every push. A `staticwebapp.config.json` file at the project root controls routing rules, response headers, fallback routes for SPA navigation, and authentication settings.

**Firebase Hosting**

On Google Cloud, [Firebase Hosting](https://firebase.google.com/docs/hosting) is a fully managed static hosting service that serves content over a global CDN, provides SSL certificates, supports custom domains, and includes atomic deploys with one-click rollback. For Angular applications, Firebase Hosting integrates directly with the [Angular CLI](https://angular.dev/tools/cli) through the [`@angular/fire`](https://github.com/angular/angularfire) library, allowing the team to build and deploy from a single command after the initial project configuration.

### Serverless Containers

Serverless containers extend the container model by removing the need to provision or manage the underlying host machines. The application is still packaged as a Docker image, but the infrastructure layer that runs it is abstracted away entirely. Teams define how much virtual CPU and memory each container needs, and the platform handles placement, scaling, networking, and patching transparently. This approach suits Angular applications served by [Nginx](https://nginx.org/) or another static web server inside a container when teams want more control over the runtime than static hosting provides but without the operational overhead of managing clusters or virtual machines.

**Amazon ECS with Fargate**

On AWS, [Amazon Elastic Container Service](https://aws.amazon.com/ecs/) (ECS) combined with [AWS Fargate](https://aws.amazon.com/fargate/) provides a fully serverless container runtime. Teams define a task definition specifying the container image, resource requirements, port mappings, and environment variables, and Fargate provisions the exact compute requested without exposing any underlying EC2 instances. An ECS service keeps the desired number of task replicas running and integrates with an [Application Load Balancer](https://aws.amazon.com/elasticloadbalancing/) to distribute traffic, while Auto Scaling policies adjust the task count based on CPU or memory usage. The container image is typically stored in [Amazon ECR](https://aws.amazon.com/ecr/) (Elastic Container Registry), and is built and pushed from a CI/CD pipeline.

**Azure Container Apps**

On Azure, [Azure Container Apps](https://azure.microsoft.com/products/container-apps) is the common serverless container platform for production workloads. It provides a fully managed environment with built-in autoscaling from zero to many replicas, traffic splitting between revisions for canary deployments, and health probes. Teams push the container image to [Azure Container Registry](https://azure.microsoft.com/products/container-registry), define the container app configuration with resource limits and ingress settings, and the platform handles TLS termination, load balancing, and scaling transparently.

**Google Cloud Run**

On Google Cloud, [Cloud Run](https://cloud.google.com/run) is a fully managed serverless container platform that scales automatically from zero to thousands of instances based on incoming traffic. Teams push the container image to [Artifact Registry](https://cloud.google.com/artifact-registry) and deploy through the `gcloud` CLI or [Cloud Build](https://cloud.google.com/build) CI/CD platform. Cloud Run integrates with [Cloud Load Balancing](https://cloud.google.com/load-balancing) for custom domains and global traffic management, [Cloud CDN](https://cloud.google.com/cdn) for edge caching of static assets, and [Cloud IAM](https://cloud.google.com/iam) for access control.

### Kubernetes Cluster

[Kubernetes](https://kubernetes.io/) is an open-source container orchestration platform that automates the deployment, scaling, and operational management of containerized workloads across a cluster of nodes. Running Angular applications on Kubernetes makes sense when teams need fine-grained control over routing, rolling updates, resource quotas, health probes, multi-environment configuration, or want to co-locate the frontend with backend microservices in a unified platform. Each major cloud provider offers a managed Kubernetes service that removes the responsibility of operating the control plane while teams manage workloads through standard Kubernetes primitives such as Deployments, Services, Ingress resources, and horizontal pod autoscaling.

**Amazon Elastic Kubernetes Service**

On AWS, [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (EKS) provides a managed Kubernetes control plane integrated with [IAM](https://aws.amazon.com/iam/) for access control, [Amazon ECR](https://aws.amazon.com/ecr/) for image storage, and the [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/) for ALB-backed Ingress resources. EKS also supports [Fargate profiles](https://docs.aws.amazon.com/eks/latest/userguide/fargate.html) for running pods serverlessly without managing node groups. The deployment manifests are standard Kubernetes YAML. Only cloud-specific annotations for the Ingress controller and the image registry URL differ from other providers.

**Azure Kubernetes Service**

On Azure, [Azure Kubernetes Service](https://azure.microsoft.com/products/kubernetes-service) (AKS) is the managed Kubernetes offering. Azure handles control plane provisioning, upgrades, and health monitoring, while teams manage only the worker node pools. Container images are stored in [Azure Container Registry](https://azure.microsoft.com/products/container-registry), and the cluster integrates with [Azure Application Gateway](https://azure.microsoft.com/products/application-gateway) for traffic routing, [Azure Monitor](https://azure.microsoft.com/products/monitor) for observability, and [Azure RBAC](https://learn.microsoft.com/azure/role-based-access-control/) for access control.

**Google Kubernetes Engine**

On Google Cloud, [Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine) (GKE) is the managed Kubernetes service. GKE offers two modes: **Standard**, where teams manage node pools and have full control over cluster configuration, and **Autopilot**, where Google manages the nodes entirely and teams only define workloads. Container images are stored in [Artifact Registry](https://cloud.google.com/artifact-registry), and the cluster integrates with [Cloud Load Balancing](https://cloud.google.com/load-balancing) for traffic routing, [Cloud Logging](https://cloud.google.com/logging) and [Cloud Monitoring](https://cloud.google.com/monitoring) for observability, and [Cloud IAM](https://cloud.google.com/iam) for access control.

## Infrastructure Components

### Hosting

In web infrastructure, [web hosting](https://en.wikipedia.org/wiki/Web_hosting_service) is the service layer that makes an application available on the internet and keeps it reliably accessible over time. At the simplest level, static files (HTML, CSS, JavaScript, and images) are uploaded and served directly by a web server.

As requirements grow, teams typically move to managed hosting or virtual servers that add operational capabilities such as TLS, autoscaling, monitoring, and managed networking. At the advanced end, platforms like [Kubernetes](https://kubernetes.io/) orchestrate many containerized services across clusters, providing automated deployments, load balancing, self-healing, and rolling updates for large distributed systems.

For Angular applications, hosting is often static-first because the build output is mostly static assets. A basic setup can serve the generated files from [Nginx](https://nginx.org/) or [Apache](https://httpd.apache.org/), while larger platforms often combine CDN delivery, reverse proxy routing, and containerized workloads for scale and resilience. Choosing the right hosting model depends on expected traffic, latency targets, operational maturity, compliance constraints, and the level of runtime control required by the team.

### Domain Name

A [domain name](https://en.wikipedia.org/wiki/Domain_name) gives users a human-readable address to reach a web application. Instead of remembering IP addresses, users access services through domain names such as `example.com`, while the [Domain Name System (DNS)](https://en.wikipedia.org/wiki/Domain_Name_System) resolves that name to the correct infrastructure endpoint.

In practice, a domain is not just branding. It is also part of the routing layer of the system. DNS records can point traffic to web servers, reverse proxies, CDNs, load balancers, email providers, and verification services. Proper domain management therefore affects availability, security, and deployment flexibility.

From an operational perspective, teams should also consider DNS propagation times, certificate management, subdomain strategy, and failover planning.

### SPA

A [single-page application](https://en.wikipedia.org/wiki/Single-page_application) (SPA) architecture allows teams to move away from monolithic systems and build decoupled frontends.

In a SPA architecture, the browser downloads an initial HTML document together with JavaScript, CSS, and static assets, and then the client application takes responsibility for routing, rendering views, and communicating with backend services. This separation allows frontend teams to evolve the user experience independently from backend implementation details. The frontend can consume REST, GraphQL, or other HTTP APIs without being tightly coupled to server-side page rendering. That makes it easier to adopt headless and composable architectures where multiple services contribute to the final user experience.

SPAs also introduce infrastructure considerations. Because routing happens client-side, the hosting layer must usually rewrite unknown routes back to `index.html`. Teams must also pay close attention to bundle size, caching strategy, SEO requirements, and API latency, since much of the perceived application performance depends on efficient delivery of static assets and fast network calls.

### Reverse Proxy

A [reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy) sits in front of one or more backend services and receives requests on their behalf. It can then forward traffic to application servers, static asset servers, or upstream APIs based on hostnames, paths, headers, or other routing rules. In many production systems, Nginx, Apache, or a cloud-managed gateway performs this role.

Reverse proxies are valuable because they centralize concerns that should not be reimplemented in every application service. For example, they can terminate TLS, compress responses, cache content, enforce security headers, limit request rates, rewrite URLs, and shield internal services from direct public exposure. They are also an effective edge security control point. At this layer, teams can apply WAF policies, bot filtering, IP reputation rules, and request-size or request-rate limits before traffic reaches application services.

For Angular applications, a reverse proxy is commonly used to serve the static assets, route `/api` traffic to backend services, and ensure that SPA routes resolve correctly. It becomes a key control point for both performance and security.

### Load Balancer

A [load balancer](https://en.wikipedia.org/wiki/Load_balancing_(computing)) distributes incoming web traffic across multiple servers to ensure efficient resource utilization and prevent server overload.

Load balancing improves both scalability and availability by distributing requests across multiple application instances. Instead of sending all traffic to a single server, the load balancer chooses from a pool of healthy targets, reducing the risk of bottlenecks and improving fault tolerance when one instance becomes slow or unavailable. Modern cloud platforms often provide managed load balancers with built-in health checks, TLS support, autoscaling integration, and traffic policies.

For frontend applications, load balancing is particularly useful when serving Angular applications from multiple nodes or when splitting traffic between several backend APIs. Combined with health probes and rolling deployments, it allows infrastructure changes to happen with minimal or no downtime.

### CDN

A [Content Delivery Network (CDN)](https://en.wikipedia.org/wiki/Content_delivery_network) uses a geographically distributed network of proxy servers that handle incoming requests from nearby users. The main goal of a CDN is to reduce latency and improve reliability by serving cached content from edge locations closer to end users. Static resources such as JavaScript bundles, stylesheets, images, fonts, and videos benefit significantly from CDN delivery because those files can often be cached and reused across many requests.

To keep CDN behavior reliable across releases, teams should define explicit cache keys, TTL policies by content type, and invalidation or purge workflows as part of deployment automation. Without this, stale HTML or mismatched assets can cause hard-to-diagnose frontend regressions. When loading resources from CDNs, use [Sub-Resource Integrity](https://developer.mozilla.org/docs/Web/Security/Subresource_Integrity) (SRI) hashes to ensure the resource has not been tampered with.

For Angular applications, a CDN is especially effective because the build output is largely static and fingerprinted. That makes it safe to apply aggressive cache policies to versioned assets while keeping the main HTML document on a shorter cache lifetime. When configured correctly, this pattern reduces origin load and speeds up rendering for users across different regions.

## Infrastructure Provisioning

### Infrastructure as Code

[Infrastructure as Code](https://en.wikipedia.org/wiki/Infrastructure_as_code) (IaC) is the most reliable way to implement infrastructure provisioning in modern web platforms. Instead of creating resources manually in cloud consoles, teams declare the desired infrastructure in version-controlled files and let tooling plan, apply, and track changes. This approach reduces configuration drift, improves peer review, and makes rollbacks and audits easier because every change is tied to a pull request and a deployment log.

In open-source IaC workflows, teams often start with [Terraform](https://developer.hashicorp.com/terraform) because of its broad provider ecosystem and predictable plan/apply lifecycle. [OpenTofu](https://opentofu.org/) follows the same declarative model and HCL-based syntax, offering a community-governed path with strong Terraform compatibility for many existing modules and workflows. Other open-source approaches include [Pulumi](https://www.pulumi.com/), which defines infrastructure using general-purpose languages, and [Crossplane](https://www.crossplane.io/), which extends Kubernetes with control-plane-style infrastructure orchestration so cloud resources can be managed as Kubernetes APIs.

Cloud providers offer closer integration with their respective platforms. On AWS, [CloudFormation](https://aws.amazon.com/cloudformation/) is the native declarative engine for provisioning and updating stacks of resources with built-in drift detection and change sets. On Azure, [Bicep](https://learn.microsoft.com/azure/azure-resource-manager/bicep/overview) provides a concise language that compiles to ARM templates and is optimized for Azure Resource Manager deployments. On Google Cloud, [Infrastructure Manager](https://cloud.google.com/infrastructure-manager/docs/overview) is the current managed Terraform-based provisioning service integrated with Google Cloud governance and IAM controls. In practice, these cloud-native tools are especially useful when teams want first-class access to provider features as soon as they are released and deep integration with platform policy, identity, and audit capabilities.

Provisioning and orchestration tools define and create infrastructure resources, but they do not replace day-2 host and runtime configuration. After networks, clusters, load balancers, and storage are provisioned, teams still need to configure operating systems, packages, services, users, certificates, and application runtime settings consistently. That is where configuration management tools such as [Ansible](https://www.ansible.com/), [Chef](https://www.chef.io/), and [Puppet](https://puppet.com/) connect to the IaC workflow. Provisioning creates the infrastructure substrate, and configuration management converges each node or service to the intended operational state.

[Cloud Native Computing Foundation](https://www.cncf.io/) (CNCF) provides the governance ecosystem behind key provisioning and operations standards, including projects such as [Kubernetes](https://kubernetes.io/), [Prometheus](https://prometheus.io/), [OpenTelemetry](https://opentelemetry.io/), and [Helm](https://helm.sh/) that many web platforms depend on.

### Ansible

[Ansible](https://www.ansible.com/) is an open-source software provisioning, configuration management, and application deployment tool enabling Infrastructure as Code. It runs on many Unix-like systems and can configure both Unix-like systems as well as Microsoft Windows. Unlike provisioning tools that create infrastructure from scratch, Ansible is designed to configure existing machines, for example, installing packages, managing services, and enforcing desired state on servers that are already running. It connects to target hosts over SSH, requires no agent on the managed node, and uses its own declarative language to describe system configuration.

Ansible uses playbooks written in YAML to define the tasks that should be applied to one or more hosts. A playbook describes the desired state of the system, and Ansible ensures that state is reached. To use Ansible, we only need to know minimal YAML structure.

The following playbook demonstrates how to install and start [Nginx](http://nginx.org/) on an existing machine:

```yaml
---
- name: Install nginx
  hosts: localhost
  connection: local
  become: true

  tasks:
    - name: Install nginx
      apt: name=nginx state=latest
    - name: Start nginx
      service:
        name: nginx
        state: started
```

Assuming we have already set up *passwordless* *sudo*, we don't need to supply the *-K* or *--ask-become-pass* flags. We can also validate the playbook syntax before running it by adding the *--syntax-check* flag. Then, to execute the playbook run the following command:

```shell
ansible-playbook ./nginx_install.yml
```

After the command completes, Nginx should be installed and running on the target machine. We can verify by visiting the host IP address in a web browser.

Ansible can be used in a CI/CD workflow to configure the target environment and to then deploy the application on it. The purpose of using Ansible in the pipeline flow is to reuse roles and playbooks for configuration, leaving the CI/CD server only as a process orchestrator instead of a shell script executor. To test playbooks, it is recommended to use virtual machines. To do so, we can use [Vagrant](https://www.vagrantup.com/), which allows us to quickly set up virtual environments independently from the virtualization backend used on the local system.

Ansible works against multiple managed nodes or “hosts” in the infrastructure at the same time, using a list or group of lists known as [inventory](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html). Frequently, we manage passwords, and saving these passwords in plain text is a huge security risk. [Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) is a feature of Ansible that allows us to keep sensitive data such as passwords or keys in encrypted files, rather than as plaintext in Playbooks or Roles.

Finally, it is recommended to take a look at [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html) in the official documentation.

### Terraform

[Terraform](https://developer.hashicorp.com/terraform) is an open-source Infrastructure as Code tool used to provision and manage cloud infrastructure declaratively. Instead of creating resources manually in cloud consoles, Terraform lets teams define infrastructure in version-controlled files and apply those definitions consistently across environments. This approach improves repeatability, enables peer review and CI/CD automation, and reduces configuration drift.

Terraform uses configuration files written in HCL (HashiCorp Configuration Language). A Terraform configuration describes the desired state of infrastructure, and then Terraform calculates the changes required to reach that state through its `plan` and `apply` workflow.

In Kubernetes-based platforms, Terraform is commonly used to provision shared infrastructure such as resource groups, container registries, clusters, identities, and networking before application deployment. The following Terraform example provisions the core Azure resources needed for Kubernetes, namely, a resource group, an Azure Container Registry, an AKS cluster, and the role assignment that allows AKS nodes to pull container images from ACR.

```hcl
# Create a resource group
resource "azurerm_resource_group" "default" {
  name     = "rg-angular-production"
  location = "eu-west-1"

  tags = {
    environment = "production"
  }
}

# Create a container registry within the resource group
resource "azurerm_container_registry" "default" {
  name                = "private-registry"
  location            = azurerm_resource_group.default.location
  resource_group_name = azurerm_resource_group.default.name
  sku                 = "Standard"
  admin_enabled       = false
}

# Create a kubernetes cluster within the resource group
resource "azurerm_kubernetes_cluster" "default" {
  name                = "aks-angular-production"
  location            = azurerm_resource_group.default.location
  resource_group_name = azurerm_resource_group.default.name
  dns_prefix          = "angular-prod"

  default_node_pool {
    name       = "default"
    node_count = 2
    vm_size    = "Standard_D2_v2"
  }

  identity {
    type = "SystemAssigned"
  }

  network_profile {
    network_plugin    = "azure"
    load_balancer_sku = "standard"
  }

  tags = {
    environment = "production"
  }
}

# Attaching a container registry to a Kubernetes cluster
resource "azurerm_role_assignment" "default" {
  principal_id         = azurerm_kubernetes_cluster.default.kubelet_identity[0].object_id
  role_definition_name = "AcrPull"
  scope                = azurerm_container_registry.default.id
}
```

After defining the configuration, the standard execution flow is to initialize the working directory, review the execution plan, and then apply the changes with:

```shell
terraform apply -var-file="terraform.tfvars" -auto-approve=true
```

After `terraform apply` completes, the Azure resources should be created and linked. We can verify this by checking the resource group, AKS cluster, and ACR integration in the Azure portal or with the Azure CLI. To make this configuration reusable across different environments, define input variables (for names, locations, node sizes, tags, etc.) and use a remote state backend with locking for team-safe collaboration.

Terraform fits naturally into CI/CD pipelines because infrastructure changes can be validated, reviewed, and applied using the same workflow as application code. We can test Terraform code with [Terraform tests](https://developer.hashicorp.com/terraform/language/tests), enforce consistent style with [terraform fmt](https://developer.hashicorp.com/terraform/cli/commands/fmt), catch configuration issues with [TFLint](https://github.com/terraform-linters/tflint), and scan for security/compliance risks with [Checkov](https://www.checkov.io/). These checks help preventing regressions in infrastructure code.

We can use [Make](https://www.gnu.org/software/make/) to simplify and standardize Terraform work. We can create a `Makefile` that loads environment settings, initializes Terraform, manages workspaces, and gives quick commands for format/lint/test, plan/apply/destroy by environment, plus helper tasks like documentation, security scanning, and cleanup.

Finally, it is recommended to review [Terraform Best Practices](https://www.terraform-best-practices.com/), [Terraform Style Guide](https://developer.hashicorp.com/terraform/language/style) and [Terraform State](https://developer.hashicorp.com/terraform/language/state) in the official documentation.

### Helm

[Helm](https://helm.sh/) is an open-source package manager for [Kubernetes](https://kubernetes.io/) that standardizes how application workloads are packaged, configured, and released. Instead of maintaining duplicated raw manifests for each environment, teams define reusable chart templates and environment-specific values files, then apply them consistently across development, staging, and production.

In Kubernetes-based platforms, [Helm](https://helm.sh/) and [Kustomize](https://kustomize.io/) are commonly used after infrastructure provisioning to install and upgrade runtime components and application workloads. For Angular applications served from containers (for example, with [Nginx](https://nginx.org/)), templates make it straightforward to parameterize replicas, image repositories and tags, ingress hosts, TLS settings, resource requests and limits, autoscaling behavior, and other environment-specific settings.

A minimal Helm template for an Angular application can be stored as `templates/angular-app.yaml` file:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: angular-app
  namespace: "{{ .Values.environment.namespace }}"
  labels:
    app: angular-app
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: angular-app
  template:
    metadata:
      labels:
        app: angular-app
    spec:
      securityContext:
        runAsUser: 1000
        runAsGroup: 1000
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: angular-nginx
          image: private-registry.azurecr.io/angular-app:latest
          ports:
            - containerPort: 8080
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
              - ALL
          resources:
            requests:
              cpu: 250m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 256Mi
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            timeoutSeconds: 5
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 15
            timeoutSeconds: 5
          volumeMounts:
            - name: "cache-nginx-volume"
              mountPath: /var/cache/nginx
            - name: "run-volume"
              mountPath: /var/run
            - name: "tmp-volume"
              mountPath: /var/lib/nginx/tmp
      volumes:
        - name: "cache-nginx-volume"
          emptyDir: {}
        - name: "run-volume"
          emptyDir: {}
        - name: "tmp-volume"
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: angular-app-svc
spec:
  selector:
    app: angular-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: angular-app-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - "{{ .Values.ingress.host }}"
      secretName: angular-app-tls
  rules:
    - host: "{{ .Values.ingress.host }}"
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: angular-app-svc
                port:
                  number: 80
```

Unlike a plain Kubernetes manifest, a Helm template is parameterized through `values.yaml`, which is what makes the same chart reusable across environments. A typical production setup also uses an [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/) with [cert-manager](https://cert-manager.io/) to automate TLS certificate provisioning.

To apply the chart, the command below drives an idempotent release workflow:

```shell
helm upgrade --install --wait app . --values=values.yaml
```

The `upgrade --install` pattern ensures the release is created if missing and upgraded if it already exists, `--wait` blocks until Kubernetes resources become ready, `app` is the release name, `.` points to the chart in the current directory, and `--values=values.yaml` injects environment-specific settings.

This pattern fits CI/CD well because it keeps rollout logic consistent across all environments while still allowing per-environment overrides through values files. Teams usually pair this with [Helm lint](https://helm.sh/docs/helm/helm_lint/) and [KubeLinter](https://docs.kubelinter.io/) for Kubernetes manifest and policy checks, and [Helm Unittest](https://github.com/helm-unittest/helm-unittest) for chart unit tests. We can also use [helm test](https://helm.sh/docs/helm/helm_test/) for integration or smoke tests against the deployed chart.

Policy controls should be treated as part of provisioning, not an afterthought. Kubernetes policy engines enforce organizational guardrails such as "no latest tags", mandatory resource requests/limits, required labels/annotations, restricted hostPath usage, and approved ingress/security settings. [Kyverno](https://kyverno.io/) is especially practical for application teams because policies are written as Kubernetes-native YAML and can validate, mutate, or generate resources at admission time.

### Twelve-Factor App

The [Twelve-Factor App](https://12factor.net/) methodology is a methodology for building software-as-a-service (SaaS) applications. These best practices are designed to enable applications to be built with portability and resilience when deployed to the web.

The twelve factors are:

1. **Codebase**: One codebase tracked in version control can support many deployments, such as development, staging, and production. For Angular projects, that means a single application repository should remain the source of truth, even if it is deployed to multiple environments.
2. **Dependencies**: All dependencies should be declared explicitly and isolated from the system runtime. In Angular projects, this is typically handled through `package.json`, lock files, and reproducible install steps in CI/CD.
3. **Config**: Configuration that varies between environments should be stored outside the code. API URLs, feature flags, analytics keys, and similar values should be injected per environment rather than hard-coded into the application logic.
4. **Backing services**: Databases, message brokers, storage systems, and third-party APIs should be treated as attached resources that can be swapped without major code changes. For frontend systems, this applies especially to backend APIs, identity providers, and observability services.
5. **Build, release, run**: The build stage creates an artifact, the release stage combines it with configuration, and the run stage executes it. For Angular applications, this means producing the deployable static bundle first, then promoting that artifact through environments with environment-specific settings handled separately.
6. **Processes**: Applications should run as one or more stateless processes. Even when Angular is served as static files, the surrounding runtime components such as Nginx containers or SSR processes should avoid relying on local mutable state.
7. **Port binding**: The application should be self-contained and expose its service through a port. This is most relevant when Angular is served through a container, SSR runtime, or edge server process that listens on a defined port rather than depending on a manually managed web server installation.
8. **Concurrency**: Scale out by running multiple process instances instead of making a single instance larger. In practice, this means serving the Angular app from multiple containers, nodes, or edge locations behind load balancing when traffic increases.
9. **Disposability**: Processes should start quickly and shut down gracefully. Fast startup and clean termination improve rolling deployments, autoscaling, and recovery behavior for the containers or runtimes that deliver the frontend.
10. **Dev/prod parity**: Keep development, staging, and production as similar as possible to reduce deployment surprises. Teams should align Node versions, build steps, container images, and infrastructure behavior across environments.
11. **Logs**: Logs should be treated as event streams and emitted to standard output, then collected by the platform. For frontend systems, this usually means streaming web server, CDN, SSR, and platform logs into centralized observability tooling instead of storing them locally.
12. **Admin processes**: One-off administrative tasks should run as short-lived processes in the same environment as the application. Examples include running database migrations for backend dependencies, validating deployment configuration, or executing operational scripts from the same container image or platform context.

Many of these practices have become standard for modern platforms such as containers and orchestration systems. As a result, many organizations view the Twelve-Factor App as a strong foundation and complement it with newer cloud-native practices.

### Well-Architected Frameworks

Well-Architected Frameworks provide a practical decision model for defining infrastructure that is reliable, secure, cost-aware, and operationally sustainable over time. In infrastructure provisioning, they help teams move from ad hoc choices to explicit architecture principles that can be validated during design reviews, encoded in Infrastructure as Code, and continuously checked in CI/CD. Instead of asking only whether an environment can be provisioned, these frameworks guide teams to ask whether it can be operated safely at scale, recovered after failures, evolved without disruptive migrations, and governed consistently across environments.

For web applications, this translates into concrete provisioning decisions, namely, selecting managed services where they reduce operational risk, defining network and identity boundaries early, standardizing logging and metrics during provisioning, planning backup and disaster recovery paths, and embedding cost and performance trade-offs into architecture reviews. Well-Architected guidance also improves repeatability by encouraging tagged resources, documented ownership, policy-as-code controls, and measurable service objectives, all of which reduce configuration drift and make platform behavior more predictable in production.

Major cloud providers publish their own versions of this guidance, with similar goals but provider-specific implementation patterns. The [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/) focuses on pillars such as operational excellence, security, reliability, performance efficiency, cost optimization, and sustainability. The [Azure Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/) emphasizes architecture quality through reliability, security, cost optimization, operational excellence, and performance efficiency in Azure-native services. Lastly, the [Google Cloud Well-Architected Framework](https://docs.cloud.google.com/architecture/framework) provides design and governance recommendations across operations, security, reliability, performance, cost, and sustainability. Using any of these frameworks as a recurring review baseline helps teams define better infrastructure from the start and continuously improve it as requirements evolve.

## Infrastructure Costs

### Cloud Cost Estimates

Estimating cloud spend should happen before provisioning, not after the invoice arrives. For Angular applications, this usually means estimating static hosting, CDN traffic, load balancing, container runtime, observability, and data transfer costs early in design and again during delivery.

Each major cloud provider offers native tooling for early estimates, namely, [AWS Pricing Calculator](https://calculator.aws/), [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/), and [Google Cloud Pricing Calculator](https://cloud.google.com/products/calculator). These calculators are useful for architecture planning and what-if analysis, but they are mostly manual.

For Infrastructure as Code workflows, [Infracost](https://www.infracost.io/) complements cloud provider calculators by reading Terraform code directly and producing cost estimates as part of CI/CD. To use Infracost, create a free API key and set `INFRACOST_API_KEY` so the CLI can retrieve up-to-date pricing data.

To show a breakdown of costs, use the following command:

```bash
infracost breakdown --path .
```

Internally, *Infracost* parses Terraform HCL directly, so cloud credentials are typically not required just to estimate planned infrastructure.

Infracost is often used in addition to budget alerts and cost management reports to provide an addition layer of communication and protection in CI/CD. [Cost policies](https://www.infracost.io/docs/features/cost_policies/) allow DevOps and FinOps teams to define budget guardrails in pull requests and catch high-impact cost changes before merge.

### FinOps

[FinOps](https://www.finops.org/) is an operational framework for managing cloud spending collaboratively across Engineering, Finance, and Product so that teams can make faster decisions with a clear understanding of cost, usage, and business value. The [FinOps Foundation](https://www.finops.org/introduction/what-is-finops/) describes FinOps as a practice for maximizing the business value of the cloud, not simply reducing spend. In practice, this means cloud costs are treated as an engineering concern alongside performance, reliability, and security. Teams do not wait for an end-of-month invoice to discover waste. Instead, they use near-real-time usage and pricing data to understand where money is going, who owns that spend, and whether the resulting architecture is delivering appropriate value.

The FinOps lifecycle is often summarized as `inform`, `optimize`, and `operate`. First, teams build visibility through tagging strategies, account structures, dashboards, budgets, and allocation rules so they can attribute spend to products, environments, and business units. Next, they optimize by rightsizing compute, selecting appropriate storage tiers, reducing idle resources, improving autoscaling behavior, and making better use of discounts such as Reserved Instances, Savings Plans, or committed-use contracts. Finally, they operationalize those practices by reviewing forecasts regularly, embedding cost checks into delivery pipelines, and making cloud efficiency part of normal engineering planning rather than a one-off cleanup exercise.

The [FinOps Framework](https://www.finops.org/framework/) and cloud guidance such as [AWS Cloud Financial Management](https://aws.amazon.com/aws-cost-management/), the [Azure FinOps guide](https://learn.microsoft.com/cloud-computing/finops/), and [Google Cloud FinOps](https://cloud.google.com/learn/what-is-finops) all emphasize that cloud cost management works best when accountability is distributed to the teams creating the spend.

### Best Practices

The following practices outline how to build a repeatable FinOps operating model with clear ownership, visibility, and optimization workflows:

- [ ] **Build accurate cost visibility**: Create near-real-time dashboards that show cost, usage, and trends by account, service, workload, and team. Include variance views (day/week/month) so spend drivers are easy to spot.

- [ ] **Map all spend to the business**: Map cloud costs to cost centers, products, and business units using a strict tagging standard plus account/subscription structure. Define a fallback method for untagged and untaggable resources.

- [ ] **Use showback or chargeback with shared-cost rules**: Make teams accountable for their total cost by publishing showback (or chargeback) reports and allocating shared platform/support costs with a transparent formula.

- [ ] **Set budgets, forecasts, and guardrails**: Maintain budgets and rolling forecasts per team/workload. Add proactive alerts for threshold breaches and anomalies to catch overspend early.

- [ ] **Define ownership, roles, and operating cadence**: Assign clear ownership for every major workload and spending area. Align Engineering, Finance, and Product on a regular FinOps review cadence with explicit responsibilities.

- [ ] **Govern with policy and automation**: Enforce tagging, provisioning, and lifecycle policies through automation, not manual review. Block or remediate non-compliant resources where possible.

- [ ] **Optimize utilization continuously**: Track utilization and rightsize compute, storage, and databases. Remove idle or redundant resources, and automate cleanup of temporary/dev environments.

- [ ] **Scale and schedule intelligently**: Use auto-scaling, scheduled start/stop windows, and environment TTLs to match capacity with demand and avoid paying for inactive resources.

- [ ] **Optimize commitment and purchasing strategy**: Actively manage Reserved Instances, Savings Plans, and CUDs, including coverage, utilization, and expirations. Use spot or preemptible capacity for fault-tolerant workloads.

- [ ] **Optimize architecture and data economics**: Choose cost-appropriate services and storage classes, enforce retention policies, and minimize data transfer/egress costs through better workload placement and design.

- [ ] **Manage vendors and commercial contracts**: Review provider pricing changes, support plans, and contracts regularly. Re-negotiate commitments when usage patterns change.

- [ ] **Deliver spend insights quickly to stakeholders**: Publish daily or weekly cost reports to engineering, product, and finance so teams can react while changes are still inexpensive to correct.

- [ ] **Integrate optimization into engineering workflows**: Push rightsizing and waste-reduction recommendations into sprint planning and backlog tools, so cost improvements are tracked like any other delivery work.

- [ ] **Control software license spend**: Audit marketplace and third-party license usage, remove unused subscriptions, and evaluate open-source alternatives where they meet reliability and security requirements.

- [ ] **Eliminate shadow cloud usage**: Route provisioning through approved platforms and guardrails to prevent unmanaged cloud accounts or ad hoc resources from creating hidden risk and untracked spend.

- [ ] **Shift cost optimization left in the SDLC**: Evaluate cost impact during design and implementation (not only in operations) so architecture and feature decisions include performance, reliability, and cost trade-offs from the start.

- [ ] **Benchmark efficiency externally**: Compare normalized cost and utilization metrics against industry peers to identify whether internal optimization results are competitive or lagging.

- [ ] **Measure outcomes, not just savings**: Track KPIs that connect cost to business value (for example: unit cost, budget variance, commitment coverage/utilization, and optimization realization rate). Use scorecards to drive continuous improvement.

## Dockerizing Angular Applications

### Nginx

[Nginx](https://nginx.org/) is a lightweight, high-performance web server, reverse proxy, and load balancer that is widely used in frontend delivery stacks. It is designed to handle many concurrent connections efficiently, which makes it a common choice for serving static assets, terminating TLS, and routing requests to upstream APIs or application services.

Nginx is often used for Angular applications because it is simple to deploy, easy to configure, and well suited to serving the compiled output of a single-page application. Nginx is usually responsible for serving the built files with good cache behavior and rewriting unknown routes back to `index.html` so the Angular Router can handle client-side navigation.

The following minimal configuration serves an Angular build output, adds a cache-friendly route for static assets, and rewrites application routes to the SPA entry point:

```nginx
server {
  listen 8080 default_server;

  root /usr/share/nginx/html;
  index index.html;

  location / {
    try_files $uri $uri/ /index.html;
  }

  location ~* \.(js|css|png|jpg|jpeg|gif|svg|ico|woff2)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
    try_files $uri =404;
  }
}
```

In a containerized Angular deployment, this configuration is commonly paired with a Docker image that copies the build output into Nginx's default HTML directory and exposes the application on a non-privileged port. From there, Nginx can also be extended to add security headers, compression, request limits, or an `/api` reverse-proxy route when the frontend and backend share the same origin.

### Docker Image

A deployable image is the most common delivery possibility for Angular applications. That packages the built Angular application together with a web server or runtime configuration. Examples include Docker images based on [Nginx](https://nginx.org/), or a hardened enterprise base image stored in a private registry, [Docker Hub](https://hub.docker.com/), or a cloud registry.

A Dockerfile is a recipe for building a Docker image. Next *Dockerfile* uses [Nginx Alpine](https://hub.docker.com/_/nginx) as base image:


```dockerfile
FROM nginx:1.25.3-alpine

COPY config/nginx/nginx.conf /etc/nginx/nginx.conf
COPY dist/browser /usr/share/nginx/html

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
```

This Dockerfile defines a recipe for packaging an Angular application into a self-contained, deployable container. It starts from a lightweight Nginx image, adds a custom server configuration and the compiled Angular application files via `ng build` command, declares the port the server will listen on, and specifies the command that keeps the server running when the container starts.

We can build the Docker image using the following command:

```bash
docker build -f Dockerfile -t angular-app:latest .
```

The resulting image can be deployed anywhere to serve the Angular application. Note that a Nginx configuration is included which contains rules required for redirecting all URLs to *index.html*, where the built-in Angular Router can take over. For more information, see the [Nginx documentation](https://nginx.org/docs/) and Angular's [Server configuration](https://angular.dev/tools/cli/deployment#server-configuration) guidance.

### Best Practices

The following practices provide a practical checklist for building secure, reproducible, and production-ready Docker images for Angular applications:

- [ ] **Use official images**: Start from trusted, maintained bases like the [official Node.js image](https://hub.docker.com/_/node) and [Docker Official Images](https://hub.docker.com/search?q=&type=image&image_filter=official) instead of building runtimes from scratch.

- [ ] **Use a private registry**: Reduce supply-chain risk with trusted private registries. Alternatives include [Nexus](https://www.sonatype.com/products/repository-oss) and [Artifactory](https://jfrog.com/artifactory/).

- [ ] **Use specific image versions**: Avoid mutable `latest`. Pin versions (typically `major.minor`) or use digests for reproducibility with an update strategy.

- [ ] **Use small-sized images**: Consider using a minimal Linux base image like [Alpine](https://hub.docker.com/_/alpine) or [distroless](https://github.com/GoogleContainerTools/distroless) images from Google's distroless project.

- [ ] **Update images frequently**: Rebuild on top of regularly updated base images and follow stable/LTS versioning to capture security fixes.

- [ ] **Scan images regularly**: Shift security left by scanning images in CI with [Docker image scanning](https://docs.docker.com/dhi/how-to/scan/), [Docker and Snyk](https://snyk.io/docker/), [Clair](https://github.com/quay/clair), [Trivy](https://github.com/aquasecurity/trivy), or [Docker Bench for Security](https://github.com/docker/docker-bench-security).

- [ ] **Sign and verify images**: Enforce provenance and integrity with [Sigstore Cosign](https://docs.sigstore.dev/quickstart/quickstart-cosign/). Sign images in CI and verify signatures before pull/run.

- [ ] **Use metadata labels**: Add useful image metadata (owner, license, project) through `LABEL` to improve discoverability and automation.

- [ ] **Exclude unnecessary packages**: Keep only runtime requirements by copying minimal outputs, using production-only dependency install, and multi-stage builds. Use `.dockerignore` file for unnecessary files.

- [ ] **Use multi-stage builds**: Split build and runtime stages with [Docker multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/) to produce smaller and cleaner images.

- [ ] **Use layer caching**: Order Dockerfile steps from least to most frequently changing to maximize [image layer](https://docs.docker.com/get-started/09_image_best/#image-layering) reuse and faster builds.

- [ ] **Build one image for all environments**: Build once in CI and promote the same artifact across dev/test/prod using externalized configuration.

- [ ] **Avoid privileged containers**: Avoid running containers as root. Do not use `--privileged` unless unavoidable. Prefer minimal capabilities via [--cap-drop](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) or via [Security Context](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) in Kubernetes.

- [ ] **Do not map any ports below 1024**: Avoid privileged host ports unless explicitly needed. Expose only required ports.

- [ ] **Prevent confidential data leaks**: Never bake secrets into layers. Inject at runtime/build-time and use orchestrator secret stores like [Docker Swarm](https://docs.docker.com/engine/swarm/) or [Kubernetes](https://kubernetes.io/).

- [ ] **Include health/liveness checks**: Add [HEALTHCHECK](https://docs.docker.com/engine/reference/builder/#healthcheck) for Docker/Swarm or [Liveness/Readiness](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) for Kubernetes.

- [ ] **Configure logging mechanisms**: Log to `stdout`/`stderr` when possible. Prefer structured JSON logs, and use sidecar collection for file-based legacy apps.

- [ ] **Use stateless and immutable containers**: Containers are designed to be stateless and immutable. Stateless means that any state (persistent data of any kind) is stored outside of a container. Immutable means that a container won't be modified during its life.

- [ ] **Limit container resources**: It's a good practice to limit the memory and CPU usage of the containers, especially if we are running multiple containers on a single machine. This can prevent any of the containers from using all available resources and consequently hindering the others.

- [ ] **Use a linter**: Add Dockerfile linting in CI with tools like [hadolint](https://github.com/hadolint/hadolint) to catch anti-patterns early.

## Observability and Monitoring

### Fundamentals

[Observability and Monitoring](https://www.ibm.com/think/topics/observability-vs-monitoring) are closely related, but they are not the same thing.

[Monitoring](https://cloud.google.com/discover/what-is-application-monitoring) focuses on tracking predefined signals to determine whether a system is healthy. It answers questions such as whether latency is rising, error rates crossed a threshold, or a service is unavailable. [Continuous Monitoring](https://en.wikipedia.org/wiki/Continuous_monitoring) extends this idea into an operational practice where systems and infrastructure are watched continuously through dashboards, alerts, and status checks so teams can detect issues quickly and respond before they become outages.

[Observability](https://www.redhat.com/en/topics/devops/what-is-observability) goes further. It is the ability to understand the internal state of a system by examining the signals it emits. In practice, this means collecting and correlating logs, metrics, traces, events, dashboards, and alerts so teams can investigate why a problem is happening, not only whether it is happening. Observability is critical for platform engineering, [Site Reliability Engineering](https://www.redhat.com/topics/devops/what-is-sre) (SRE), and DevOps because modern applications rarely fail in simple and isolated ways.

For Angular and other web applications, the distinction matters. Monitoring is useful for detecting symptoms such as high response times, increased JavaScript error rates, failed health checks, or sudden drops in traffic. Observability is what helps teams connect those symptoms to underlying causes across browsers, APIs, CDNs, reverse proxies, containers, and cloud services. In distributed architectures, where a single user action may involve many components, both capabilities are necessary.

Gathering metrics, monitoring components, and configuring alerts is an essential part of setting up and managing production infrastructure. Observability should be built into the platform from the start rather than added only after failures occur. Once instrumentation is in place, teams can define meaningful service objectives, detect regressions earlier, and respond faster when incidents affect users.

[OpenTelemetry](https://opentelemetry.io/) is the open standard most teams use to implement observability consistently. It provides shared conventions for collecting the [three pillars of observability](https://www.ibm.com/think/insights/observability-pillars), logs, metrics, and traces, across browsers, backend services, and infrastructure, while keeping the telemetry pipeline independent from any single vendor.

### Logging

#### Definition

Logging records discrete events that happened in the system and gives teams the context needed to investigate failures, security incidents, performance degradation, and unexpected user behavior.

For Angular applications, logging is important at two levels, namely, client-side and server-side. In the browser, client-side logs help capture runtime exceptions, failed HTTP requests, routing problems, feature-flag issues, and other errors that may never appear in server logs. In containerized environments, logging is also important because a container can be rescheduled, replaced, or terminated at any time.

Effective logging in Angular and container platforms usually follows a few practical rules:

- Use structured logs, preferably JSON, so events can be filtered, correlated, and queried reliably;
- Include useful context such as environment, service name, version, request path, trace or correlation ID, user session identifier, and severity level;
- Avoid writing secrets, tokens, personal data, or other sensitive information into logs;
- Aggregate logs centrally so teams can investigate incidents across browsers, CDNs, reverse proxies, containers, and backend APIs in one place.

When logs are collected consistently across the frontend and the infrastructure, teams can connect a user-visible problem to the underlying technical cause much faster. A JavaScript error in the Angular client can be correlated with an API timeout, a Nginx upstream failure, or a container restart, which makes logging a foundational part of operating modern web applications.

Following the Twelve-Factor recommendation, containerized applications and delivery layers such as Nginx should write logs to standard output and standard error, allowing the platform to collect, enrich, and forward them to a centralized logging system such as [Loki](https://grafana.com/oss/loki/), [Elasticsearch](https://www.elastic.co/elasticsearch), [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [Azure Monitor](https://azure.microsoft.com/products/monitor), or [Google Cloud Logging](https://cloud.google.com/logging).

#### Logs for Nginx

Nginx provides flexible logging capabilities that can be configured to capture request and response data in structured or custom formats. Nginx writes access logs to a file specified by the `access_log` directive. The `log_format` directive defines a custom format template that controls which fields are captured and how they are presented.

The following example defines a standard log format called `main` that captures essential request and response data:

```nginx
log_format main '$remote_addr - $remote_user [$time_local] $status '
  '"$request" $body_bytes_sent "$http_referer" '
  '"$http_user_agent" "$http_x_forwarded_for"';
```

A more structured JSON format is often preferred because it enables parsing and filtering in log aggregation systems. The next example captures the same data, but displays it in JSON format:

```nginx
log_format json escape=json '{'
  '"time_local": "$time_local",'
  '"remote_addr": "$remote_addr",'
  '"remote_user": "$remote_user",'
  '"request": "$request",'
  '"status": $status,'
  '"body_bytes_sent": $body_bytes_sent,'
  '"request_time": $request_time,'
  '"http_referer": "$http_referer",'
  '"http_user_agent": "$http_user_agent",'
  '"http_x_forwarded_for": "$http_x_forwarded_for"'
'}';
```

Once a format is defined, apply it to the `access_log` directive in the main `http` block, a server block, or a location block. By structuring logs with consistent formats and including trace IDs or request correlation headers, teams can correlate frontend events with backend logs and infrastructure telemetry, enabling faster incident investigation and root-cause analysis.

For containerized environments like Docker or other container orchestration tool, it is recommended to adjust the `access_log` and `error_log` directives in the Nginx configuration to log to `stdout` and `stderr`. This approach helps capture logs in the standard output and error streams, allowing the platform to collect and forward them centrally.

### Metrics

#### Definition

Metrics are numerical measurements collected over time that describe how a system behaves. They are essential because they make trends, saturation, regressions, and service health visible at a glance. Unlike logs, which explain individual events, metrics help answer questions such as whether request latency is increasing, whether error rates are rising after a deployment, whether CPU or memory limits are being approached, or whether traffic patterns changed significantly during a release.

For Angular applications, the most useful metrics are often user and delivery-oriented rather than purely server-oriented. Teams commonly track frontend performance indicators such as page load times, [Core Web Vitals](https://web.dev/vitals/), JavaScript error rates, API response times, cache hit ratios, and request volumes. In containerized applications, metrics are also critical for understanding platform behavior. Common infrastructure metrics include CPU usage, memory consumption, container restarts, network throughput, disk I/O, request rate, error rate, latency, and health-check status. These metrics help connect technical behavior to user experience.

Collecting metrics in containerized environments usually involves several layers:

- The application exposes its own metrics endpoint, often at a path such as `/metrics`, with counters, gauges, and histograms for request rate, duration, and failures;
- The container runtime and host provide infrastructure metrics about CPU, memory, filesystem, and network usage;
- Orchestrators such as Kubernetes add workload-level signals such as pod state, restarts, scheduling failures, and replica health;
- Reverse proxies, load balancers, CDNs, and managed cloud services emit their own operational metrics, which should be collected alongside application telemetry.

To make metrics useful, teams should focus on a small set of meaningful indicators instead of instrumenting everything indiscriminately. For web applications, the [RED Method](https://grafana.com/blog/the-red-method-how-to-instrument-your-services/) (Rate, Errors, and Duration) is often a good starting point. Combined with resource metrics and a few frontend performance measures, this gives a reliable baseline for understanding whether the Angular application, its delivery layer, and its runtime platform are healthy.

In managed cloud platforms, metrics can be sent to services such as [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [Azure Monitor](https://azure.microsoft.com/products/monitor), or [Google Cloud Monitoring](https://cloud.google.com/monitoring) and then visualized or federated into a broader observability platform. Alternatively, [Prometheus](https://prometheus.io/) is a common way to collect these signals. Prometheus scrapes HTTP endpoints on a schedule, stores the resulting time-series data, and makes it available for dashboards and alerts.

#### Metrics for Nginx

[Nginx](http://nginx.org/) can expose operational counters with the built-in [http_stub_status_module](http://nginx.org/en/docs/http/ngx_http_stub_status_module.html#stub_status), and this is usually the default starting point in production because it is simple, lightweight, and available in standard builds.

The following example adds the `access_log` directive to a dedicated location block:

```nginx
location /metrics {
  access_log off;
  stub_status;
}
```

For richer telemetry, [Nginx Plus](https://www.nginx.com/products/nginx/) exposes additional metrics through its [API module](https://nginx.org/en/docs/http/ngx_http_api_module.html) and live activity views. In practice, [Prometheus](https://prometheus.io/) does not scrape `stub_status` directly. A common pattern is to expose `stub_status` on an internal endpoint and use the [Nginx Prometheus Exporter](https://github.com/nginxinc/nginx-prometheus-exporter), which reads that endpoint and republishes the values in Prometheus format. This gives a stable and widely adopted integration path without custom parsing logic in application code.

If exposing Prometheus metrics directly from Nginx is preferred, a custom endpoint pattern also works where full control over output format is required. See the following example:

```nginx
location /metrics {
  access_log off;
  add_header Content-Type text/plain;
  set $metrics '# TYPE connections_active gauge\n';
  set $metrics '${metrics}connections_active $connections_active\n';
  set $metrics '${metrics}# TYPE connections_reading gauge\n';
  set $metrics '${metrics}connections_reading $connections_reading\n';
  set $metrics '${metrics}# TYPE connections_writing gauge\n';
  set $metrics '${metrics}connections_writing $connections_writing\n';
  set $metrics '${metrics}# TYPE connections_waiting gauge\n';
  set $metrics '${metrics}connections_waiting $connections_waiting';
  return 200 $metrics;
}
```

When teams need custom business or request-path metrics inside Nginx itself, the [lua-nginx-module](https://github.com/openresty/lua-nginx-module) provides a practical solution to increment counters in Lua during request processing and expose them on `/metrics` in Prometheus exposition format. This is more flexible than `stub_status` and avoids an external application just for metric shaping, while still integrating cleanly with Prometheus.

Operationally, the recommended order is to start with `stub_status` or custom endpoint for Prometheus as the default path, move to [Nginx Plus](https://www.nginx.com/products/nginx/) when advanced platform telemetry is needed, or adopt [lua-nginx-module](https://github.com/openresty/lua-nginx-module) when domain-specific metrics generated at the edge layer are needed.

### Tracing

#### Definition

Tracing records the lifecycle of a request as it moves through a system. It captures a chain of spans, where each span represents a unit of work such as a browser action, an API call, a database query, or an external service request. While metrics show aggregate behavior and logs capture discrete events, traces reveal how a single transaction actually flowed end to end.

For Angular and other web applications, tracing is especially useful in distributed architectures where one user action can involve many components. In practice, tracing should connect browser telemetry with backend and infrastructure spans by propagating context across service boundaries. Once this instrumentation is in place, teams can correlate traces with logs and metrics, speed up root-cause analysis, and prioritize performance improvements based on real request paths rather than assumptions.

Practical action points for tracing in Angular and container platforms:

- Instrument both frontend and backend paths so a single user action can be followed end-to-end;
- Propagate trace context headers across gateways, APIs, and async boundaries to preserve span relationships;
- Capture trace attributes that improve debugging context, such as environment, service name, version, route, and error status;
- Correlate trace IDs with logs and key metrics in dashboards and alerts to accelerate incident response.

Cloud providers offer managed tracing services that eliminate the need to operate backend infrastructure. On AWS, [AWS X-Ray](https://aws.amazon.com/xray/) can record trace data from SDKs in application code, automatically capture calls from AWS services, and create service dependency diagrams that show latency and error rates at each hop. On Azure, [Application Insights](https://learn.microsoft.com/azure/azure-monitor/app/app-insights-overview) provides distributed tracing with automatic instrumentation for Node.js applications, along with integration with Azure services. On Google Cloud, [Cloud Trace](https://cloud.google.com/trace) integrates with Google Cloud services to collect latency data and generate span distributions and service maps.

For teams that prefer open-source tooling or require multi-cloud tracing, [Jaeger](https://www.jaegertracing.io/) is the de facto standard for distributed tracing. Built on the OpenTelemetry standard and the OpenTracing API, Jaeger provides trace collection, storage, and visualization without lock-in to any cloud provider.

#### Jaeger

[Jaeger](https://www.jaegertracing.io/) is a CNCF-backed open-source project for collecting and visualizing traces from distributed applications. It follows the OpenTelemetry specification, making it compatible with instrumentation libraries for Node.js, Python, Java, Go, and other languages. Jaeger consists of a collector that receives spans from instrumented code, a backend storage layer that can use [Elasticsearch](https://www.elastic.co/elasticsearch), [Cassandra](https://cassandra.apache.org/), or [BadgerDB](https://dgraph-io.github.io/badger/), and a UI for querying and visualizing traces.

A Jaeger deployment typically includes the collector (which receives spans), the query service (which provides the UI), and a backend storage system. For local development or testing, the fastest way to start is using the all-in-one image:

```shell
docker run -d -p 16686:16686 jaegertracing/all-in-one:latest
```

Then, the Jaeger UI is accessible at port `16686`. Teams can search for traces by service name, tags, duration, or error status, then drill into individual traces to see the call graph, latency breakdown, and any error messages or logs associated with each span. In production, Jaeger can be scaled horizontally with a distributed collector fleet and a backend like Elasticsearch, allowing it to handle millions of spans per second from large microservice deployments. Jaeger also supports sampling policies, which can reduce storage and network costs by keeping only traces that match certain criteria, such as errors or slow requests above a threshold.

#### Tracing for Angular

Tracing in Angular usually starts at the browser edge, where a user interaction triggers one or more HTTP requests to backend services. The main goal is to preserve enough request context that teams can follow a user-visible problem across the Angular client, reverse proxies, APIs, and downstream services.

The simplest approach is to attach a trace ID or correlation header to each outgoing request from the web application. In Angular, this is commonly implemented in an `HttpInterceptor` so every outgoing request receives a generated correlation value consistently:

```typescript
@Injectable()
export class ApiInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const clone = req.clone({ setHeaders: { 'X-Correlation-Id': uuid() } });

    return next.handle(clone);
  }
}
```

This approach works well when the primary requirement is operational correlation rather than full span visualization. It is especially useful for teams that already centralize logs because the shared request identifier becomes the join key between frontend and backend events.

The more advanced approach is full distributed tracing with [OpenTelemetry](https://opentelemetry.io/) and a tracing backend such as [Jaeger](https://www.jaegertracing.io/). In that model, the Angular application emits spans for browser interactions and HTTP requests, propagates trace context headers downstream, and lets instrumented backend services continue the same trace. [OpenTelemetry](https://opentelemetry.io/) provides the vendor-neutral instrumentation model and context propagation rules, while [Jaeger](https://www.jaegertracing.io/) provides collection, storage, and visualization of the resulting trace data. This gives teams a full request timeline rather than just a shared identifier, which makes it much easier to identify where latency was introduced, which dependency failed, and how a single user action traversed the system.

### Health Checks

#### Definition

Health checks are lightweight signals used to verify whether an application instance can run correctly and serve traffic. They help platforms decide when to route requests, restart unhealthy processes, or wait for slow-starting services. In modern delivery systems, health checks are foundational for availability and safe rollouts.

For Angular and other web applications, health checks are usually implemented in the web server or the application container. The three core probe types remain essential, namely, liveness confirms the process is alive, readiness confirms it can accept traffic, and startup prevents premature restarts during initialization.

Practical action points for health checks in Angular and container platforms:

- Expose dedicated endpoints such as `/health`, `/ready`, and `/live` with clear and fast responses;
- Keep liveness checks shallow and readiness checks dependency-aware to avoid false restarts;
- Configure probe intervals and timeouts to match application startup and recovery behavior;
- Monitor probe failures as first-class operational signals and correlate them with logs and alerts.

In practice, health checks are usually exposed as a simple HTTP endpoint such as `/health`, `/ready`, or `/live`, and then evaluated by infrastructure components such as [Docker `HEALTHCHECK`](https://docs.docker.com/engine/reference/builder/#healthcheck), [Kubernetes Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/), or external uptime tools such as [Prometheus Blackbox Exporter](https://github.com/prometheus/blackbox_exporter).

#### Health Checks for Nginx

In [Nginx](https://nginx.org/), a health check can be implemented as a small dedicated endpoint such as `/health`.

```nginx
location /health {
  access_log off;
  default_type application/json;
  return 200 '{"status": "UP"}';
}
```

The `access_log off;` directive avoids noisy probe logs, `application/json` declares the response type, and `{"status": "UP"}` returns a fast success response. This is a good fit when the goal is to confirm that the web server itself is alive.

### Alerts

#### Definition

Alerts turn observability data into operational response. Metrics, logs, and traces describe system behavior, while alerting defines when a condition requires human or automated action. A useful alert rule combines a signal, a condition, and an evaluation window, then routes notifications to the appropriate channel.

For Angular and other web applications, effective alerts should be aligned with user impact and service objectives, not raw infrastructure noise. Typical high-value signals include sustained increases in CPU and memory utilization, HTTP 5xx errors, latency degradation, failing readiness checks, repeated container restarts, certificate expiration, CDN origin failures, and abrupt traffic drops. High-quality alerts also provide clear context, including service, environment, current value, threshold, and links to dashboards or runbooks.

Practical action points for alerting in Angular and container platforms:

- Prioritize user-facing service indicators first, especially error rate, latency, and availability;
- Define thresholds and evaluation windows that reduce flapping while still detecting real incidents quickly;
- Route alerts by severity and ownership, with escalation paths for unresolved critical events;
- Include actionable context in every alert payload, such as service, environment, current value, threshold, and dashboard link.

Cloud platforms include built-in tooling for alerting. On AWS, [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) can route alarms through [Amazon SNS](https://aws.amazon.com/sns/). On Azure, [Azure Monitor](https://azure.microsoft.com/products/monitor) evaluates metric and log conditions and dispatches notifications through Action Groups. On Google Cloud, [Cloud Monitoring](https://cloud.google.com/monitoring) supports threshold, query-based, and uptime alert policies.

In Kubernetes and self-managed platforms, [Prometheus](https://prometheus.io/) and [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) provide evaluation, deduplication, silencing, escalation, and routing.

#### Prometheus

[Prometheus](https://prometheus.io/) is an open-source monitoring system with a dimensional data model, flexible query language, efficient time series database, and modern alerting approach. Prometheus scrapes metrics from a client (target) over HTTP and places the data into its time series database, which can be queried using its own DSL. Prometheus uses “exporters” that are installed/configured on the clients to convert and expose their metrics in a Prometheus-compatible format.

The Prometheus ecosystem consists of multiple components, many of which are optional. The [Prometheus server](https://github.com/prometheus/prometheus) scrapes and stores time series data. By default, it comes with a UI out of the box. Another key component is the “Service discovery” feature, which allows Prometheus to auto-discover targets in dynamic environments, such as distributed systems or microservice architectures. This eliminates the need to manually update a static list of addresses. Prometheus uses a separate component for alerting called the [Alertmanager](https://github.com/prometheus/alertmanager). It receives metrics from the Prometheus server, groups and processes them, and forwards alerts to the chosen notification system. Users can also build dashboards and integrate visualization tools such as [Grafana](https://grafana.com/) or [Kibana](https://www.elastic.co/kibana).

We can instantiate Prometheus from Docker image and connect it to the exported metrics of the Docker daemon. Prometheus needs to be configured appropriately so that it knows where to fetch the data from. Such configuration takes place via the *prometheus.yml* file. Below is an example configuration:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    monitor: 'codelab-monitor'
rule_files:
  - 'alert.rules'
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  - job_name: 'docker'
    static_configs:
      - targets: ['localhost:9323']
```

In this configuration, Prometheus scrapes targets every 15 seconds. The `scrape_configs` section defines two `job_name` entries, namely, one for the Prometheus server at port 9090 and another for the Docker engine at port 9323. Detailed instructions for configuring the Docker daemon as a Prometheus target can be found in the [Collect Docker metrics with Prometheus](https://docs.docker.com/engine/daemon/prometheus/) page.

To start a Prometheus service using previous configuration, run the following command:

```shell
docker run -d -p 9090:9090 \
  -v $PWD/prometheus.yml:/etc/prometheus/prometheus.yml \
  -v $PWD/alert.rules:/etc/prometheus/alert.rules \
  prom/prometheus
```

After a few seconds, we can access the Prometheus web interface at [http://localhost:9090](http://localhost:9090). The custom Prometheus configuration and alert rules are optional and serve to demonstrate the concepts described.

#### Alertmanager

[Alertmanager](https://github.com/prometheus/alertmanager) is the alert handling component in the Prometheus ecosystem. Prometheus evaluates alerting expressions and sends firing alerts to Alertmanager, which then groups, deduplicates, silences, and routes notifications to channels such as email, [Microsoft Teams](https://www.microsoft.com/microsoft-teams/), or [Slack](https://slack.com/).

Prometheus alerting rules need to be added to an *alert.rules* file. [Alerting rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/) allow us to define alerts based on metric conditions. When a rule fires, Prometheus forwards that alert to Alertmanager for notification routing. Below is an example of a simple *alert.rules* file:

```yaml
groups:
  - name: example
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} has been unreachable for more than 5 minutes."
```

In this example, the `InstanceDown` alert triggers when the `up` metric equals 0 for 5 minutes, indicating that an instance is down. The `labels` section categorizes the alert, and the `annotations` section provides additional context for the alert notification.

We can find several alert rule examples on the [Awesome Prometheus Alerts Rules](https://samber.github.io/awesome-prometheus-alerts/) site.

### Dashboards

#### Definition

Dashboards are the operational view layer for observability. They turn metrics, logs, and traces into visual context so teams can quickly understand system health, detect anomalies, and prioritize response. Effective dashboards summarize the current state while allowing fast drill-down during incidents.

For Angular and other web applications, dashboards should combine user-experience and platform signals in one place. Typical views include traffic, error rate, latency, Core Web Vitals, API performance, cache behavior, container health, and resource saturation. Bringing these layers together helps teams connect frontend symptoms to backend and infrastructure causes.

Practical action points for dashboards in Angular and container platforms:

- Start with a small set of service-level indicators (availability, latency, error rate, and traffic) before adding deeper panels;
- Use consistent labels, variables, and naming conventions so dashboards are reusable across environments and services;
- Add links from dashboard panels to logs, traces, and runbooks to speed up incident investigation;
- Store dashboards as code so changes are reviewable, versioned, and reproducible across environments.

Cloud platforms provide managed dashboard services integrated with native monitoring stacks. [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/), [Azure Monitor](https://azure.microsoft.com/products/monitor), and [Cloud Monitoring](https://cloud.google.com/monitoring) support custom visualizations, shared dashboards, and environment-scoped filtering for production operations.

For teams that prefer open-source tooling or need multi-cloud visibility, [Grafana](https://grafana.com/) is the de facto standard for dashboard creation and visualization.

#### Grafana

[Grafana](https://grafana.com/) is a multi-platform open-source analytics and interactive visualization web application. It provides charts, graphs, and alerts for the web when connected to supported data sources. It is expandable through a plug-in system. End users can create complex monitoring dashboards using interactive query builders.

Grafana transforms multiple feeds from Prometheus into metrics in a centralized dashboard. To run Grafana in Docker and integrate it with Prometheus, start with the following command:

```shell
docker run -d -p 3000:3000 grafana/grafana
```

Open browser to port 3000 and login with default admin credentials. Once logged in, click on the Grafana icon in the top left corner to show the main menu. Select *Data Sources* to navigate to the data source list page. Click *Add New* in the top navigation bar and fill in the form with the values for Prometheus. Using the *Save & Test* option will verify that we can connect to the Prometheus instance.

With data sources configured and checked, we can now import dashboards for Grafana. We can either import it from [Grafana Dashboards](https://grafana.com/grafana/dashboards) or we can use the JSON file option.

#### Dashboards for Nginx

To build a useful Grafana dashboard for Nginx in containerized environments, first ensure Nginx containers expose metrics and that Prometheus scrapes both Nginx and container-level metrics.

In Grafana, create a dashboard scoped by labels such as `container`, `pod`, `namespace`, or `service` to filter per environment, and add panels for request rate, 4xx/5xx error rate, request latency (`p50/p95/p99`), active connections, upstream response time, and traffic volume (bytes in/out). It is also recommended to include resource and reliability context in the same view, such as container CPU/memory usage, restart count, and basic availability checks, because these help explain spikes in latency or errors.

To accelerate adoption, download and import community dashboards from [Grafana Dashboards](https://grafana.com/grafana/dashboards) and then adapt variables and thresholds to team-specific labels and [SLOs](https://www.atlassian.com/incident-management/kpis/sla-vs-slo-vs-sli).

### Best Practices

Understanding the state of infrastructure and systems is essential for ensuring the reliability and stability of services. One of the best ways to gain this insight is with a robust monitoring system that gathers metrics, visualizes data, and alerts operators when things appear to be broken. Here is a proposal list of best practices for Monitoring and Alerting:

- [ ] **Define monitoring scope and system boundaries**: We need to estimate the scope of the Continuous Monitoring solution, including the systems in the infrastructure. In addition, we should also estimate the systems that should be a part of the monitoring processes.

- [ ] **Perform risk evaluation and asset classification**: We should implement a risk assessment for different assets. In addition, we can also classify assets based on risk and the estimated impact of service failure or data breach.

- [ ] **Prioritize critical services, metrics, and alerts**: Before establishing threshold and notification settings, we should first determine which are the most important. It´s recommend listing these in descending order, starting with what could potentially put the job at risk and finishing with minor issues.

- [ ] **Configure monitoring and log aggregation tools**: The configuration is essential to obtain data from applications. These tools incorporate a feature called log aggregation that collects log data from different applications deployed on the infrastructure.

- [ ] **Apply the RED Method for service monitoring**: There are countless metrics that can be chosen for monitoring, and it can be overwhelming to separate the signal from the noise coming in.  The [RED Method](https://grafana.com/blog/the-red-method-how-to-instrument-your-services/) provides a general framework for monitoring the health of a request-based service via three metrics: Rate, Errors and Duration.

- [ ] **Define SLIs, SLOs, SLAs and error budgets**: Service-level indicators and objectives should be explicit for user-critical journeys. Use error budgets to balance reliability work and feature delivery, and align alert thresholds to service-level violations.

- [ ] **Monitor availability with uptime checks**: We can use uptime monitoring tools. This gives us a dashboard of the uptime checks for the site, whether the checks succeeded, and how long they’ve taken, plus statistics about service uptime.

- [ ] **Use synthetic monitoring for critical user journeys**: Monitor key paths such as login, checkout, and API dependencies from multiple locations. Synthetic tests detect regressions that basic endpoint checks may miss.

- [ ] **Implement Application Performance Monitoring**: Application Performance Monitoring (APM) can provide request and response information, database connection information, remote profiling and tracing for slow spots, and other metrics related to the health of services.

- [ ] **Correlate logs, metrics, and traces with shared context**: Propagate trace or correlation IDs across frontend, edge, and backend components so teams can investigate incidents across systems using a single identifier.

- [ ] **Monitor logs with actionable event standards**: Logs are generated all the time and provide more information than metrics. The first key to good logging is to make logs useful. To do that, only log actionable events that machines or humans can use.

- [ ] **Standardize telemetry naming and labels**: Use consistent metric names, dimensions, severity levels, and environment labels across services to improve dashboard reuse, alert accuracy, and cross-team troubleshooting.

- [ ] **Define retention, privacy, and sampling policies**: Establish how long logs, metrics, and traces are stored, what sensitive data must be redacted, and where sampling is required to control cost without losing incident visibility.

- [ ] **Create clear and actionable dashboards**: Dashboards should combine user-experience and platform signals in one place. Make sure that dashboards include a clear and organized visual representation of the health of the system.

- [ ] **Use vendor support and managed expertise**: Make the most of support offered by software vendor. Recommended when applicable.

- [ ] **Align alerting channels with the target audience**: Monitoring tools that provide a wide range of alerting methods will ensure that, when that alert comes there will be someone there to hear it.

- [ ] **Maintain runbooks and escalation paths for alerts**: Every high-severity alert should map to an owner, a clear runbook, and a defined escalation route to reduce mean time to acknowledge and resolve incidents.

- [ ] **Measure alert quality and reduce noise**: Track false positives, duplicate alerts, and flapping rules. Regularly tune conditions and windows so alerts remain actionable and do not create fatigue.

- [ ] **Monitor the monitoring system itself**: Do not assume that no alerts mean nothing is wrong. There could be a problem with the monitor itself, or the settings.

- [ ] **Prefer buying monitoring solutions over building**: This tricky one attracts contention and debate. While it’s tempting to avoid the cost and risk of being locked in with a vendor by building in-house, modern IT systems are complex, so it’s far safer to buy IT monitoring tools.

- [ ] **Review metrics and thresholds regularly**: The thresholds we establish might be too high or too low. If we’re receiving too many alerts or we aren’t notified when a legitimate issue occurs, it’s time to review the threshold settings.

- [ ] **Test alerting workflows proactively**: We shouldn’t find out whether the alert system is effective only once a critical issue has arisen. Conduct tests so the robustness and effectiveness of the system is certified before an issue occurs.

- [ ] **Run post-incident reviews**: After incidents, identify missing telemetry, unclear dashboards, and weak alerts. Convert findings into tracked actions so the monitoring system improves over time.

- [ ] **Run Continuous Auditing regularly**: The continuous audit is one of the supporting continuous monitoring of best practices that improve the overall process. We should evaluate the constantly emerging and centralized information every day on a consistent basis.

- [ ] **Document the monitoring system**: It is crucial to document exactly how the monitoring tool has been set up and why. Make sure that this documentation can be editable in real-time and easily accessible for all team members.

## Web Security

### Fundamentals

Web security encompasses a range of solutions and policies that organizations rely on to protect their web applications, APIs, users, and supporting infrastructure against unauthorized access, data exposure, tampering, service disruption, and abuse. In practice, it is not a single feature but a combination of controls applied across the browser, application code, API layer, hosting platform, and delivery pipeline.

Many of the most damaging weaknesses in modern systems come from broken access control, insecure defaults, excessive data exposure, weak secret handling, vulnerable dependencies, and misconfigured infrastructure rather than from a single dramatic exploit. This is why security work must cover both application design and operational hardening, namely, encrypt traffic with TLS, limit privileges, isolate services, keep software patched, verify build artifacts, scan dependencies, and monitor logs and telemetry for signs of abuse or compromise.

Web security should therefore be treated as a continuous engineering practice rather than a final review step before release. Threats change over time, dependencies evolve, cloud configurations drift, and new features create new trust boundaries. Effective teams design security requirements early, verify them during development with code review and automated testing, enforce them in CI/CD with dependency scanning and policy checks, and validate them in production through logging, monitoring, alerting, and periodic security assessments.

For Angular applications and other SPA architectures, one of the main concerns is that the browser is an untrusted execution environment. Application code runs on devices the team does not control, communicates over networks that may be observed or manipulated if transport security is weak, and depends on content delivered through CDNs, APIs, and third-party scripts. This makes browser-side threats such as [Cross-Site Scripting](https://owasp.org/www-community/attacks/xss/), [Cross-Site Request Forgery](https://owasp.org/www-community/attacks/csrf), clickjacking, insecure client-side storage, and token theft especially important. At the same time, a frontend application cannot enforce security on its own. Client-side validation improves usability, but real protection still depends on server-side authentication, authorization, input validation, secure session management, and correctly configured HTTP protections such as [Content Security Policy](https://developer.mozilla.org/docs/Web/HTTP/CSP), [HTTP Strict Transport Security](https://developer.mozilla.org/docs/Web/HTTP/Headers/Strict-Transport-Security), cookie attributes, and strict [CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS) rules.

Standards and guidance such as the [OWASP Top 10](https://owasp.org/Top10/), [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/), [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/), [OWASP API Security Top 10](https://owasp.org/API-Security/), [web.dev secure](https://web.dev/explore/secure), and browser-focused documentation from [MDN Web Docs](https://developer.mozilla.org/docs/Web/Security) provide a useful vocabulary for identifying the main concerns and choosing proportionate controls.

### OWASP Top 10

The [OWASP Top 10](https://owasp.org/Top10/) is the most widely used awareness document for common and high-impact web application security risks. It is not a complete security standard and it does not replace a formal verification framework such as [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/), but it gives teams a practical way to recognize the classes of weaknesses that appear most often in real systems.

For DevSecOps teams, the value of the OWASP Top 10 is not limited to backend code review. It helps shape frontend architecture, API design, reverse proxy configuration, session handling, logging, dependency governance, hosting platforms, and CI/CD security controls. It is best treated as a risk taxonomy that highlights where to look first rather than as a checklist that can be ticked off once and forgotten.

The OWASP Top 10:2025 categories are:

1. [A01:2025 - Broken Access Control](https://owasp.org/Top10/2025/A01_2025-Broken_Access_Control/)
2. [A02:2025 - Security Misconfiguration](https://owasp.org/Top10/2025/A02_2025-Security_Misconfiguration/)
3. [A03:2025 - Software Supply Chain Failures](https://owasp.org/Top10/2025/A03_2025-Software_Supply_Chain_Failures/)
4. [A04:2025 - Cryptographic Failures](https://owasp.org/Top10/2025/A04_2025-Cryptographic_Failures/)
5. [A05:2025 - Injection](https://owasp.org/Top10/2025/A05_2025-Injection/)
6. [A06:2025 - Insecure Design](https://owasp.org/Top10/2025/A06_2025-Insecure_Design/)
7. [A07:2025 - Authentication Failures](https://owasp.org/Top10/2025/A07_2025-Authentication_Failures/)
8. [A08:2025 - Software or Data Integrity Failures](https://owasp.org/Top10/2025/A08_2025-Software_or_Data_Integrity_Failures/)
9. [A09:2025 - Security Logging and Alerting Failures](https://owasp.org/Top10/2025/A09_2025-Security_Logging_and_Alerting_Failures/)
10. [A10:2025 - Mishandling of Exceptional Conditions](https://owasp.org/Top10/2025/A10_2025-Mishandling_of_Exceptional_Conditions/)

Several of these categories directly correspond to concerns addressed in this chapter. Broken access control and authentication failures affect how APIs authorize users and how tokens, sessions, and roles are managed. Cryptographic failures relate to TLS, key handling, and protection of data in transit and at rest. Injection remains relevant not only for SQL and command execution on the server, but also for browser-facing risks such as [Cross-Site Scripting](https://owasp.org/www-community/attacks/xss/) when untrusted data reaches unsafe rendering paths. Security misconfiguration appears at the infrastructure layer through overly permissive CORS rules, missing security headers, exposed admin endpoints, weak container defaults, or incorrect CDN and reverse proxy behavior.

In practice, the most effective use of the OWASP Top 10 is to translate each category into concrete engineering controls. That means enforcing server-side authorization for every protected action, using secure defaults in reverse proxies and cloud services, validating and encoding all untrusted input, scanning dependencies, protecting build and release integrity, centralizing security-relevant logs, and testing regularly for both application and infrastructure weaknesses.

### Best Practices

The following recommendations are widely accepted web security best practices for websites, web applications, and APIs:

- [ ] **Use Multi-Factor Authentication**: Require MFA for administrators and privileged users, and strongly encourage it for all user accounts. Prefer phishing-resistant factors (for example, FIDO2/WebAuthn security keys) over SMS whenever possible.

- [ ] **Implement strong password policies**: Enforce long passphrases, block breached/common passwords, and rate-limit login attempts. Store password hashes using modern algorithms such as Argon2id, bcrypt, or scrypt with per-password salts and appropriate work factors.

- [ ] **Apply the Principle of Least Privilege**: Grant only the minimum permissions required for each user, service account, and runtime identity. Review and remove unused permissions regularly to reduce blast radius.

- [ ] **Use Role-Based or Attribute-Based Access Control**: Start with RBAC for clear entitlement boundaries and adopt ABAC where context-aware rules are needed. Always enforce authorization server-side, not only in frontend route guards.

- [ ] **Use HTTPS everywhere**: Encrypt all traffic with TLS, redirect HTTP to HTTPS, and enable HSTS with suitable max-age and subdomain coverage. Decommission weak ciphers and old protocol versions.

- [ ] **Encrypt sensitive data**: Encrypt data in transit and at rest with strong, current algorithms. Classify data first so encryption requirements match sensitivity and compliance obligations.

- [ ] **Implement robust key management**: Keep keys in managed KMS/HSM systems, rotate them regularly, and restrict key usage by policy. Never hardcode encryption keys in source code or container images.

- [ ] **Protect application secrets**: Store API keys, tokens, and credentials in secret-management systems and inject them at runtime. Prevent secret exposure with repository scanning, least-privilege access, and short-lived credentials.

- [ ] **Avoid unsafe deserialization**: Avoid deserializing untrusted data formats unless absolutely necessary. If required, validate payload structure strictly and use safe serializers with constrained type handling.

- [ ] **Validate and sanitize all inputs**: Treat every external input as untrusted, validate against strict allowlists, and enforce schema validation at API boundaries. Apply output encoding per sink to reduce injection risk.

- [ ] **Prevent Cross-Site Scripting**: Use framework-safe rendering patterns, contextual output encoding, and sanitization for untrusted HTML. Add CSP and Trusted Types as defense-in-depth against DOM XSS.

- [ ] **Prevent Cross-Site Request Forgery**: Use anti-CSRF tokens (or double-submit cookie patterns), set cookies with proper SameSite policies, and verify origin/referer where appropriate. Keep CSRF protection enabled on all state-changing endpoints.

- [ ] **Implement secure file uploads**: Restrict file type, size, and content. Validate MIME and magic bytes, and scan uploads for malware. Store uploads outside executable paths and serve them from isolated domains or storage.

- [ ] **Set secure cookie attributes**: Mark sensitive cookies as `Secure`, `HttpOnly`, and `SameSite` (`Lax` or `Strict` as feasible). Scope cookie domain/path narrowly and avoid exposing session tokens to JavaScript.

- [ ] **Use secure session management**: Use short-lived session identifiers or access tokens, regenerate session IDs on authentication events, and revoke sessions on logout or suspicion of compromise. Enforce idle timeout and absolute session lifetime.

- [ ] **Use Strong Authentication for APIs**: Prefer OAuth 2.0/OIDC with scoped tokens and well-defined audiences. Validate token issuer, audience, signature, and expiry on every protected endpoint.

- [ ] **Protect sensitive API endpoints**: Enforce authentication and authorization checks consistently at every sensitive API boundary. Add object-level and function-level access control checks to prevent horizontal and vertical privilege escalation.

- [ ] **Implement rate limiting**: Apply per-IP, per-user, and per-token throttling, with stricter controls for auth and high-risk endpoints. Combine rate limiting with bot detection and anomaly rules.

- [ ] **Configure strict CORS**: Use explicit origin allowlists, methods, and headers, especially for credentialed requests. Avoid wildcard CORS on authenticated APIs.

- [ ] **Use Security Headers**: Configure `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options`, `X-Frame-Options`, and `Referrer-Policy`. Validate headers continuously in staging and production.

- [ ] **Segment networks and isolate services**: Separate public ingress from internal services and sensitive data stores. Enforce deny-by-default network policies and reduce lateral movement paths.

- [ ] **Use a Web Application Firewall**: Use WAF controls to block common attacks such as XSS, injection, and protocol abuse before they reach the app. Keep signatures, managed rules, and custom detections updated.

- [ ] **Use API Gateways for security management**: Place APIs behind gateways to centralize authentication, authorization, throttling, request validation, and observability. This reduces direct exposure of backend services.

- [ ] **Enable comprehensive logging and monitoring**: Log security-relevant events such as authentication failures, privilege changes, sensitive data access, and suspicious API calls. Centralize logs in SIEM/observability tools and wire actionable alerts.

- [ ] **Protect log integrity and retention**: Prevent log tampering with append-only storage, signatures, or immutable retention controls. Define retention and access policies aligned with legal and incident-response needs.

- [ ] **Perform regular vulnerability scanning**: Run authenticated infrastructure and application scans continuously, including container, host, and cloud configuration checks. Triage findings by exploitability and business impact.

- [ ] **Conduct Penetration Testing**: Perform periodic manual penetration tests and targeted red-team exercises for critical systems. Feed findings into backlog remediation with clear ownership and due dates.

- [ ] **Use SAST, DAST, IAST, and SCA**: Combine static, dynamic, interactive, and dependency analysis to improve vulnerability coverage. Integrate these checks into CI/CD with quality gates for critical issues.

- [ ] **Secure third-party libraries and dependencies**: Use trusted, actively maintained packages, keep dependencies current, and remove unused libraries. Continuously monitor CVE feeds and advisories for transitive dependencies.

- [ ] **Strengthen supply chain security**: Generate SBOMs, verify package provenance/signatures, and use trusted registries. Defend against dependency confusion and typosquatting with scoped registries and repository policy controls.

- [ ] **Automate security in CI/CD pipelines**: Shift security left with policy-as-code, secret scanning, dependency checks, IaC scanning, and container image scanning. Protect pipelines with least-privilege identities, branch protections, and artifact signing.

- [ ] **Integrate DevSecOps practices**: Embed security checks and approval gates throughout planning, build, release, and operations workflows. Keep feedback loops short so teams fix issues before production.

- [ ] **Implement a secure SDLC**: Define security requirements at design time, apply secure coding standards, and enforce security reviews at each delivery stage. Treat security as a release criterion, not an optional activity.

- [ ] **Practice threat modeling continuously**: Identify assets, trust boundaries, abuse cases, and likely attacker paths early. Revisit threat models when architecture, integrations, or business flows change.

- [ ] **Practice OWASP Top 10 and ASVS mapping**: Translate OWASP risk categories into concrete engineering controls and test cases. Use ASVS levels to drive consistent verification depth.

- [ ] **Create and test an incident response plan**: Define detection, containment, eradication, recovery, and communication procedures for security incidents. Run tabletop and live exercises to validate operational readiness.

- [ ] **Perform regular Risk Assessments**: Evaluate technical and business risk on a recurring cadence and whenever major changes ship. Prioritize mitigation using likelihood, impact, and exploitability.

- [ ] **Meet compliance and audit requirements**: Map controls to applicable standards (for example, ISO 27001, GDPR, NIST). Maintain evidence through continuous controls monitoring and periodic audits.

- [ ] **Provide security awareness training**: Train developers, operators, and reviewers on secure coding, API security, threat modeling, and incident response. Reinforce learning with hands-on exercises and security-focused peer reviews.

- [ ] **Design for security from the start**: Incorporate security requirements into architecture, implementation, testing, and operations from day one. Security-by-design reduces rework and lowers long-term risk.

## Angular-Specific Security Strategies

### Preventing Cross-Site Scripting

#### Cross-Site Scripting Security Model

Cross-Site Scripting (XSS) allows attackers to inject malicious code into web pages in order to steal data or impersonate users. Angular reduces this risk by treating template-bound values as untrusted by default, escaping and sanitizing them before they reach the DOM, and pairing those protections with platform controls such as Content Security Policy (CSP) and Trusted Types. See [Preventing cross-site scripting](https://angular.dev/best-practices/security#preventing-cross-site-scripting-xss) and [Angular's cross-site scripting security model](https://angular.dev/best-practices/security#angulars-cross-site-scripting-security-model) for more details.

Angular-specific guidance for XSS prevention:

- **Keep templates static and trusted**: Treat template source as trusted code and bound values as untrusted data. Never generate templates by concatenating user input with template syntax, because that creates a code-injection path.
- **Validate browser messaging**: Verify the `origin` of incoming `postMessage` events against a strict allowlist and set an explicit `targetOrigin` when sending messages. Treat `postMessage` channels as untrusted input boundaries equivalent to HTTP endpoints.
- **Harden web worker boundaries**: Treat data exchanged with Web Workers through `postMessage` as a serialization boundary. Validate inputs inside the worker and avoid passing sensitive tokens into worker contexts.

#### Sanitization and Security Contexts

Sanitization inspects an untrusted value and turns it into a value safe for insertion into the DOM. Angular sanitizes values according to their context and provides APIs for cases where automatic sanitization cannot be used. Angular defines four security contexts, namely, `HTML` (used when binding to `innerHTML`), `Style` (used when binding CSS into `style`), `URL` (used for URL properties such as `<a href>`), and `Resource URL` (a URL loaded and executed as code, such as `<script src>`). Angular sanitizes untrusted values for HTML and URLs automatically. Resource URLs cannot be sanitized because they contain arbitrary code. See [Sanitization and security contexts](https://angular.dev/best-practices/security#sanitization-and-security-contexts) for more details.

#### Direct DOM APIs and Explicit Sanitization

Unless Trusted Types are enforced, built-in browser DOM APIs such as `document`, the node available through [ElementRef](https://angular.dev/api/core/ElementRef), and third-party libraries do not protect against XSS automatically. Avoid direct DOM interaction and prefer Angular templates. When unavoidable, sanitize with [DomSanitizer.sanitize()](https://angular.dev/api/platform-browser/DomSanitizer#sanitize) and the appropriate [SecurityContext](https://angular.dev/api/core/SecurityContext). See [Direct use of the DOM APIs and explicit sanitization calls](https://angular.dev/best-practices/security#direct-use-of-the-dom-apis-and-explicit-sanitization-calls) for more details.

#### Trusting Safe Values

When an application genuinely needs to include executable code, display an `<iframe>` from a URL, or construct a potentially dangerous URL, automatic sanitization must be bypassed explicitly. These calls should be treated as security-sensitive code paths and kept as rare, documented exceptions. To bypass sanitization, inject [DomSanitizer](https://angular.dev/api/platform-browser/DomSanitizer) and call `bypassSecurityTrustHtml`, `bypassSecurityTrustScript`, `bypassSecurityTrustStyle`, `bypassSecurityTrustUrl`, or `bypassSecurityTrustResourceUrl`. These methods should only be used for values already verified as safe. See [Trusting safe values](https://angular.dev/best-practices/security#trusting-safe-values) for more details.

#### Content Security Policy

Content Security Policy (CSP) is a platform-level control that provides defense-in-depth against XSS at the browser level, independent of Angular's own sanitization. It restricts which scripts the browser allows, complementing Angular's output encoding. See [Content security policy](https://angular.dev/best-practices/security#content-security-policy) for more details.

Recommendations for setting up and enforcing CSP in Angular applications:

- **Configure CSP headers**: Configure the web server to return a `Content-Security-Policy` header. The minimal policy for a new Angular application is `default-src 'self'; style-src 'self' 'nonce-randomNonceGoesHere'; script-src 'self' 'nonce-randomNonceGoesHere';`. Provide the nonce to Angular using the `autoCsp` workspace option, the `ngCspNonce` attribute on the root element, or the [CSP_NONCE](https://angular.dev/api/core/CSP_NONCE) injection token. Always use unique, unpredictable nonces per request. When behind a CDN, generate nonces at the edge layer rather than the origin.
- **Control third-party script loading at runtime**: Establish a governance process for onboarding external scripts that includes security review, CSP policy update, and [Sub-Resource Integrity](https://developer.mozilla.org/docs/Web/Security/Subresource_Integrity) (SRI) hashes. Load third-party scripts asynchronously and in sandboxed contexts when possible.
- **Apply Permissions Policy**: Use the `Permissions-Policy` HTTP header (formerly `Feature-Policy`) to disable unused browser features such as `camera`, `microphone`, `geolocation`, `payment`, or `usb`, limiting capabilities available to injected or third-party scripts.

#### Enforcing Trusted Types

[Trusted Types](https://w3c.github.io/trusted-types/dist/spec/) provide an additional layer of XSS protection at the DOM level by restricting which DOM sinks the browser allows, complementing Angular's output encoding. Angular defines several policies for enabling Trusted Types, namely, `angular` (required for Angular to function under enforcement), `angular#bundler` (used by the CLI for lazy chunk files), and `angular#unsafe-bypass` (required when `bypassSecurityTrust*` methods are used). Configure the corresponding `Content-Security-Policy: trusted-types` header in production serving infrastructure. See [Enforcing Trusted Types](https://angular.dev/best-practices/security#enforcing-trusted-types) for more details.

#### Using AOT Compilation

The AOT compiler prevents template injection vulnerabilities by compiling templates at build time, and greatly improves application performance. The AOT template compiler is the default compiler used by Angular CLI applications, and we should use it in all production deployments. See [Use the AOT template compiler](https://angular.dev/best-practices/security#use-the-aot-template-compiler) for more details.

An alternative to the AOT compiler is the JIT compiler which compiles templates to executable template code within the browser at runtime. Angular trusts template code, so dynamically generating templates and compiling them, in particular templates containing user data, circumvents Angular's built-in protections. This is a security anti-pattern. For information about dynamically constructing forms in a safe way, see the [Dynamic Forms](https://angular.dev/guide/forms/dynamic-forms) guide.

#### Securing Server-Side Rendering

Server-side Rendering (SSR) introduces unique security concerns because HTML is constructed on the server and user input can reach the rendering pipeline. Injecting template code into an Angular application gives the attacker full control over it. Do not create Angular templates on the server using a templating language, as this introduces template-injection vulnerabilities. Keep all dynamic content in data bindings and never construct templates dynamically from user-supplied data in the SSR request handler. See [Server-side XSS protection](https://angular.dev/best-practices/security#server-side-xss-protection) for more details.

Additional SSR-specific hardening measures:

- **Protect transfer state**: Angular escapes `TransferState` data embedded in `<script id="ng-state">` by default. Keep custom serialization logic from bypassing that protection, and do not place raw user input into transfer state without validation.
- **Generate CSP nonces per request**: Each SSR response should carry a unique, cryptographically random nonce injected into the `Content-Security-Policy` header and into Angular through the `CSP_NONCE` token or the `ngCspNonce` attribute. Do not reuse nonces across requests.
- **Prevent cross-request data leakage**: Avoid global singletons and module-scoped mutable state that can leak data between requests. Scope all user-specific state through Angular's dependency injection and use `REQUEST`/`RESPONSE` tokens from `@angular/ssr`.
- **Apply response cache isolation for authenticated content**: Set `Cache-Control: private, no-store` on SSR responses that contain personalized content. Use `Vary` headers to separate cache entries by authentication state and never cache user-specific HTML without explicit cache-key segmentation.

### Preventing HTTP-level Vulnerabilities

#### Preventing Cross-Site Request Forgery

[Cross-site request forgery](https://owasp.org/www-community/attacks/csrf) (CSRF/XSRF) tricks authenticated users into unknowingly submitting requests that the server cannot distinguish from legitimate ones. The standard defense is a cookie-to-header token pattern because the server sets a randomly generated token in a cookie, the client reads it and sends it as a custom request header, and the server rejects requests where the values do not match. This works because same-origin policy prevents attacker-controlled pages from reading cookies or setting custom headers for another domain. See [Cross-site request forgery](https://angular.dev/best-practices/security#cross-site-request-forgery) for more details.

Angular provides built-in client-side support for CSRF protection through its HttpClient:

- **HttpClient XSRF/CSRF security**: Angular's [HttpClient](https://angular.dev/api/common/http/HttpClient) includes an interceptor that reads a token from a cookie (by default `XSRF-TOKEN`) and sets it as an HTTP header (`X-XSRF-TOKEN`) on all mutating requests to relative and same-origin URLs. Then, the backend must set the token cookie and validate the header on each eligible request.
- **Configure custom cookie/header names**: Use [withXsrfConfiguration](https://angular.dev/api/common/http/withXsrfConfiguration) when backend conventions differ from the defaults. To prevent collisions when multiple Angular applications share the same domain or subdomain, give each application a unique cookie name.

#### Preventing Cross-Site Script Inclusion

[Cross-Site Script Inclusion (XSSI)](https://angular.dev/best-practices/security#cross-site-script-inclusion-xssi) allows an attacker's website to read data from a JSON API by overriding built-in JavaScript object constructors and including the API URL with a `<script>` tag. The attack succeeds only if the returned JSON is executable as JavaScript. Servers prevent this by prefixing all JSON responses with the well-known string `")]}',\n"` to make them non-executable. Angular's [HttpClient](https://angular.dev/api/common/http/HttpClient) recognizes this convention and strips the prefix automatically before parsing. Enable server-side JSON prefixing for sensitive endpoints and treat XSSI protection as defense-in-depth alongside CORS, authentication, and authorization.

#### Authentication, Authorization, and Session Security

Authentication, authorization, and session management are frequently under-specified in SPA architecture documents, yet they are among the most important controls in Angular platforms. This is especially true for how tokens are stored and transmitted, which is a common source of frontend security incidents. The following practices reduce the attack surface across identity, sessions, and credential storage:

- **Use standards-based authentication**: Prefer OAuth 2.1/OIDC with Authorization Code + PKCE for browser applications instead of custom login or token flows.
- **Keep authorization server-side**: Route guards improve UX but are not security boundaries. Enforce role- and permission-based authorization in backend APIs.
- **Apply least privilege**: Request only required scopes and permissions, and separate admin capabilities from standard user paths.
- **Harden session lifecycle**: Implement idle timeout, absolute session expiration, and token revocation for compromised sessions.
- **Session tokens in cookies**: Prefer HttpOnly, secure cookies when the architecture allows it, reducing JavaScript access to credentials.
- **Long-lived token storage**: Avoid storing long-lived tokens in `localStorage` for sensitive applications, because any successful XSS can read them.
- **Token lifetime strategy**: Use short-lived access tokens and rotating refresh tokens to limit replay windows.
- **Protect cookies correctly**: Set `Secure`, `HttpOnly`, and `SameSite` attributes, and use HTTPS end-to-end.
- **Token logging**: Do not log tokens in browser logs, network error messages, or observability payloads.
- **Prevent open redirect through Angular Router**: Validate redirect target URLs before navigating. Reject absolute URLs pointing to external domains unless they appear in an explicit allowlist, and prefer relative paths with `Router.navigate()` or `Router.navigateByUrl()`.
- **Harden service worker lifecycle**: Restrict service worker scope to the minimum necessary path, serve the script with `Cache-Control: no-cache`, and implement a kill-switch to force-unregister compromised versions. Ensure the precache manifest only includes expected application assets.

#### Securing APIs and Configuring CORS

Angular security also depends on API boundary controls, especially in headless and microservice architectures where the frontend communicates with multiple backend services. Proper configuration at the API layer prevents unauthorized access and data leakage:

- **Use strict CORS allowlists**: Allow only trusted origins, methods, and headers. Avoid wildcard origins for authenticated endpoints.
- **Validate all input on the server**: Client-side validation improves usability but does not protect backend resources.
- **Apply API protections**: Use schema validation, rate limiting, request size limits, and consistent error handling that avoids leaking internal details.
- **Use TLS everywhere**: Use TLS between browser, edge, and service tiers to prevent token and data exposure in transit.

#### HTTP Security Headers and Browser Isolation

HTTP security headers harden delivery at the reverse proxy layer and reduce classes of client-side attacks. The Nginx example below combines SPA routing with a practical production baseline that covers transport security, content-type enforcement, clickjacking prevention, and referrer control.

```nginx
server {
  location / {
    try_files $uri $uri/ /index.html;

    add_header Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self' data:" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "DENY" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
  }
}
```

Beyond the baseline described above, additional browser isolation measures can further reduce the remaining client-side attack surface. These include preventing framing with `frame-ancestors 'none'` in the CSP and `X-Frame-Options: DENY`, adopting cross-origin isolation headers where applicable, restricting embedded third-party content, and aligning these controls with CSP and Trusted Types.

### Preventing Server-Side Request Forgery

Angular validates the `Host`, `X-Forwarded-Host`, `X-Forwarded-Proto`, `X-Forwarded-Prefix`, and `X-Forwarded-Port` headers strictly in the SSR/hybrid rendering request pipeline to prevent header-based [SSRF](https://developer.mozilla.org/docs/Web/Security/Attacks/SSRF). Invalid headers trigger an error log, and requests with unrecognized hostnames receive a `400 Bad Request` response. See [Preventing Server-Side Request Forgery (SSRF)](https://angular.dev/best-practices/security#preventing-server-side-request-forgery-ssrf) for more details.

Angular-specific settings for SSRF protection:

- **Configuring allowed hosts**: Define explicit allowed hostnames in `angular.json` under `security.allowedHosts` or when initializing [AngularAppEngine](https://angular.dev/api/ssr/AngularAppEngine) / [AngularNodeAppEngine](https://angular.dev/api/ssr/node/AngularNodeAppEngine). Wildcard patterns such as `*.example.com` are supported. For Node.js deployments, the `NG_ALLOWED_HOSTS` environment variable (comma-separated) provides a runtime alternative. Using `*` to allow all hostnames is strongly discouraged and should only be used when host validation is enforced at a separate layer such as a load balancer or reverse proxy.
- **Configuring trusted proxy headers**: By default, Angular ignores all `X-Forwarded-*` headers. Enable `trustProxyHeaders` only when the application runs behind a trusted reverse proxy that strictly validates or overwrites these headers. Configure trusted headers when initializing [AngularAppEngine](https://angular.dev/api/ssr/AngularAppEngine) or [AngularNodeAppEngine](https://angular.dev/api/ssr/node/AngularNodeAppEngine), either as a list of specific headers or as `true` to trust all `X-Forwarded-*` headers.

### Application Security Governance

#### Secure SDLC, Dependency Security, and Supply Chain

Dependency and supply chain controls reduce the risk of compromised packages reaching production. Angular applications rely on a deep dependency tree, making proactive management essential:

- **Establish dependency update policy**: Use regular patch windows and emergency vulnerability response runbooks.
- **Pin and verify CI/CD actions and build tooling**: Reduce pipeline compromise risk by pinning and validating build pipeline dependencies. Pin build tool versions, audit build dependencies separately from runtime dependencies, and restrict network access to only the required data sources.
- **Generate and track SBOMs**: Improve vulnerability response and compliance reporting across the dependency tree.
- **Sign build artifacts and container images**: Verify signatures during deployment to enforce integrity and provenance.
- **Prevent build-time secret leakage**: Ensure that environment variables, API keys, and service tokens used during pre-rendering are never serialized into the generated HTML or JavaScript bundles. Use CI/CD secret scanning tools to audit build output before publishing.
- **Review pre-rendered content for data sensitivity**: Verify that pre-rendered pages do not contain role-specific content, draft records, or user-specific data. Content that requires authorization should be loaded at runtime through authenticated API calls, not baked into static pages.
- **Validate SRI hashes for CDN-delivered assets**: Generate [Sub-Resource Integrity](https://developer.mozilla.org/docs/Web/Security/Subresource_Integrity) (SRI) hashes for JavaScript and CSS bundles at build time and include them in `integrity` attributes so browsers reject tampered assets served from CDN or intermediate caches.

#### Security Auditing and Continuous Verification

Angular applications must follow the same security principles as regular web applications and should be audited accordingly:

- **Add static and dynamic security testing** (SAST/DAST) in CI/CD for both Angular code and deployed endpoints.
- **Conduct regular security audits**: Review Angular-specific patterns including `bypassSecurityTrust*` usage, direct DOM manipulation, dynamic template generation, and third-party library integrations.
- **Automate dependency vulnerability scanning**: Integrate tools such as `npm audit`, [Dependabot](https://github.com/dependabot), or [Snyk](https://snyk.io/) into CI/CD pipelines to catch known vulnerabilities before deployment.
- **Verify security controls in production**: Periodically validate that CSP headers, Trusted Types, CORS policies, and other security configurations remain correctly applied across all environments.

## Final Remarks

This chapter covered the core elements required to deliver and operate Angular applications in production, spanning hosting and cloud architecture, automation, and Infrastructure as Code. The key themes (observability, security, and cost management) are not afterthoughts but fundamental to reliable operations.

Building effective operations demands sustained investment in tooling, process, and culture. Cloud providers and open-source ecosystems offer mature solutions for most challenges, but teams must configure them thoughtfully, establish clear ownership, document decisions, and cultivate a culture in which infrastructure work is valued on an equal footing with application development. Infrastructure is not a fixed destination but an evolving practice shaped by business needs and technological change. By combining technical depth with sound governance and a commitment to continuous improvement, teams can build Angular applications that are resilient, secure, and ready to adapt to emerging challenges.
