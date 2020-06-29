---
lab:
    title: 'Security baseline on Azure - Model Answer'
---

# Security baseline on Azure - Model Answer

## Preferred solution

*High-level architecture*

Without getting into the details (the following sections will address the details), diagram your initial vision for handling the top-level requirements.

1. Network:

    ![High-level network architecture on the left, an Admin icon and an Agent icon point at an internet icon, which points at a box in the middle. In this box are three smaller boxes (WEB-1, PAW-1, and DB-1) that are interconnected with icons for Azure SQL, a DNS server, and an icon of a key on a green circle. The big box in the middle points to four different sites labeled Site 1-4.](images/Whiteboarddesignsessiontrainerguide-Azuresecurityprivacyandcomplianceimages/media/image3.png)

2. Auditing and compliance:

    ![High-level auditing and compliance on the left, Admin, DPO, and SIEM icons point at other icons and icons inside another large box. Inside the box are various icons and three smaller boxes with similarly clustered icons: WEB, DB, and Main; DB, Web, and PAW; and DB, Web, and Main.](images/Whiteboarddesignsessiontrainerguide-Azuresecurityprivacyandcomplianceimages/media/image4.png)

*Securing sensitive data*

On your diagram, indicate how you would secure any sensitive data at rest and in transit with respect to the following:

1. Web Tier

    a.  The topology deploys independent Azure VMs for their corporate website and their external data collection website. Cloud Services were ruled out so that Contoso could avoid changes to their development and deployment experience, making the migration more direct via a lift and shift effort.  However, in the future they are open to deploying to App Services and use  [Azure Virtual Networks](https://docs.microsoft.com/en-us/azure/app-service/web-sites-integrate-with-vnet) to limit traffic.

    b.  Access to the corporate website is restricted to within the virtual network, as no public endpoint is exposed. Access to the public website is possible, as a public endpoint listening for TLS connection on port 443 is exposed via the Azure load balancer, and similarly, the VM itself has a firewall rule allowing TCP connections on port 443. TLS needs to be properly configured, with the appropriate certificates installed and setup with IIS on each VM.

    c.  The connection strings used by the Web Apps to access the database should be removed and replaced with calls to Azure Key Vault.

2. Database Tier

    a.  SQL Azure was setup, and the on-premises database was migrated to it.

    b.  SQL Transparent Data Encryption (TDE) was enabled to provide additional security to sensitive data at rest.

    c.  For highly sensitive fields, Column Level Encryption (CLE) was enabled using Azure Key Vault to protect fields having sensitive data from users who have the ability to view the contents of the table but have no need to read the sensitive data.

    d.  For other reporting users, data masking was implemented to allow partial viewing of sensitive data.

    e.  The SQL Azure server's virtual network was configured only to allow SQL traffic from the web and reporting applications with logging enabled.

3. Network, Internal and External Communications

    a.  With the websites hosted in Azure VM's, the Azure VMs are joined to an isolated virtual network with VPN access enabled from the corporate site (via Site-to-Site configuration).

    b.  The application tiers are separated by virtual networks and subnets, the website VM\'s are placed within a \"front-end\" subnet within the Virtual Network, and the SQL Database VMs are placed in a \"back-end\" subnet. This separation helps to create a security boundary that can be leveraged by firewalls and intrusion detection systems to more easily detect suspicious traffic, and to quickly quarantine traffic from the websites front-end to the back-end database should a breach be detected.

    c.  In the case where the data collection website is hosted in Azure, the Virtual Network must be enabled for point-to-site connectivity, which requires the creation of an address space to be used in assigning IP addresses to member Web Apps as well as a Gateway. By doing so, a similar isolation to that provided by the \"front-end\" VM subnet is enabled.

    d.  The Azure virtual network uses industry-standard IPSEC protocol to secure all communications between the corporate VPN gateway and Azure. Additionally, communication with both should occur using TLS.

*Ensuring auditing and compliance*

Describe how you will use Azure features to ensure the following:

1. How will you monitor and audit VM Access?

    a.  Azure Security Center can be used to implement Just-in-Time (JIT) virtual machine access. You can also lock down VM modification using Azure IAM settings using specific users or groups with specific Azure roles assigned.

    b.  Ensure that admins are included in the appropriate resource administrative groups with appropriate IAM roles assigned and using Privileged Identity Management (PIM).

2. How will you monitor and audit network traffic across Virtual networks?

    a.  Azure Security Center can be used to create custom alerts based on logging data from the Network Security Group rule execution.

    b.  You can use Azure Monitor to do network testing and packet capture across virtual machines and virtual networks.

    c.  Azure Monitor and Log Analytics can be leveraged to create queries across Azure event logging event data that will feed into Power BI reports.

    d.  Azure Sentinel can be used to create alerts and cases for auto-assignment and investigation activities.

3. How will you monitor and audit Azure SQL?

    a.  Auditing and Threat detection was enabled on all SQL Azure databases.

    b.  Azure Diagnostics was enabled, and all logs are being forwarded to Log Analytics and Azure Sentinel.

4. Create custom alerts and execute remediation and investigation activities on detection?

    a.  Azure Sentinel can be used to hunt down specific events and create alerts based on event data.

    b.  Azure Runbooks can be setup to execute on alerts.

5. What tools would you setup to surface audit and for compliance reporting to IT Executives?

    a.  Azure Policy can be used to determine if resource group owners have been following best practice organizational policies.

    b.  Compliance Manager can be used to implement compliance activities and task assignments as it related to legislative requirements.

    c.  Secure score can also be used to determine what activities can be performed to further secure the Azure subscription and its resources.

*Ensuring availability and business continuity*

Describe how you would ensure that the following resources would be available in the unlikely event of an attack or intentional or unintentional data loss.

1. Virtual Machines

    a.  Azure Recovery Service Vault **configured to use GRS** was utilized to make backups of virtual machines.

2. Azure SQL

    a.  Azure Recovery Service Vault was utilized to make hourly log backups of SQL Azure.

    b.  Azure SQL geo-replication was enabled to ensure high availability and redundancy.

*Ensuring protection*

Describe how you would secure each Azure resource from internal and external attacks.

1. Ensure that admin credentials are sufficiently protected and monitored.

    a.  Azure Premium features should be enabled to configure admin credentials to have Multi-factor Authentication.

    b.  Azure AD Identity protection features should be enabled to ensure that if a credential is or attempting to be compromised that the information is available for alerting and reporting, that includes logons, failed logins, locked accounts, active critical issues, etc.

2. Prevent admins from causing intended and unintended harm to the environment such as unapproved software installs.

    a.   Adaptive application controls can be enabled on all virtual machines to ensure that no software is being installed that has not been approved.

    b.  Through Security Center, implement the Auto Provisioning of the Microsoft Monitoring Agent (MMA) extension on all existing and newly created VMs for data collection activities.

3. Admins access Azure resources from secured and/or compliant corporate assets and do not directly access any production Virtual Machines from the internet.

    a.  Privileged Access Workstation (PAW) virtual machine(s) will be set up as the entry point to gain access to all other virtual machines in the Azure subscription. No Access to virtual machines other than the PAW will be remote desktop enabled.

    b.  Microsoft Intune can be used to ensure that only corporate and/or compliant devices can access Azure and Azure resources via conditional access policies.

    c.  Additionally, the VM's in the web app and data tiers can leverage the Microsoft Antimalware extension, or partner security extensions (currently from Symantec and Trend Micro) that include antivirus, antimalware, firewalls and intrusion detection systems.

    d.  All other measures Contoso has already put in place to secure PHI will continue to exist when the solution is deployed to Azure.

## Checklist of preferred objection handling

1. Can Azure support the lift and shift of their web and database applications?

    Yes, and it is the next logical step for many organizations in their cloud journey: 

    ![An arrow labeled Existing .NET application modernization: Maturity models spans four boxes labeled Existing apps (on-premises), Cloud Infrastructure-Ready (Azure), Cloud DevOps-Ready (Azure), and Cloud-Optimized (Azure). The Cloud Infrastructure-Ready box is highlighted in yellow. Below the Cloud Infrastructure-Ready and Cloud DevOps-Ready boxes is an arrow labeled LIFT and SHIFT: No re-architect, no coded changes. Below the Cloud-Optimized box is an arrow labeled Architect for the cloud (might need new code).](images/Whiteboarddesignsessiontrainerguide-Azuresecurityprivacyandcomplianceimages/media/image5.png "Existing .NET application modernization: Maturity models ")

    There are many programs that customers can take advantage of to help with the move Azure including:

    - [Microsoft FastTrack for Azure](https://azure.microsoft.com/en-us/roadmap/fasttrack-for-azure/)

    - [Azure Migrate](https://azure.microsoft.com/en-us/blog/announcing-azure-migrate/)

2. Is Azure SQL secure enough to host their application databases?

    Transparent data encryption performs real-time encryption and decryption of the database, associated backups, and transaction log files to protect information at rest. Transparent data encryption provides assurance that stored data hasn't been subject to unauthorized access.

    Firewall rules prevent all access to database servers until proper permissions are granted. The firewall grants access to databases based on the originating IP address of each request.

    Encrypted columns ensure that sensitive data never appears as plain text inside the database system. After data encryption is enabled, only client applications or application servers with access to the keys can access plain-text data.

    Dynamic data masking limits sensitive data exposure by masking the data to nonprivileged users or applications. It can automatically discover potentially sensitive data and suggest the appropriate masks to be applied. Dynamic data masking helps to reduce access so that sensitive data doesn't exit the database via unauthorized access. Customers are responsible for adjusting settings to adhere to their database schema.

3. Admins are worried that they won't have the bandwidth to perform deployments of the corporate website and other supporting web applications.

    Azure ExpressRoute can be used to create private connections between Azure datacenters and infrastructure on your premises or in a colocation environment. ExpressRoute connections don\'t go over the public Internet, and they offer more reliability, faster speeds, and lower latencies than typical Internet connections. In some cases, using ExpressRoute connections to transfer data between on-premises systems and Azure can give you significant cost benefits.

4. Can Azure help contain costs for minimally used costly production and development resources?

    Yes, Azure Cost Management + Billing has free features that allows you to monitor your cloud spend, drive organizational accountability and optimize your cloud efficiency.

5. Does Azure support the ability to allow VPN connections to specific resources?

    Yes, Azure provides the ability to do site-to-site IPSec VPNs and the ability to create point-to-site VPNs. By utilizing a site-to-site VPN Gateway, you can connect your on-premises networks with Azure utilizing IPSec and IKE. Point-to-site will allow individual client computers to connect to your Azure virtual network(s) via SSTP or IKE.

6. Can Microsoft employees or government entities access our data?

    Access to customer data by Microsoft operations and support personnel is denied by default. When granted, access is carefully managed and logged. Data center access to the systems that store customer data is strictly controlled via lock box processes.

    Microsoft does not share data with third parties unless permitted by the customer.

    Microsoft provides only the data it is legally compelled to provide, but never voluntarily.

    Both employees and subcontractors work at Microsoft data centers.

    We require subcontractors to join Microsoft\'s Vendor Privacy Assurance Program, to meet our privacy requirements by contract, and to undergo regular privacy training. We contractually obligate subcontractors that work in facilities or on equipment controlled by Microsoft to follow our privacy standards. All other subcontractors are contractually obligated to follow privacy standards equivalent to our own.

    Azure Support can access your VMs utilizing the audited Azure Lockbox feature when needed in the resolution of difficult issues.

7. How does Azure protect against threats?

    Azure Active Directory Privileged Identity Management can be used by customers to minimize the number of users who have access to certain resources. Administrators can use Azure AD Privileged Identity Management to discover, restrict, and monitor privileged identities and their access to resources. This functionality also can be used to enforce on-demand, just-in-time administrative access when needed.

    Intrusion detection and prevention systems, denial of service attack prevention, regular penetration testing, and forensic tools help identify and mitigate threats from both outside and inside of Azure. Azure Active Directory Identity Protection detects potential vulnerabilities that affect an organization’s identities. It configures automated responses to detected suspicious actions related to an organization’s identities. It also investigates suspicious incidents to take appropriate action to resolve them.

    Microsoft Antimalware is built-in to Cloud Services and can be enabled for Virtual Machines to help identify and remove viruses, spyware and other malicious software and provide real-time protection. Customers can also run antimalware solutions from partners on their Virtual Machines.

    Microsoft has decades of operating system security experience running its applications and services and latest developments allow the Microsoft Security Graph to analyze events to determine if and when an attack is occurring.

    Azure Application Gateway and Azure Firewall can be combined with Azure DDoS and Threat Intelligence services to ensure bad actors are not being allowed access to the web applications.

    If the customer has a healthy security budget, they could migrate their web applications to Application Service Environments (ASE) with Web Application Gateway (WAG) to implement an even more segregated and secured environment.

8. Does Azure allow enough granular RBAC controls to meet our least privileged needs?

    **Azure RBAC** can be used by administrators to define fine-grained access permissions to grant only the amount of access that users need to perform their jobs. Instead of giving every user unrestricted permissions for Azure resources, administrators can allow only certain actions for accessing data. Subscription access is limited to the subscription administrator.

    Azure comes with several pre-built built-in roles - <https://docs.microsoft.com/en-us/azure/active-directory/role-based-access-built-in-roles> and allows you to create custom roles - <https://docs.microsoft.com/en-us/azure/active-directory/role-based-access-control-custom-roles>.

    Additionally, you can assign individual users and groups to just about any Azure resource.

    Azure Monitoring and Log Analytics can be used to monitor user and application security activity (such as RBAC changes). Organizations can use it to audit, create alerts, and archive data. They also can track API calls in their Azure resources. The activity logs provide insight into operations performed on resources in a subscription. Activity logs can help determine an operation's initiator, time of occurrence, and status.

9. Is Azure virtual networking flexible enough to meet our requirements?

    Azure Virtual Network gives you an isolated and highly-secure environment to run your virtual machines and applications. Use your private IP addresses and define subnets, access control policies, and more. Use Virtual Network to treat Azure the same as you would your own datacenter.

    Traffic between Azure resources in a single region, or in multiple regions, stays in the Azure network---intra-Azure traffic doesn't flow over the Internet. In Azure, traffic for virtual machine-to-virtual machine, storage, and SQL communication only traverses the Azure network, regardless of the source and destination Azure region. Inter-region virtual network-to-virtual network traffic also flows entirely across the Azure network.

    In a virtual network, run your favorite network virtual appliances such as WAN optimizers, load balancers, and application firewalls and define traffic flows, allowing you to design your network with a greater degree of control.

    Use Virtual Network to build your hybrid cloud applications that securely connect to your on-premises datacenter---so an Azure web application can access an on-premises SQL Server database, or authenticate customers against an on-premises Azure Active Directory service.

10. Can Azure supplement on-premises and 3rd party SIEM systems for auditing and compliance tasks?

     Yes, Azure has a robust set of integration scenarios for customers including:

     - Command Line Interface (CLI), PowerShell and a [REST API](https://docs.microsoft.com/en-us/azure/monitoring-and-diagnostics/monitoring-rest-api-walkthrough).

     - Email and webhooks based on triggers.

     - Events streamed to Event Hub.

     - Power BI integration.

     - Events saved to a Storage Account for analysis later.

     - Export data using Log Profiles with Log Analytics.

     Diagnostic logs can be enabled on almost every Azure resource. These logs include Windows event system logs, Storage logs, Key Vault audit logs, and Application Gateway access and firewall logs. Diagnostic logs can be written to a centralized and encrypted Azure storage account for archival or to event hubs or stream analytics for further processing. Users can configure the retention period, up to 730 days, to meet their specific requirements.

     Azure Sentinel can ultimately be a replacement for on-premises SIEM systems with built in support for Azure resources through Log Analytics workspaces.

     Azure Security center can be utilized to enforce policies and compliance against Azure resources.

11. What certifications does Azure have and can Azure hosted applications meet the US and European compliance goals?

    - Certifications

        a. U.S. Specific Certifications

         -   Cloud Security Alliance CCM

         -   HIPAA BAA

         -   FedRAMP

         -   FISMA

         -   FBI CJIS

         -   FERPA

         -   FIPS 140-2

         -   FDA 21 CFR Part 11

        b. International Certifications

         -   PCI DSS Level 1

         -   ISO 27001 / 27002

         -   SOC 1 / SSAE 16 / ISAE 3402 and SOC 2

         -   EU Model Clauses

         -   United Kingdom G-Cloud / IL2

         -   Australian Government IRAP

         -   Singapore MTCS Standard

         -   China Cloud Computing and Policy Forum (CCCPPF)

         -   MLPS (China)

    Azure hosted applications can meet compliance goals is designed and maintained properly.

12. Is Azure flexible enough to support data sovereignty needs and issues like those referenced in GDPR articles?

     Microsoft was among the first cloud providers to sign a set of EU "model clauses," which are standard contractual clauses that govern the international transfer of personal data. When included in service agreements with data processors, the model clauses assure customers that appropriate steps have been taken to help safeguard personal data, even if data is stored in a cloud-based service center located outside the EEA. In affirming these, a data processor is affirming that they are meeting the data protection standards set forth by the EU Directive on Data Protection.

     Microsoft has UK Government IL2 certification. UK Government Departments require products and services to be accredited to IL2 or IL3. These terms, Business Impact Levels 2 or 3, essentially require the telecoms operator or service provider to pass an audit based on ISO 27k as additionally extended by the *NGN Good Practice Guide*.

13. How can we ensure continued SOC 1 and SOC 2 compliance?

     To maintain SOC 1 and SOC 2 compliance, there are auditing requirements for the customer. To address this need, Microsoft makes certificates and audit reports available in support of the customer\'s reports and certifications.

     - ISO/IEC 27001:2005 Audit and Certification certificates are publicly available.

     - SOC 1 and SOC 2 reports are available under NDA to customers to meet auditing requirements.

     Customers cannot audit the data center. However independent audits and certifications are shared instead of individual customer audits. These certifications and attestations accurately represent how Microsoft obtains and meets security and compliance objectives, and serve as a practical mechanism to validate Microsoft promises for all customers.?? Allowing potentially thousands of customers to audit Microsoft services would not be a scalable practice and might compromise security and privacy.?? The independent third-party validation program includes audits that are conducted on an annual basis to verify Azure security controls.

14. Does Azure permit penetration testing as a part of a security assessment?

     Yes, and as of mid-2017, you no longer need to get approval to execute these tests, but you can still notify Microsoft if you are planning to do so.

## Customer quote (to be read back to the attendees at the end)

"We are moving into the secure digital world with Azure, helping our agents ensure the security and protection of client personal information and protecting them from the unpredictable and unforeseen events of life."

Jack Tradewinds, CIO of Contoso Insurance Ltd.
