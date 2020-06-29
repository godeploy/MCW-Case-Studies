---
lab:
    title: 'Cloud-native applications - Model Answer'
---

# Cloud-native applications - Model Answer

## Preferred solution

After evaluating the options for container platforms on Azure and discussing Azure Kubernetes Service (AKS) features with the team at Microsoft, Fabrikam Medical Conferences decided to move forward with Azure Kubernetes Service (AKS).

They also decided to move forward with Azure DevOps for container DevOps workflows.

**Design**

*High-level architecture*

1. Based on the customer situation, what containers would you propose as part of the new microservices architecture for a single conference tenant?
  Each tenant will have the following containers:

     - **Conference Web site**: The SPA application that will use configuration settings to handle custom styles for the tenant.

     - **Admin Web site**: The SPA application that conference owners use to manage conference configuration details, manage attendee registrations, manage campaigns, and communicate with attendees.

     - **Registration service**: The API that handles all registration activities, creating new conference registrations with the appropriate package selections, and associated cost.

     - **Email service**: The API that handles email notifications to conference attendees during registration, or when the conference owners choose to engage the attendees through their admin site.

     - **Config service**: The API that handles conference configuration settings such as dates, locations, pricing tables, early bird specials, countdowns, and related.

     - **Content service**: The API that handles content for the conference such as speakers, sessions, workshops, and sponsors.

2. Without getting into the details (the following sections will address the particular details), diagram your initial vision of the container platform, the containers that should be deployed (for a single tenant), and the data tier.

    The solution will use Azure Kubernetes Service (AKS), which means that the container cluster topology is provisioned according to the number of requested nodes. The proposed containers deployed to the cluster are illustrated below. The data tier is provided by Cosmos DB outside of the container platform:

![A diagram showing the solution, using Azure Kubernetes Service with a CosmosDB back end.](media/solution-topology.png)

*Choosing a container platform on Azure*

1. List the potential platform choices for deploying containers to Azure.

    **Azure Web App for Containers**

    Azure Web App for Containers specifically targets container deployments, which makes it easy to run containers in a fully managed App Service Plan. This option is ideal for solutions that do not require the features offered by an orchestration platform such as Kubernetes.

    **Azure Container Instances**

    Azure Container Instances provide a serverless approach to running containers on demand and at scale enabling additional compute power and elasticity for select workloads - with the security of hypervisor isolation.

    **Windows Server Containers on Windows Server**

    Windows Server Containers allow Windows applications to be containerized. Windows Server 2016 or later versions support the installation of Docker Engine to run containers. For orchestration features you can also set up a cluster with an orchestration platform such as Docker Engine (Community or Enterprise), Kubernetes or other platforms - if you want to take responsibility for managing the clustering and related configurations.

    **Azure Kubernetes Service (AKS)**

    Azure Kubernetes Service (AKS) is the easiest way to manage a Kubernetes cluster on Azure - providing you with a managed control plane and configurable cluster with automatic updates and easy scaling capabilities. AKS removes the management overhead of container orchestration cluster, allowing teams to focus on the application and core DevOps workflows relevant to the solution.

2. Which would you recommend and why?

    Azure Kubernetes Service (AKS) is the recommended platform for the following reasons:

    - It has the necessary orchestration features without the management overhead of the control plane.

    - Ability to monitor and manage applications using a Management UI. This will also make it easier to view the overall state of all tenant applications in a single pane, and drill down into the health of an individual tenant easily.

    - Integration with Container Monitoring Solution in Azure for additional visibility into containers running in the AKS cluster from the Azure Portal, without connecting to the Kubernetes control plane.

    - Full set of integrated features, working out of the box including load balancing, service discovery, self-healing capabilities, scheduling, orchestration, task monitoring, and more.

    - Simple REST API supporting automation with DevOps workflows.

    - Open source, mature, and production tested platform.

    Generally, if the customer has experience with one of the supported orchestrators, you can apply that experience in Azure Kubernetes Service (AKS). There is a great deal of momentum in the community behind Kubernetes, and with Microsoft providing a fully managed solution based on this platform, it is the natural choice.

3. Describe how the customer can provision their Azure Kubernetes Service (AKS) environment to get their POC started.

    - The Azure Kubernetes Service (AKS) environment is deployed using a few simple Azure CLI commands.

*Containers, discovery and load-balancing*

1. Describe the high-level manual steps developers will follow for building images and running containers on Azure Kubernetes Service (AKS) as they build their POC. Include the following components in the summary:

   - The Git repository containing their source.

   - Docker image registry.

   - Steps to build Docker images and push to the registry.

   - Run containers using the Kubernetes dashboard.

   The basic workflow is to build an image from the service source repository, push the image to a registry from which it is deployed, and run as a container.

   A Dockerfile describing each container can reside in the Git repository together with the source. Using command line tools, the developers can build Docker images and push to the registry. A CI process can also automate building images and push to the registry when changes are checked in using Azure DevOps build pipelines.

   To deploy and run a container, the developer can:

   - Securely access the Kubernetes dashboard and create a deployment specifying an image from the repository manually

   - POST a service definition file (JSON) to the REST API using `kubectl` from the command line. This process can also be automated as part of a CD process using Azure DevOps release pipelines.

   - Create Azure DevOps CICD build and release pipelines to automate building images and deploying them to run in the cluster.

2. What options does the customer have for a Docker image registry, and what would you recommend?

   The image registry is core to the CICD workflow and must be a production worthy implementation as it is the source of container images, versioning, deployment, upgrade, and rollback strategies. Registry images can also be used for cross-environment promotion (between development, test, staging, and production for example).

   The following are a few natural options for image registries that could support Azure container deployments:

   - Azure Container Registry is a natural fit with Azure deployments, and it integrates well with deployment options previously mentioned for Docker containers in Azure. This includes an integrated experience in the Azure portal to view the repositories, images, tags, and the contents of manifests associated with an image. In addition, Azure Container Registry has new security features including image quarantine (currently in preview).

   - For development, you can also consider a public Docker Hub account. As all images in the public Docker Hub repository are public; however, this is not typically viable for corporate assets.

   - You can optionally pay for a private repository on Docker Hub, which enables you to control who can access your repository. This comes at a reasonable cost and is fully managed.

   - You can deploy and manage your own Docker Registry in Azure VMs---which would have to be clustered for high availability and this is not trivial to set up. This is not a recommended option when a hosted repository can fit solution requirements.

   Deploying and configuring a Docker Registry, clustered or not, is a complex and time-consuming task. We recommend the use of Azure Container Registry where possible for Azure solutions.

3. How will the customer configure web site containers so that they are reachable publicly at port 80/443 from Azure Kubernetes Service (AKS)?

   When you configure services for a Kubernetes deployment, you can choose to use the public load balancer such that each service instance will be accessible through the Azure load balancer. So long as the required ports are openly accessible, the Azure load balancer will be able to route traffic to all available service instances associated with the endpoint.

   Kubernetes also seamlessly supports load balanced services without making them publicly accessible. Requests from within the cluster can reach internal services and will load balanced across all service instances.

4. Explain how Azure Kubernetes Service (AKS) can route requests to multiple web site containers hosted on the same node at port 80/443

   The location of a container across all nodes in the Azure Kubernetes Service (AKS) cluster should not matter to calling clients. A client application will send a request to a particular endpoint (URL) and expect it to find the correct container instance to service the request. Container routing is an important part of this.

   Web application and api service containers bind to random ports on their host node allowing multiple instances per node. Kubernetes supports dynamic service port discovery and will choose between all instances across nodes to route requests.

*Scalability considerations*

1. Explain to the customer how Azure Kubernetes Service (AKS) supports cluster auto-scaling.

   You can scale the agent nodes in the cluster with the Kubernetes Autoscaler as of Kubernetes 1.10.

*Automating DevOps workflows*

1. Describe how Azure DevOps can help the customer automate their continuous integration and deployment workflows and the Azure Kubernetes Service (AKS) infrastructure.

   With Azure DevOps you can create build definitions that, on commit or check-in can produce build artifacts from the latest source (for example) and build Docker images, then push them to a Docker image repository such as Azure Container Registry. This build definition can be configured to respond to specific folder changes, can build one or more Docker image based on different project folders, and tag images with build number, required image repository tags and other information useful to your image promotion workflows.

   To trigger deployment, you can also use Azure DevOps to produce release definitions that can create or update services in AKS. You may, for example, want your development cluster to always deploy the latest images as code is committed. On the other hand, for test, UAT or production clusters you may want to manually run release jobs based on a specific image tag of the environment in order to control when a new version of a service or services are released.

2. Describe the recommended approach for keeping Azure Kubernetes Service (AKS) nodes up to date with the latest security patches or supported Kubernetes versions.

   Azure applies security patches on a nightly schedule however you must reboot the servers to apply the update. This can be done through the Azure portal or Azure CLI, for example.

   You can upgrade the cluster to a later version of Kubernetes using Azure CLI commands.

## Checklist of preferred objection handling

1. There are many ways to deploy Docker containers on Azure. How do those options compare and what are motivations for each?

   The best of all worlds is to go with a managed orchestration platform like AKS -- native to Azure. It reduces the cost and management overhead of the cluster, while still providing a solution that supports growth, scale, and native management tooling.

    With Kubernetes you will have additional features at your fingertips beyond the pure Docker approach including:

    - The Kubernetes dashboard includes web interface and remote APIs for managing, running, and scaling containers, including CICD integration options.

    - The kubectl command line tool for engaging remote Kubernetes APIs and assisting with automation.

    - Built-in dynamic service discovery simplifies the deployment of new container instances to a load balanced environment.

2. Is there an option in Azure that provides container orchestration platform features that are easy to manage and migrate to, that can also handle our scale and management workflow requirements?

    The easiest way to move to containers on Azure is to deploy containers to the Linux variant of App Service. However, this option does not provide a full-featured container orchestration platform with highly customizable load balancing, dynamic service discovery, and a holistic approach to container monitoring.

    Azure Container Instances also provide a simple way to manage individual containers without management tooling.

    Azure Kubernetes Service (AKS) provides a fully managed service with the full set of orchestration and management tools. This is the best possible choice for reduced management overhead while still having access to the features provided with orchestration platforms like Kubernetes.

## Customer quote

"With Azure Kubernetes Service (AKS) we feel confident we can make the move to a container-based platform with the right DevOps support in place to be successful with a small team."

- Arthur Block, VP of Engineering at Fabrikam Medical Conferences
