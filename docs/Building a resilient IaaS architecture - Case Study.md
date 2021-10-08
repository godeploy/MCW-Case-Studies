---
lab:
    title: 'Building a resilient IaaS architecture '
---

# Building a resilient IaaS architecture

## Abstract and learning objectives 

In this case study, you will look at how to design for converting/extending an existing IaaS deployment for resiliency. Throughout the whiteboard design session, you will look at the various configuration options and services to help build resilient architectures.

At the end of the case study, you will be better able to design and use resiliency concepts including high availability with Availability Zones, disaster recovery for virtual machines to another region using Azure Site Recovery, and SQL Server high availability and disaster recovery using Always On Availability Groups. You will also learn how to assess the availability SLA, RPO and RTO of your design, and how to use Azure Backup to protect and secure your SQL data and VMs against corruption and loss.

You will also discuss how to achieve a similar level of resiliency for a PaaS-based implementation the same application, based on Azure App Service and Azure SQL Database. Finally, you will consider the costs associated with both approaches.

## Step 1: Review the customer case study 

**Outcome** 

Analyze your customer’s needs.


### Customer situation

Contoso Insurance (CI), headquartered in Miami, provides insurance solutions across North America. Its products include accident and health insurance, life insurance, travel, home, and auto coverage. CI manages data collection services by sending mobile agents directly to the insured to gather information as part of the data collection process for claims from an insured individual. These mobile agents are based all over the US and are residents of the region in which they work. Mobile agents are managed remotely, and each regional corporate office has a support staff responsible for scheduling their time based on requests that arrive to the system. The company's headquarters in in Miami, Florida with a second large location in Seattle, Washington along with three smaller branch offices scatted around the United States.

Contoso would be considered by most as a classic IT shop, mainly focused on their infrastructure. Their application development department's skill set is dated, predominantly focused on client/server development. Two years ago, the company began a project to move portions of their infrastructure to Azure to gain efficiencies and eventually exit the hardware obsolescence cycle. This project has been executed under the leadership of Lewis Franklin, head of infrastructure and operations.

Many Contoso applications have an Active Directory Domain Services (AD DS) dependency, and so building a domain controller infrastructure was the starting point for Contoso's cloud build-out. For the current AD DS implementation in Azure, the team has deployed a single domain controller VM in the West Central US region. This region was chosen due to its proximity to the Cheyenne Headquarters. It is running on a Standard D2 instance with Active Directory deployed on the C: drive.

![image](https://user-images.githubusercontent.com/25365143/136541498-991016be-abb5-4d2d-ba92-5e4aa02e37b1.png)

For many years Contoso's claims process was done mainly via phone by their remote agents. This was then upgraded to a web-based claims application. The web application team then migrated the claims application to Azure using VMs within the same West Central US region. They have deployed a load balancer in front of the web servers and configured a TCP health probe to monitor the servers in the load balanced pool. When they need scalability, they manually configure another web server and often leave it running even after the need for additional capacity has passed.

Taking their cue from the AD and Web teams, the Database Administrators have also rolled out their SQL servers onto Azure VMs, choosing to host them in the West Central US region as well. The claims application database has been deployed on a single VM with multiple disks. One disk is utilized for the data; the other disk is for backup and log file storage.

![image](https://user-images.githubusercontent.com/25365143/136541504-1d81253e-6b84-421c-8e0f-3dd78613fa5c.png)
Each of the branch offices are small enough to not require an on-site server infrastructure. These locations have connectivity to the Cheyenne headquarters through a Virtual Private Network (VPN). 

While the Azure deployments have served Contoso well so far, they are concerned about their reliability:

-  Agents have reported intermittent issues with the reliability of the claims application. These incidents have been correlated with service health issues of the underlying SQL Server VM.

-  Over a recent three-day holiday weekend, there was an incident with the AD DS Domain Controllers where the disk drive housing the AD database filled up and corrupted the database. This prompted a high-priority support call to Microsoft. While the damage was mitigated, the team was fortunate that the consequences were minimal. Retroactively, checks were made on other Azure VM disk drives and there were several of them that were getting close to capacity due to teams not proactively monitoring their servers. 

-  At times, various branch offices have experienced connectivity issues over the VPN to Cheyenne. While there is some understanding of these occurrences, there is a desire to increase the stability of the connection as growth continues. Contoso is connected via a Windows Server Routing and Remote Access Service (RRAS) VPN connection to Azure via a Site-to-Site Gateway. They are looking for options to provide redundancy for the hybrid connectivity to Azure due to recent network issues.

These issues prompted Contoso to perform a business impact analysis of the claims application. In the resulting report, Janet Lewis, business continuity team director, says, "It appears that while services have moved to the cloud, the overall paradigm has not moved from the single datacenter model we have always deployed."

As a result, Lewis's team has been given an executive mandate to implement an claims application SLA of at least 99.95% over each calendar month, with proactive monitoring and alerting for key metrics on all critical servers. In addition, the mandate requires a disaster recovery solution in place in the event of a failure of the entire West Central US Azure region. The business has specified a 4 hours recovery time objective (RTO) with a recovery point objective (RPO) of 6 hours for the data. Backup of all critical VMs and databases must also be verified and monitored.

In parallel with the above, Jordan North, the Senior Development Lead responsible for the claims application, has been working on the next-generation architecture for the claims application. He plans to migrate the application from IaaS to PaaS, using Web Apps for the web tier and Azure SQL Database for the database. Aware of the increased focus on resiliency, he is aware that the PaaS migration project will be stalled if it offers a lower level of resilience than the enhanced IaaS implementation. He is therefore looking to implement an equivalent level of high availability, disaster recovery, and backup.

### Customer needs 

1.  Redundancy and resiliency for the AD DS domain controller servers, and the web and database servers for the claims application, to deliver the 99.95% or greater SLA required by the business.

2.  Improved reliability for their VPN connections from branch offices.

3.  An automated mechanism for a quick recovery of the claims application in the event of disaster impacting the entire West Central US Azure region.

4.  A plan for recovery from data corruption or accidental deletion for all critical infrastructure components.

5.  A comprehensive solution for monitoring the health of Azure VMs, databases and backups, with proactive alerting of any issues.

6.  An understanding of how to achieve an equivalent level of high availability, disaster recovery and backup for the next-generation PaaS-based implementation of the claims application.

In addition, Contoso require a detailed understanding of the costs associated with each of the above.

### Customer objections 

1.  Contoso are uncomfortable with any situation that assumes the cloud provider will handle their fail-over.

2.  Contoso want to know their BCDR and backup solutions are secure.

3.  Contoso also want to be able to test both the BCDR and Backup solutions regularly.

### Infographic for common scenarios

![image](https://user-images.githubusercontent.com/25365143/136541524-97efb04c-855d-43bd-a62e-55d90a9a177b.png)

## Step 2: Design a proof of concept solution

**Outcome** 

Design a solution and prepare to present the solution to the target customer audience in a 15-minute chalk-talk format. 

Timeframe: 60 minutes

**Business needs**

Directions: With all participants at your table, answer the following questions and list the answers on a flip chart: 

1.  Who should you present this solution to? Who is your target customer audience? Who are the decision makers? 

2.  What customer business concerns do you need to address with your solution?

**Design**

Directions: Design the solution architecture by drawing it on the board, and separately provide insight into how you will address the following requirements. Identify the steps needed to implement a proof of concept for the proposed solution(s) as well as what would need to be demonstrated to stakeholders.

1.  How will you provide an SLA in excess of 99.95% (per month) for the overall claims application?
  
    -  Consider each application tier: Web, database, and domain controllers.

2.  How can you improve the reliability for the Contoso branch office VPN connections?

    -  Identify and eliminate as many single-points-of-failure as you can.

3.  Describe how you will implement a disaster recovery solution for the claims application.

    -  Which secondary Azure region will you use?
    -  How will the DR be configured? Consider each component (web, database, AD, VPN)
    -  What process is required to fail over to the secondary site? Consider each component (web, database, AD, VPN). Are all process steps automated?
    -  What is the impact on agents using the application? How are they routed to the DR site after failover?
    -  Does the solution meet the RPO and RTO requirements?

4.  How will you protect both VMs and databases from data corruption or accidental deletion?

    -  Describe both the solution and the recovery process.

5.  How will you monitor and alert on Azure VMs metrics? Does this approach extend to SQL monitoring? What about backup monitoring?

6.  How can the PaaS implementation of the claims application achieve an equivalent level of resiliency?

    - How is high availability provided by the Web Application and SQL Database? Can the SLA target be met?
    - How will the PaaS solution recover from a complete failure of the primary Azure region? Can the RPO and RTO targets be met?
    - How is backup implemented and executed?

**Pricing**

Provide an estimate of the costs associated with each aspect of your solution.

-  Be sure to cover all aspects of the design, including the primary site, DR solution, backup solution, VPN, and monitoring costs
-  Include a comparison of the IaaS solution and the PaaS solution
-  Have you included all appropriate cost-saving measures?


**Prepare**

Finalize your preparations to present your solution to the customer.

1.  Identify any customer needs that are not addressed with the proposed solution.

2.  Identify any additional benefits of your solution.

3.  Determine how you will respond to the customer’s objections.







##  Additional references

|    |            |
|----------|:-------------:|
| **Description** | **Links** |
| Microsoft Azure Reference Architectures| <https://docs.microsoft.com/azure/guidance/guidance-architecture> |
| Azure Resiliency Overview | <https://azure.microsoft.com/features/resiliency/> |
| Regions and Availability Zones in Azure | <https://docs.microsoft.com/azure/availability-zones/az-overview> | 
| High availability checklist | <https://docs.microsoft.com/azure/resiliency/resiliency-high-availability-checklist> |
| Azure resiliency technical guidance | <https://azure.microsoft.com/documentation/articles/resiliency-technical-guidance/> |
| Introduction to Active Directory Domain Services (AD DS) Virtualization (Level 100) | <https://docs.microsoft.com/windows-server/identity/ad-ds/introduction-to-active-directory-domain-services-ad-ds-virtualization-level-100> |
| Running your AD in Windows Azure | <https://docs.microsoft.com/azure/architecture/reference-architectures/identity/adds-extend-domain> |
| Running VMs for an N-tier architecture on Azure | <https://docs.microsoft.com/azure/guidance/guidance-architecture> |
| High availability with VPN Gateway | <https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-highlyavailable> |
| Azure Backup documentation | <https://docs.microsoft.com/azure/backup/> |
