---
lab:
    title: 'Building a resilient IaaS architecture - Model Answer'
---

# Building a resilient IaaS architecture- Model Answer


**Solution**

Azure provides two approaches to high availability for VM-based applications: availability sets and availability zones. Availability sets provide an SLA of 99.95%, whereas availability zones provide an SLA of 99.99%. To achieve a composite SLA across all application tiers in excess of 99.95%, the application must use availability zones.

This will require migrating from the West Central US region to one of the [4 US regions that supports Availability Zones](https://docs.microsoft.com/azure/availability-zones/az-region). The closest supported regions to West Central US are West US 2 and Central US. Central US is recommended, being closer to Contoso's Cheyenne HQ.

![image](https://user-images.githubusercontent.com/25365143/136542028-e3758e32-14f9-4f05-8e10-c2fe3f9da065.png)

Using Availability Zones will require Standard-tier load-balancer and Standard-tier public IP addresses to be used.

Considering each tier in turn:

*Web Tier*

-  Web VMs should be straightforward to deploy into Availability Zones.

-  The Web VM load-balancer currently uses a TCP health probe. This should be changed to a HTTP probe, preferably pointing to a custom health check page that verifies the application is working as expected, including access to the database.

-  With the change from the Basic to Standard LB SKU required for availability zone, the behavior of the load-balancer in the event of all probes failing (often a result of misconfiguration) changes so TCP flows continue.

-  Migrating to VM Scale Sets would allow the Web tier to auto-scale based on demand (rather than the current approach of manually adding VMs and forgetting to remove them later). VM Scale sets support regional deployment spanning availability zones.

*SQL Server(s)*

-  A second SQL server should be deployed to the primary site, forming a SQL Server Always On Availability Group. This should be configured with synchronous replication and automatic failover. A storage account can be used as the 'cloud witness'.

-  The servers should be behind an internal Azure load balancer with Direct Server Return (DSR) enabled.

-  The SQL Servers will use premium managed disks with a database and log files on separate disks. The TempDB will be housed on the local host SSD drive and split to match the number of cores in the VM.

    SQL Servers will have three drive letters: C:\\, F:\\ and G:\\

    | Drive  | Type      |  Purpose | 
    | ------- | ----------- | ----------------------------------------------------------- |
    | C:\\   | S10       | OS  | 
    | D:\\   | Local SSD  | TempDB broken into multiple files (match number of cores) |
    | F:\\   | P20        | SQL Database Files | 
    | G:\\   | P20        | SQL Database Log Files |

    >**Note:** Never use the E:\\ drive on an Azure VM as some Azure Regions have Host machines that contain DVD Drives.

*AD DS Domain Controllers*

-  AD DS Domain Controller should be deployed into separate availability zones, with two VMs in the Central US region. These servers should be synchronized with the on-premises AD via the VPN. The virtual network DNS server settings should be updated to point to these servers.

-   Storing the AD files on a data disk with caching set to None will keep the AD DS database and SYSVOL from any potential corruption due to caching.

-   AD native Replication across regions allows for disaster recovery from region wide outage should the need arise and faster recovery of the AD DS database.


2. How can you improve the reliability for the Contoso branch office VPN connections?

-  Identify and eliminate as many single-points-of-failure as you can.

**Solution**

The current VPN has single points of failure at the on-premises VPN gateway (RRAS server), the Azure Virtual Network Gateway, and the ISP connection from the on-premises site to the Internet. All three of these single points of failure can be eliminated.

![image](https://user-images.githubusercontent.com/25365143/136542057-6f34e517-c501-452f-bb94-ba8f7c4c9902.png)

*On-premises gateway:*

-   The RRAS VPN servers can be deployed in a cluster ([here's how](https://docs.microsoft.com/windows-server/remote/remote-access/ras/cluster/deploy-remote-access-in-cluster)). Alternatively, the servers could be upgraded to dedicated hardware VPN devices.

*ISP Internet connection:* 

-   You need to create multiple S2S VPN connections from each on-premises VPN device to Azure. When you connect multiple VPN devices from the same on-premises network to Azure, you need to create one local network gateway for each VPN device, and one connection from your Azure VPN gateway to the local network gateway.

-   The local network gateways corresponding to your VPN devices must have unique public IP addresses in the \"GatewayIpAddress\" property. These IP addresses can be provided by different ISPs.

-   BGP is required for this configuration. Each local network gateway representing a VPN device must have a unique BGP peer IP address specified in the \"BgpPeerIpAddress\" property.

-   The AddressPrefix property field in each local network gateway must not overlap. You should specify the \"BgpPeerIpAddress\" in /32 CIDR format in the AddressPrefix field.

-   You should use BGP to advertise the same prefixes of the same on-premises network prefixes to your Azure VPN gateway, and the traffic will be forwarded through these tunnels simultaneously.

-   Each connection is counted against the maximum number of tunnels for your Azure VPN gateway (max 30 for the non-Basic SKUs).
  
*Azure VPN Gateway:*

- The VPN gateway should be configured for 'active-active' mode rather than the default failover mode.

-  An 'Az' SKU should be used to provide protection against failure of individual data centers in the chosen Azure region.

3. Describe how you will implement a disaster recovery solution for the claims application.

-  Which secondary Azure region will you use?
-  How will the DR be configured? Consider each component (web, database, AD, VPN)
-  What process is required to fail over to the secondary site? Consider each component (web, database, AD, VPN). Are all process steps automated?
-  What is the impact on agents using the application? How are they routed to the DR site after failover?
-  Does the solution meet the RPO and RTO requirements?

**Solution**

![image](https://user-images.githubusercontent.com/25365143/136542083-fdd014d5-7903-4899-96ae-8c72c70deb47.png)

*Which secondary Azure region will you use?*

-  The primary region chosen above was Central US (changed from West Central US to gain Availability Zone support). The secondary region should be the appropriate region pair, in this case East US 2.


*How will the DR be configured? Consider each component (web, database, AD, VPN)*.

-  Web server VMs: These should be configured for replication and failover using Azure Site Recovery (ASR). The underlying 'landing zone' infrastructure in the secondary region (network, load balancer, etc.) should be provisioned in advance, since ASR will only fail over the Web VMs themselves.

-  SQL Server: There are two approaches for extending the SQL Server infrastructure to the secondary region.

   -  Additional SQL Server VMs can be provisioned, and configured as asynchronous replicas in the same Always On availability group as the primary site. During failover, a 'forced failover' will promote one of these secondary servers to primary, and the other server should be reconfigured to replicate synchronously from this new primary.
   -  Alternatively, SQL Server VMs in the secondary site can be configured as a second Always On Availability Group, with synchronous replication between servers in the secondary site and asynchronous replication between primary and secondary sites (this is known as a 'distributed Availability Group'). This is more complicated to set up.
  
-  Domain Controller VMs: A pair of domain controller VMs should be provisioned into availability zones in the secondary site, replicating the set up in the primary site. These are required in the event of a failover, and should be kept running rather than only being provisioned in the event of a failover, so they are always up-to-date.

-  VPN: The secondary site should implement VPN connections similar to the primary site. This is required for domain controller replication and so that branch offices can access the secondary site in the event of a failover.

*What process is required to fail over to the secondary site? Consider each component (web, database, AD, VPN). Are all process steps automated?*

-  The AD and VPN components are pre-provisioned in both primary and secondary sites in an active-active configuration. We will therefore focus on the Web VMs and SQL Server database.

-  Provisioning of the Web VMs in the secondary site is carried out by ASR during failover. Once provisioned, the failover VMs must be integrated with the load-balancer in the secondary region. ASR supports load-balancer configuration as part of the failover settings, however, this only supports internal load-balancers. For a public load-balancer, adding the VMs to the backend pool is an additional post-failover step.
  
-  For the SQL Server database, we will assume the first of the two approaches described earlier is used, with secondary replicas in the failover site replicating asynchronously as part of the same Always On Availability Group. At the start of the failover, one of the secondary replicas must be promoted to primary ('forced failover') and the other configured to act as a synchronous replica of this primary.

-  These failover steps can be automated using Azure Automation Runbooks. The Automation Account should be provisioned to a separate Azure region, so it is not impacted by the failover in any way.

-  The overall failover process should be orchestrated using an ASR Recovery Plan. This plan defines the various failover steps, including scripts/runbooks to execute and VM failover actions. For the claims application, the sequence is:
    - First, make the secondary SQL as Active using automation script.
    - Second, failover the web servers and start the machines.
    - Third, configure the load balancer for the web servers in the secondary region.

    ![image](https://user-images.githubusercontent.com/25365143/136542105-3bdd6f3e-2e16-4c89-a162-74b3970a5b89.png)
    
*What is the impact on agents using the application? How are they routed to the DR site after failover?*

The claims application is an Internet-facing application. The public IP address of the application will change during the failover between Azure regions. Agents using the application must be directed to the new IP address. This can be achieved in one of three ways.

-  The DNS entry for the claims application can be updated as a custom step in the ASR recovery plan. The DNS zone can be hosted in Azure DNS, or in a third-party DNS provider (with API access). Be sure to use a suitably short TTL on the DNS entry (both before and after failover), otherwise DNS caching in external systems will continue to route agents to the (failed) primary site, and impair your ability to fail back to the primary once it recovers.
-  Azure Traffic Manager can be used with the '[Priority](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-routing-methods#priority-traffic-routing-method)' traffic routing method to automatically direct users to the secondary site deployment when the primary site fails (detected via health probes)
-  Azure Front Door can be used as a proxy. Front Door will receive the agent request and forward to either the primary or secondary site based on the monitoring status.

*Does the solution meet the RPO and RTO requirements?*

-  The recovery time achieved depends on the time required for ASR to execute the failover. For a VM, ASR has an [RTO SLA of 2 hours](https://docs.microsoft.com/azure/site-recovery/azure-to-azure-common-questions#failover), although usually a recovery is executed in minutes. You can view the recovery plan recovery time for test failovers in the Azure portal. This comfortably exceeds the RTO requirement of 4 hours specified by the business.

-  Achieving the 6-hour recovery point objective depends on the replication of data to the secondary site. Of primary concern is the SQL Server database, although the web VMs should also be considered in case they store any state locally. Both ASR replication and SQL Server asynchronous replication occur continuously, and should give an RPO far shorter than the 6 hours specified by the business.

-  Automating the recovery plan using ASR and Azure Automation is critical to keeping the RTO as short as possible. It has the added advantage of making the recovery process more testable and less prone to human error.

4. How will you protect both VMs and databases from data corruption or accidental deletion?

-  Describe both the solution and the recovery process.

**Solution**

*Azure Virtual Machines*

-  Azure Backup provides a comprehensive backup solution for Azure VMs. To enable Azure Backup, create a Recovery Services Vault, configure the storage settings (geo-redundancy and soft delete), then add VMs either from the Azure Backup or VM experiences in the Azure portal. It can also be configured via the command line. Encrypted disks are supported (ensure the backup service has the required access to the Key Vault). A retention policy determines how long the daily, weekly, monthly and yearly backups are kept.

-  Azure Backup supports several recovery options. Individual files can be recovered by mounting the backup as a file share. Alternatively, the VM disk(s) can be restored in-place (same VM), or as a new VM (the Availability Zone is preserved). You can also restore only the disks, and build the VM yourself from those disks. This is required in a [wide range of scenarios](https://docs.microsoft.com/azure/backup/backup-azure-arm-restore-vms#restore-vms-with-special-configurations).


*SQL Server in Azure Virtual Machines*

-  Azure Backup for SQL Server provides a solution that requires zero-infrastructure: no complex backup server, no management agent, and no backup storage to manage. Azure Backup provides centralized management for your backups across all servers that are running SQL Server, or even different workloads. Businesses can define their backup schedule and retention policy based on their LTR and compliance needs, do point in time restores and automatically detect and protect any new database that gets added in the future. This will help meet Contoso their RPO of 15minutes. 
      
	For details on Azure Backup for SQL Server IaaS VMs, see <https://docs.microsoft.com/azure/backup/backup-azure-sql-database>
  
-  Alternatively, you can also use SQL Server Managed Backup to Azure. It manages and automates SQL Server backups to Microsoft Azure Blob storage. 

    For details on SQL Server Managed Backup see <https://docs.microsoft.com/sql/relational-databases/backup-restore/enable-sql-server-managed-backup-to-microsoft-azure?view=sql-server-2017> 
      
5. How will you monitor and alert on Azure VMs metrics? Does this approach extend to SQL monitoring? What about backup monitoring? 

**Solution**

*VM Monitoring*

-  Azure Monitor for VMs provides in-depth monitoring for Azure VMs, VM scale sets, and non-Azure servers running on-premises or in other clouds.

-  Azure Monitor for VMs is a Log Analytics-based solution. As such, initial on-boarding requires installing the Log Analytics agent and registering it to send data to a Log Analytics workspace. 

-  In addition to the Log Analytics agent, for full functionality the dependency agent should also be installed. This captures process and network connection data from the VM enabling the inbound and outbound VM dependencies to be mapped. This is an optional feature which may not be necessary on all servers.

-  Both the Log Analytics agent and the dependency agent can be deployed to Azure VMs using Azure policy, to automate deployment across large VM footprints.

-  Once the agents are installed, the final deployment step is to configure the VM insights solution on the Log Analytics workspace. This is straightforward from the VM Insights blade in Azure Monitor; it can also be automated via a template.

-  Once data is gathered to Log Analytics, it can be used for a wide variety of monitoring scenarios. Azure monitor for VMs includes a number of pre-defined reports, such as 'top N' reports by various metrics such as CPU, memory and disk.

-  Log-based and metric-based alerts can be implemented using Azure Alerts.

*SQL Monitoring*

-  SQL metrics and logs can be gathered via Log Analytics using the above mechanism, by simply configuring the metrics required. 
  
-  DB experts should be engaged to define the precise logs and metrics required, and how alerts should be configured (details are out of scope for this exercise).

*Azure Backup*

-  Azure Backup has recently delivered two new monitoring solutions (as preview): Backup Explorer and Backup Reports. Both solutions are built using Azure Monitor Workbooks

   -  Backup Explorer provides a near-real-time dashboard of backup health. It is powered by the Azure Resource Graph, and hence does not require any additional diagnostics configuration, storage accounts, or Log Analytics workspaces. It is always available, for all Recovery Services Vaults. It only supports data from the past 7 days.

   -  Backup Reports provide deeper analysis of backup performance, including storage space consumed and historical trends. This solution is built on Log Analytics. It requires the Recovery Services Vault to be configured to send diagnostic logs to the Log Analytics workspace, and data can take up to 24 hours to surface in the reports.

-  Backup alerts can be delivered in two ways: alerts configured directly within the Recovery Services Vault (email only), or alerts based on Log Analytics queries and Azure Monitor. The built-in alerts have several limitations. For this reason, Log Analytics/Azure Monitor alerts should be used. These have the additional advantage of supporting Action Groups.

-  Log-based alerts should be configured based on the Azure Backup 'V2' diagnostic data schema. Note that V1 schema is planned for deprecation.

-  This requires that each Recovery Services Vault is configured to send diagnostic data to a Log Analytics workspace.  Diagnostic logging can be configured using Azure Policy (see '\[Preview\]: Deploy Diagnostic Settings for Recovery Services Vault to Log Analytics workspace for resource specific categories.')

-  For more information, see [https://docs.microsoft.com/en-us/azure/backup/monitor-azure-backup-with-backup-explorer].

6. How can the PaaS implementation of the claims application achieve an equivalent level of resiliency?

- How is high availability provided by the Web Application and SQL Database? Can the SLA target be met?
- How will the PaaS solution recover from a complete failure of the primary Azure region? Can the RPO and RTO targets be met?
- How is backup implemented and executed?

**Solution**

*How is high availability provided by the Web Application and SQL Database?*

-  Azure Web Apps provide built-in high availability, natively within the platform. No special configuration is required.
  
-  Azure SQL Database also provides built-in high availability. There are two models:

   -  Standard availability (used by the Basic, Standard and General Purpose tiers) relies on a stateless compute layer backed by Azure blob storage as a resilient data layer. Heavy workloads can suffer performance degradation during failure due to cold cache process starts.
   -  Premium availability (used by the Premium and Business Critical tiers) uses technology similar to SQL Server Always On Availability Groups in a 3 to 4 node cluster. This does not suffer performance degradation during failures.

    Contoso will need to choose the appropriate tier depending on their performance needs. For more information see [High availability for Azure SQL Database and SQL Managed Instance](https://docs.microsoft.com/azure/azure-sql/database/high-availability-sla).

*Can the SLA target be met?*

-  Azure Web Apps (at the Standard tier or above) provide a [99.95% availability SLA](https://azure.microsoft.com/support/legal/sla/app-service/v1_4/). No special configuration is required. Availability zones are not supported for Web Apps.

-  Azure SQL Database supports a [99.99% availability SLA](https://azure.microsoft.com/support/legal/sla/sql-database/v1_4/) across a wide range of tiers (Basic, Standard, General Purpose, Premium, Business Critical). A 99.995% SLA is provided when using the Business Critical or Premium tiers configured with zone-redundant deployment.

-  The composite SLA may therefore fall *just* short of the 99.95% demanded by the business. This should be negotiated with the business team defining the SLA requirement. If no compromise is possible, the [Web App may be run in an active-active configuration across both primary and secondary regions](https://docs.microsoft.com/azure/architecture/reference-architectures/app-service-web-app/multi-region).

*How will the PaaS solution recover from a complete failure of the primary Azure region?*

- Web Apps may be run in a multi-region deployment, with either cold standby, hot standby, or active-active configuration. For details, see the link above.

- Azure SQL Database offers [active geo-replication](https://docs.microsoft.com/azure/azure-sql/database/active-geo-replication-overview), which provides readable secondary databases in up to 4 additional regions, with failover capabilities. In addition, [auto-failover groups](https://docs.microsoft.com/azure/azure-sql/database/auto-failover-group-overview?tabs=azure-powershell) can be used to enable automatic failover based on a user-defined policy.

*Can the RPO and RTO targets be met?*

Yes. See table.

| Service | RTO | RPO |
|:--------|-----|-----|
| Web App |  < 5 min*   |  n/a (stateless)   |
| SQL DB  | [1 hr (auto) / 30 sec (manual)](https://docs.microsoft.com/en-us/azure/azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview)   |  [5 sec](https://docs.microsoft.com/azure/azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview)  |
|         |     |     |

\* Time to start secondary web app and for endpoint failover via Traffic Manager or Front Door health probes.

*How is backup implemented and executed?*

-  The Web App is stateless; hence backup does not apply.

-  Both SQL Database and SQL Managed Instance use SQL Server technology to create full backups every week, differential backups every 12-24 hours, and transaction log backups every 5 to 10 minutes. The frequency of transaction log backups is based on the compute size and the amount of database activity.

-  When you restore a database, the service determines which full, differential, and transaction log backups need to be restored.

-  These backups enable databases to restore to a point in time within the configured retention period. The backups are stored as RA-GRS storage blobs that are replicated to a paired region for protection against outages impacting backup storage in the primary region.

-  If your data protection rules require that your backups are available for an extended time (up to 10 years), you can configure long-term retention for both single and pooled databases.

- For full details, see [Automated backups - Azure SQL Database & SQL Managed Instance](https://docs.microsoft.com/azure/azure-sql/database/automated-backups-overview).

### Pricing

1. Provide an estimate of the costs associated with each aspect of your solution.

-  Be sure to cover all aspects of the design, including the primary site, DR solution, backup solution, VPN, and monitoring costs.
-  Include a comparison of the IaaS solution and the PaaS solution.
-  Have you included all appropriate cost-saving measures?

**Solution**

Pricing Azure solutions is a complex task. The example solution below includes many assumptions, for example on resource size and bandwidth consumed. These need to be validated with Contoso.

*Infrastructure*

| Component     | Site          | Details / Assumptions                                           | Monthly Cost (USD) |
|:--------------|:--------------|:--------------------------------------------------------------------------|---------:|
| DC VMs        | Central US    | 2 VMs, Windows, D2s_v3, 1 year reservation, 2x Premium SSD 128 GiB per VM | $314.50  |
| DC VMs        | East US 2     | 2 VMs, Windows, D2s_v3, 1 year reservation, 2x Premium SSD 128 GiB per VM | $289.67  |
| VPN Gateway   | Central US    | VpnGw2AZ, 730 hours, 0 additional tunnels, 100 GB traffic                 | $419.98  |
| VPN Gateway   | East US 2     | VpnGw2AZ, 730 hours, 0 additional tunnels, 100 GB traffic                 | $419.98  |
| Log Analytics | Central US    | 3GB per VM, 180 day retention                                             | $  7.08  |
| Log Analytics | East US 2   | 3GB per VM, 180 day retention                                             | $  7.08  |
| Alert Rules   | Central US    | 2 VMs x 10 metrics + 5 log signals @ 5 minutes                            | $  9.50  |
| Alert Rules   | East US  2  | 2 VMs x 10 metrics + 5 log signals @ 5 minutes                            | $  9.50  |
| **Total** | | | **$1,477.30** |
| | | | |

*Claims Application - Primary Site and BCDR*

| Component     | Site          | Details / Assumptions                                           | Monthly Cost (USD) |
|:--------------|:--------------|:--------------------------------------------------------------------------|---------:|
| Web VMs       | Central US    | 2 VMs, Windows, D4s_v3, 1 year reservation, 1x Premium SSD 128 GiB per VM | $510.56  |
| SQL VMs       | Central US    | 2 VMs (1x primary + 1x secondary), Windows, E4as_v4, 1 year reservation, SQL Enterprise, 2x Premium SSD 512GiB per VM      | $1,906.02 |
| Bandwidth     | Central US    | 500 GB                                                                    | $ 43.07  |
| Log Analytics | Central US    | 3GB per VM, 180 day retention                                             | $ 27.96  |
| Alert Rules   | Central US    | 2 VMs x 10 metrics + 5 log signals @ 5 minutes                            | $ 11.50  |
| VM Backup     | Central US    | 2x Web VMs, 80GB each, GRS, low churn, 30 daily/26 weekly/24 monthly/3 yearly RPs, steady state | $ 54.69 |
| SQL Backup    | Central US    | 300 GB, GRS, high churn, 30 daily/6 weekly/12 monthly RPs, steady state | $695.28 |
| ASR           | East US 2   | 2 instances                                                               | $ 50.00  |
| SQL VMs (DR)  | East US 2    | 2 VMs (1x primary + 1x secondary), Windows, E4as_v4, 1 year reservation, SQL Enterprise, 2x Premium SSD 512GiB per VM      | $1,613.64 |
| Traffic Manager | Global      | 10M DNS queries, 2 endpoints                                              | $  6.12  |
| VNet          | East US 2     | Global peering bandwidth for SQL replication to East US 2, 200GB          | $ 14.00  |
| Log Analytics | East US 2   | 3GB per VM, 180 day retention                                             | $  7.08  |
| Alert Rules   | East US 2   | 2 VMs x 10 metrics + 5 log signals @ 5 minutes                            | $  9.50  |
| **Total** | | | **$4,949.42** | 
| | | | |

>**Note:**
>-  SQL Server Enterprise licensing is required for Always On Availability Groups
>-  Each SQL Server Enterprise license includes one additional license for a DR server. Hence of the 4 SQL VMs, only 2 need the SQL license. This is a substantial saving.
>-  Data between Availability Zones in the same region will be billed from Feb 1, 2021

*PaaS Solution*

| Component     | Site          | Details / Assumptions                                           | Monthly Cost (USD) |
|:--------------|:--------------|:--------------------------------------------------------------------------|---------:|
| Web App       | Central US    | 2 instances, S3 tier                                                      | $584.00  |
| Web App (DR)  | East US 2     | As above                                                                  | $584.00  |
| SQL Database  | Central US    | Single DB, General Purpose, 4 vCores, PAYG, 2 instances, 500GB. Backup: RA-GRS, 1TB point-in-time, 300GB average backup size, 26 weeks/12 months/3 years retention  | $1,719.54 |
| SQL Database (DR)  | East US 2     | As above, no backup                                                | $1,530.25  |
| Traffic Manager | Global      | 10M DNS queries, 2 endpoints                                              | $  6.12  |
| VNet          | Central US    | Global peering bandwidth for SQL replication to East US 2, 200GB          | $ 14.00  |
| Bandwidth     | Central US    | 500 GB                                                                    | $ 43.07  |
| App Insights  | Central US    | 100GB/month, 5 multi-step web tests                                       | $312.20  |
| Alert Rules   | Central US    | 20 metrics + 10 log signals x 5 minutes                                   | $ 17.00  |
| **Total** | | | **$4,810.17** | 
| | | | |

This compares with a monthly total for the IaaS implementation of **$4,949.42** (excluding infra costs, on the assumption these are still required for other applications).

The PaaS implementation is roughly the same price. However, a cost-only comparison does not take into account the considerable additional benefits of a PaaS-based approach, e.g. reduced management overhead. Overall, the PaaS solutions offers significantly better value.

*Cost-saving measures*

For the above estimates, note:

-  1 year reserved VM instances included (could extend to 3 years for extra savings).
-  Does not use Hybrid Benefit (check existing licenses, for both Windows and SQL).
-  Does not use Log Analytics [Capacity Reservation](https://docs.microsoft.com/azure/azure-monitor/platform/manage-cost-storage#pricing-model).
-  PaaS solution does not include reservations. A 1-year reservation for the SQL Database would reduce monthly costs, but must be paid up-front.

## Checklist of preferred objection handling

1.  Contoso are uncomfortable with any situation that assumes the cloud provider will handle their fail-over.

    -  ASR provides full control over the failover process, including the ability to include custom steps.
    -  For the PaaS implementation, some aspects (such as in-region HA) are handled natively by the platform. Cross-region DR can be managed either manually, or automatically using services such as Traffic Manager and Front Door for the customer endpoint and SQL Database auto-failover groups.

2.  Contoso want to know their BCDR and backup solutions are secure.

    -  All of the traffic and data used for all Azure BCDR features is secured both at rest and in-transit. As a result, there is no difference in this data and any other data that is running or stored in Azure.

3.  Contoso also want to be able to test both the BCDR and Backup solutions regularly.

    -  ASR allows for non-disruptive test failovers to validate the failover process.
    -  Backups can be restored to a parallel cloud environment to verify their availability and integrity.


## Customer quote 

"By using Azure, we can build out resiliency for all aspects of our environment. It allows for infrastructure, networking, web applications, AD, and other items to be redundant and highly available. With some planning and deployment of resilient resources, I envision our LOB apps and websites will no longer be impacted by outages."

---Lewis Franklin, head of infrastructure and enterprise operations, Contoso
