---
lab:
    title: 'Enterprise-ready cloud - Case Study'
--- 

# Enterprise-ready cloud - Case Study

<!-- TOC -->

- [Enterprise-ready cloud Case Study student guide](#enterprise-ready-cloud-whiteboard-design-session-student-guide)
  - [Abstract and learning objectives](#abstract-and-learning-objectives)
  - [Step 1: Review the customer case study](#step-1-review-the-customer-case-study)
    - [Customer situation](#customer-situation)
    - [Customer needs](#customer-needs)
    - [Customer objections](#customer-objections)
    - [Infographic for common scenarios](#infographic-for-common-scenarios)
  - [Step 2: Design a proof of concept solution](#step-2-design-a-proof-of-concept-solution)
  - [Step 3: Present the solution](#step-3-present-the-solution)
  - [Wrap-up](#wrap-up)
  - [Additional references](#additional-references)

<!-- /TOC -->

# Enterprise-ready cloud Case Study

## Abstract and learning objectives

In this Case Study, you will work in a group to design a comprehensive solution to address concerns about cost management, security, subscription and resource management, identity, deployment, and other areas to help apply an enterprise governance model for Trey Research.

At the end of this Case Study, you will be better able to design a governance plan to showcase the security, governance, and cost management features of Azure. In addition, you'll learn how to provide cost tracking and alerting by business unit and environment, implement a distributed administration model, and control the deployment of Azure services, all delivered at scale across a large enterprise.

## Step 1: Review the customer case study

**Outcome**

Analyze your customer's needs.

### Customer situation

Trey Research is a manufacturing company that builds consumer products with 29.6 billion USD in annual revenue. Trey's headquarters are in New Jersey, but they have data centers and branch offices scattered across the United States with several major offices in the United Kingdom, France, and Japan.

Even as large as it is, Trey seeks to maximize the cost-effectiveness and flexibility of its IT, especially in new projects and business units. With a dizzying number of existing business units, each with its own unique requirements from IT and ballooning costs from internal hardware and data center investment, Trey is looking to the cloud.

Trey is interested in a large-scale solution that will help mitigate creeping costs and start the transition to a modern cloud-based enterprise architecture.

Trey's technical leadership has decided to move forward with a Microsoft enterprise agreement (EA) with a heavy commitment in Microsoft Azure. Ken Greenwald, Trey Research's CTO, is aware of the potential for the cloud, but also has a keen understanding that without strong governance, Trey may end up with an environment that lacks essential business controls. These incorrect practices can then be disbursed across the enterprise, leading to an unmanageable Azure estate and costs which are hard to track or control. Ken wants to start on the right foot by enforcing common-sense best practices from the start.

To kick off planning for integrating Azure into their environment, Ken introduced you to several directors within Trey's Enterprise IT group that have been part of the initial Azure planning process.

Enterprise IT is responsible for managing corporate network connectivity, datacenter distribution, capacity planning, identity, and enterprise wide SaaS services for Trey Research employees. Enterprise IT is also responsible for supporting the services, datacenters, and setting auditing policy on hardware and services.

To help drive Azure adoption and best practices, Ken has chartered a Cloud Governance team within Enterprise IT, headed by Laura Knight. This team will be responsible for all aspects of Azure governance. This includes defining, implementing and enforcing cloud governance and working with other teams to ensure best practices are adopted.

Rather than re-invent the wheel, the Cloud Governance team have decided to adopt Microsoft's Cloud Adoption Framework as a baseline upon which they will build their governance implementation. This framework is divided into five disciplines.

*Cost Management*

Trey Research has three business units: Industrial and Consumer, Electronics, and Life Sciences. Each of the Business Units has the same subunits: Product development, Marketing, and Sales and Support. Sales and Support is further divided into region sub-units (US/EU/Asia). This hierarchy is shown in the following diagram:

![Trey Research has three business units: Industrial and Consumer, Electronics, and Life Sciences. Each of the Business Units has the same subunits: Product development, Marketing, and Sales and Support. Sales and Support is further divided into region subunits (US/EU/Asia).](images/org-chart.png "Trey Research organizational flowchart")

Each business unit and subunit are allocated an Azure quota/budget and is responsible for tracking their expenditure within that budget. Within a business unit, each new project should track its consumption using a specific tag for its IO code within the business unit.

In addition, each business unit has a requirement to break down their costs into the following categories. Accurate allocation of costs between these categories is essential, since this data feeds into gross margin and net profit figures collated by the central finance team and published quarterly to investors.

- Development and Test
- Production
- Support Services
- Infrastructure

Business units and the finance department need tools to accurately and reliably track all Azure costs, including a cost management dashboard and reports to understand current costs and projected future costs. Alerts when budgets are approached or exceeded are required.

In addition, the Cloud Governance team's charter includes company-wide monitoring and reporting of all Azure spend, including reviewing usage to identify cost-saving opportunities and identifying and investigating anomalous spending.

*Security Baseline*

The IT security team have advised a precautionary approach to cloud adoption. The Cloud Governance team are keen to adopt a strong set of best practices to make sure the IT security and business teams feel comfortable to avoid security becoming an adoption blocker.

If a service has an outage, it is important to know the chain of events that led up to the outage and who (if anyone) caused it.

IT security require that all Azure VMs (Windows and Linux) meet their password complexity requirements. They also require that only approved OS images be used as the baseline for any VM.

*Resource Consistency*

While the Cloud Governance team is not yet familiar with the full range of Azure services available. They want to limit Trey's Azure adoption to an initial set of core services and locations, which will be expanded over time. These should be centrally enforced, with the ability to grant exceptions where required (for example, for teams piloting new technologies, or one-off deployments of a shared resource such as an ExpressRoute circuit).

The IT Security team is particularly concerned about production environments. These require additional controls to ensure that resources cannot be accidentally modified or deleted by administrators getting confused between a test workload versus production.

To maintain consistency, the Cloud Governance team is developing a set of naming conventions. These names will apply to subscriptions, resource groups and resources. Trey require the ability to enforce this naming convention consistently across their Azure subscriptions.

*Identity Baseline*

Trey Research has deployed Office 365 and it is configured with federated access to their ADFS servers. Trey's EA has been created within the same organization.

Providing the ability to delegate permissions to different administrators at the business unit and subunit level is critical. However, for an organization the size of Trey Research, it is not possible for the Cloud Governance team to manage all user permissions centrally. Instead, for each new project, an administrator in the business unit should be able to manage access for his or her team. The business unit administrator must not be able to override or circumnavigate governance rules defined by the Cloud Governance team.  Team members must be provided the access they need, to the resources they need, but no more.  To maintain consistency and to enable Cloud Governance team audit, Azure access should be controlled using built-in roles only, not custom roles.

The Trey e-commerce team make significant use of contingent staff. At present, these staff are granted identities in the existing Trey directory and are required to work on-site to gain access to Trey development and test environments. Trey would like to streamline this process and enable remote working. 

*Deployment Acceleration*

Increased agility is one of Trey's primary motivation for adopting the cloud. They are keen to take full advantage of deployment automation to enable agile processes that will enable them to update their services more frequently and easily.

However, they are also concerned about how to maintain consistency and control across environments.  They are concerned that production and test environments may diverge over time. However, they can't use identical automation in both environments, since test environments are often scaled differently to production to save costs.

The Cloud Governance team has developed best-practice reference implementations for commonly deployed services, such as a DMZ network or a pair of web servers. They are looking for a way to automate these deployments. They recognize that these best practices will evolve over time, and so are also looking for a way to track existing deployments to ensure updates are rolled out consistently. In addition, where resources are deployed following Cloud Governance team best practices, individual business units should not be able to modify the configuration of those resources.

### Customer needs

*Cost Management*

1. Provide cost management tools for budgets, alerts, dashboards, spending reports, forecasts, anomaly detection and investigation, and cost-saving recommendations.

2. Implement a charge back mechanism for the business units for resources they consume based on the IO code for each application.

3. Enable allocation of costs between categories: Development and Test, Production, Support Services, and Infrastructure.

*Security Baseline*

1. Enable investigation of changes leading up to any outage.

2. Ensure Windows and Linux VMs meet password complexity requirements.

3. Ensure VMs can only be created using an approved OS image.

*Resource Consistency*

1. Allow the Cloud Governance team to control which Azure services can be used across the business units, while allowing controlled exceptions.

2. Prevent accidental deletion of resources.

3. Implement a common resource naming standard across the organization.

*Identity Baseline*

1. Delegate access management to business units for each application they own. Business unit administrators should not be able to change, or override policies defined by the Cloud Governance team.

2. Ensure staff have access to what they need, but no more, while enforcing that only built-in roles are used. 

3. Identify a solution to streamline identity management and provide remote access for e-commerce team contingent staff.

*Deployment Acceleration*

1. Implement deployment automation while allowing controlled divergence between environments (e.g. smaller footprint for Dev/Test environments)

2. Provide a means to track and update existing best-practice reference implementation deployments to meet updated best practices.

3. Provide a means to prevent best-practice reference implementation deployments being modified outside the control of the Cloud Governance team.
    
### Customer objections

1. Per-subscription configuration won't scale to an organization the size of Trey Research. How can governance controls be implemented with minimum per-subscription configuration overhead?

2. As well as implementing our governance rules on how Azure is used, we need a way to audit that no deployments have been made that bypass those rules. This audit needs to scale across the entire organization.

3. How can we ensure our deployments meet Azure security best practices, and how can we protect our Production workloads even if the security perimeter is compromised?

4. How can Azure help control the costs associated with non-Production VMs left running out-of-hours?

### Infographic for common scenarios

![This is a screenshot of a slide, titled Azure Cost Management. It has the following bullet list items: Understand where costs originate; Set budgets and alerts; Roll up reports across the organization with management groups; Delegated access to reports and usage. There are two screenshots on the right - one of a daily spend chart and another that displays a full view pivot chart grouped by resource group.](images/slide-cost-mgmt.png "Common scenarios - Azure Cost Management")

![This is a screenshot of a slide, titled Role-based access control (RBAC) and Azure policy. It has the following bullet list items: Governance of Azure resources, and Granular access control. This slide also has an illustration of access control, which is separated into two sides by an arrow labeled Access Inheritance. On the left side are subscription, resource groups, and resources. On the right are three sets each of Contributors, Owner, and Readers.](images/slide-rbac-policy.png "Common scenarios - role-based access control (RBAC) and Azure policy")

![This is a screenshot of a slide, titled Management groups. It has the following bulleted list items: Apply governance at scale; Assign RBAC and policy across subscriptions; Aggregate Advisor, Security Center and Cost Management reports from across the organization. On the right, the slide shows a 4-layer management group hierarchy, with subscriptions sitting under various hierarchy nodes.](images/slide-management-groups.png "Common scenarios - management groups")

![This is a screenshot of a slide, titled Azure blueprints. It has the following bulleted list items: Automated provisioning of entire environments; Deployment tracking and updates; Optional locking against unauthorized changes. There is a diagram showing a blueprint composed of role-based access controls, policy, and templates. Arrows show this blueprint being deployed to multiple subscriptions.](images/slide-blueprints.png "Common scenarios - Azure blueprints")

## Step 2: Design a proof of concept solution

**Outcome**

Design a solution and prepare a solution.

**Business needs**

Directions:  Answer the following questions:

1. Who should you present this solution to? Who is your target customer audience? Who are the decision makers?

2. What customer business needs do you need to address with your solution?

**Design**

Directions: Respond to the following questions:

*Cost Management*

1. What tools are available to meet the cost management requirements of the individual business units, the finance team, and the Cloud Governance team? These requirements include setting budgets and alerts for each cost center, creating spending reports and forecasts, and identifying and investigating anomalies. What permissions and configuration are required to enable users to have access to these tools?
  
2. Design a charge back mechanism for the business units for resources they consume based on the IO code for each application. Assuming your design is based on resource-level tags, how will you implement the following use cases?

    - Every resource group must have an 'IOCode' tag
    - Every time a resource is created, it is assigned an 'IOCode' tag with a value matching its resource group
    - Any resource whose 'IOCode' tag is missing or does not match its corresponding resource group tag (for example, after moving the resource between resource groups) can be easily identified. Child resources, for which tags to not apply, are excluded.
  
3. Design a cost allocation mechanism to track Azure costs across the Development and Test, Production, Support Services, and Infrastructure categories.

*Security Baseline*

1. Following an outage, how can you identify and analyze any recent changes which may have contributed? Investigations will require details of which resource was changed, when it was changed, who made the change, and what was changed. How can you track changes to both resource properties (capturing both before and after state) as well as changes inside a virtual machine?

2. How can you ensure that both Windows and Linux VMs meet password complexity requirements?

3. How can you ensure that only approved OS images are used when creating new VMs? Your implementation should support a customizable list of built-in images as well as custom images.

*Resource Consistency*

1. Identify a solution to restrict which services can be used in each Azure subscription, across the company. How will your solution allow exceptions for specific resource types for approved pilot projects or one-off deployments?

2. How can you prevent accidental deletion of production resources, by administrators who require access to manage those resources?

3. How can Enterprise IT enforce a company-wide resource naming convention?

*Identity Baseline*

1. How can you delegate access management to business units for each application they own, while protecting other applications and ensuring that controls implemented by the Cloud Governance teams cannot be circumvented?

2. How can you ensure staff have access to what they need, but no more, while enforcing that only built-in roles are used? 

3. Identify a solution to streamline identity management and provide remote access for e-commerce team contingent staff.

*Deployment Acceleration*

1. How can Trey implement an 'Infrastructure as Code' approach to deployment automation, while still allowing different footprints in different environments?

2. How can the Cloud Governance team track where their best-practices reference implementations are deployed, and manage updates to those deployments?

3. How can the Cloud Governance team prevent best-practice reference implementation deployments from being modified outside of their control?

**Prepare**


1. Identify any customer needs that are not addressed with the proposed solution.

2. Identify the benefits of your solution.

3. Determine how you will respond to the customer's objections.

Prepare a 15-minute chalk-talk style presentation to the customer.

## Additional references

|                                          |                                                                                                              |
|------------------------------------------|:------------------------------------------------------------------------------------------------------------:|
| Azure Governance                         | <https://docs.microsoft.com/azure/governance/>                                                               |
| Microsoft Cloud Adoption Framework (CAF) | <https://docs.microsoft.com/azure/architecture/cloud-adoption/overview>                                      |
| CAF Large Enterprise journey             | <https://docs.microsoft.com/azure/architecture/cloud-adoption/governance/journeys/large-enterprise/overview> |
| Azure Policy                             | <https://docs.microsoft.com/azure/governance/policy/overview>                                                |
| Role Based Access Control Configuration  | <https://docs.microsoft.com/azure/active-directory/role-based-access-control-configure>                      |
| Built in Roles for RBAC                  | <https://docs.microsoft.com/azure/active-directory/role-based-access-built-in-roles>                         |
| Custom Roles for RBAC                    | <https://docs.microsoft.com/azure/role-based-access-control/custom-roles>                                    |
| Management Groups                        | <https://docs.microsoft.com/azure/governance/management-groups>                                              |
| Azure Cost Management                    | <https://docs.microsoft.com/azure/cost-management/overview-cost-mgt>                                         |
| EA Portal Roles                          | <http://www.redbaronofazure.com/?cat=111>                                                                    |
| Azure Blueprints                         | <https://docs.microsoft.com/en-us/azure/governance/blueprints/overview>                                      |
| Azure Resource Graph                     | <https://docs.microsoft.com/azure/governance/resource-graph/>                                                |
| Azure Security Center                    | <https://docs.microsoft.com/azure/security-center/security-center-intro>                                     |
| Auto-shutdown of VMs                     | <https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.StartStopVMSolution?tab=Overview>         |
