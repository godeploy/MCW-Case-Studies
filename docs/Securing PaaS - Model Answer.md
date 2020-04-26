---
lab:
    title: 'Securing PaaS - Model Answer'
---


# Case Study: Securing PaaS - Model Answer


## Preferred solution

*High-level architecture*

1.  Without getting into the details, (the following sections will address the particular details), diagram your initial vision for handling the top-level requirements for the gift card website, gift card API, and the storage for customer profiles and transactions. You will refine this diagram as you proceed.

    ![The High-level architecture diagram has two sides: Public internet, and Azure. The solution shows securing customer identity, core website components, using Azure function for reviews, using DevOps for deployments, and utilizing Azure Security Center for monitoring security health.](images/Whiteboarddesignsessiontrainerguide-SecuringPaaSimages/media/image3.png "High-level solution architecture diagram")

2.  What data would you consider sensitive in this scenario? Is there a compliance standard that you would recommend Fourth Coffee consider building their solution against?

    The gift card information, while not strictly speaking a credit card, is a financial instrument and should be secured following a similar approach to how credit card holder data is protected. You should recommend Fourth Coffee design their solution with the goal of achieving PCI DSS 3.0 compliance in mind. This would make for a secure solution today and down the road if Fourth Coffee decides they want to enable customers to re-load their cards online with payment by credit card, their solution is well designed to securely handle card holder data.
    
    In support of this, Microsoft provides a blueprint for Payment Processing for PCI DSS-compliant environments, which would serve as useful reference in designing the solution: <https://docs.microsoft.com/en-us/azure/security/blueprints/payment-processing-blueprint>

*Securing customer identity*

1.  Fourth Coffee mentioned a desire to improve how they manage and store the identities of their customers, for use when granting them access to their online profiles. What approach would recommend they take to modernize their identity management?

    They should "outsource" the identity management function to Azure Active Directory B2C instead of storing usernames and passwords in the Fourth Coffee SQL Database.

2.  Would the approach you suggested still allow customers who do not want to use social accounts to create a login with Fourth Coffee?

    Yes, AAD B2C enables applications to authenticate social accounts (e.g., Facebook, Google, LinkedIn), Enterprise accounts (e.g., integrated using open standard protocols, OpenID Connect or SAML) as well as local accounts (e.g., email address and password or username and password). Customers who do not want to use their social accounts could use local accounts in AAD B2C.

3.  One of Fourth Coffee's biggest frustrations with their existing identity management approach is that when customers forget their password, they typically must contact support- which is an expensive interaction for Fourth Coffee. How would your proposed solution improve this situation? What would be the process?

    AAD B2C provides a self-service password reset feature for users who have setup local accounts. Once self-service password reset is enabled in Active Directory, when a user arrives at a login screen a link labeled "Can't access your account?" appears. The consumer needs to enter their email address. If the email address belongs to a valid user, that address will be sent an email with a link allowing the recipient to reset the password.

*Securing the core website components*

1.  The Fourth Coffee website should be publicly accessible, but the gift card API should not be accessible from the Internet. Additionally, all traffic flowing through to the website should pass through some kind of firewall to guard against malicious requests like cross-site scripting and SQL injection, and the website itself should never be directly accessible in any other way. How would you accomplish this?

    They should host the website within a Web App and the API within an API App just like they are currently doing, except that these App Services should be hosted within an App Service Environment (ASE). The ASE itself provides a hosting infrastructure that is deployed within a subnet of a Virtual Network (VNET). The key changes in this case, relative to the multi-tenant App Services case, are that the ASE exposes only an Internal Load Balancer (ILB) endpoint for access to the App Service instances it hosts. Then, an Application Gateway of the Web Application Firewall SKU should be deployed between the public IP address used by Internet clients to access the website and the ILB ASE endpoint. The ASE and the Application Gateway are deployed to two different subnets. Network Security Groups (NSG) are configured so that the subnet that contains the ASE only allows access from the subnet which contains the Application Gateway.

2.  Fourth Coffee does not like the fact that their existing SQL Database deployment, which contains sensitive customer personally identifiable data (PII), has an endpoint that is accessible on the Internet. They have the firewall rules configured on the SQL Database Server to restrict access, but they would prefer a solution that does not have such a publicly exposed endpoint. What would you suggest and how would they accomplish this?

    With VNET service endpoints for SQL Database, Fourth Coffee can make their SQL Database available within a VNET subnet and nowhere else, including removing the public endpoint. First, they would need to create a subnet for use by their database. Then within the VNET configuration they would need to add an endpoint for Microsoft.Sql and select the subnet in which they will allow access to the database. Then they would need to configure the SQL server (which encapsulates the SQL Database) by selecting the VNET and subnet in which access should be allowed. Finally, they should create restrict access to the subnet by configuring an NSG on it. In this case, the NSG would need to allow access from the subnet containing the ASE.

3.  Fourth Coffee is storing customer purchase history for the long term in Azure Storage blobs. Like their previous concern, they would like to know if they could remove the public endpoint access from Azure Storage, since the transaction data reflects sensitive financial transaction information.

    They can use VNET service endpoints for Azure Storage, taking a similar approach as they did for SQL Database. In this case, however, they will need to configure the VNET with an endpoint for Microsoft.Storage and select the subnet. Then in the Azure Storage account, they configure the firewalls and virtual networks feature to select the VNET and subnet from which access to the Storage account should be allowed.

4.  The existing API App is deployed with the connection string information to the SQL Database saved in the web.config. Fourth Coffee would like to improve the handling of sensitive configuration. What would you recommend they do and how would they need to change their app to support your recommendation?

    Fourth Coffee should deploy Azure Key Vault. For maximum security, they should select a Premium tier which will safeguard secrets like connections strings using specialized hardware security modules (HSM). Access to Key Vault is secured using AAD principals, and Fourth Coffee would need to create a principal in AAD to represent the API App. In the configuration of the app, they would need to provide the Client ID and Client Secret from the AAD principal, as well as the Secret URI that identifies the secret in Key Vault. They would also need to add additional code that uses the Client ID, Client Secret and Secret URI to retrieve the connection string from the Key Vault before it can be used to connect to SQL DB.
    
    The proper configuration of Key Vault involves setting ARM RBAC on the management plane of the Key Vault for the solution security group, and enabling proper Key Vault access control for developers against the data plane. The solution audit group would also need data plane permissions to list secrets and keys, but not to view their contents or modify them.

*Enabling reviews*

1.  Fourth Coffee wants to be certain that their mini-platform for reviews can scale and are less concerned about securing the data (other than securing access to edit the reviews appropriately). How would you suggest Fourth Coffee deploy the logic for managing and navigating reviews, how would reviews (which could grow to become very large data sets) be stored?

    Given that the reviews are not sensitive data akin to the transaction data used by the gift cards, they do not need to be managed in the same scope of security. The logic for querying and managing reviews could be hosted within Azure Functions, and these would not need to be hosted within the ASE used by the Gift Card API. Instead these could be the multi-tenant Functions deployed using a Consumption Plan that would enable rapid scalability for executing operations. The actual review data would be stored in Cosmos DB.

2.  Fourth Coffee would like users to be able to search across reviews with free-form text, but also narrow their search by product, date range and number of stars. How would you enable end-users to search thru reviews?

    Full text searchability of the reviews by free text and product name, as well search by facets such as date range or number of stars would be enabled by indexing the reviews data stored in Cosmos DB using Azure Search. An indexer can be configured that periodically updates the Azure Search index with new reviews that appear in Cosmos DB.

3.  How would you secure access to the services you propose in support of reviews?

    Access to Azure Function, Cosmos DB and Azure Search are secured with keys. These keys should be stored as secrets within Key Vault.

*Securing DevOps*

1.  Fourth Coffee is trying to think about the complete security of their solution, and one challenge they face is in securing how developers interact with production systems for the purposes of deployments as well as troubleshooting. How would restrict deployments to production Web App or API app from occurring across Internet available endpoints?

    When deploying code to a Web App or API App, developers are deploying using MSDeploy to endpoints of the form \*.scm.myapp.com. Given that all endpoints in the ASE are internal, this is not publicly available for the ASE described previously. What makes the SCM endpoint available via the Internet is the configuration of the Application Gateway, which evaluates host headers to direct traffic to the appropriate ASE backend. You should configure the Application Gateway to route requests against the SCM.myapp.com to another URI (such as a not-found or not allowed page) so that they do not reach the back-end ASE.

2.  With this safeguard in place, how will developers perform their deployments?

    There are two ways developers (or CI/CD tools automating the process) can be provided with access to deploy updates to production code. One option is for the developers to work across a site-to-site VPN that connect their office with the VNET in Azure.
    
    Another option is for a jumpbox to be deployed within the VNET that developers can use Remote Desktop Protocol (RDP) to access when they need access to production resources or to perform deployments. This VM would only be started when developers require production access, and the ability to start this VM could be restricted using Role Based Access Controls (RBAC) so that only authorized managers are allowed to enable this access. Fourth Coffee could configure the VM to automatically shut down daily, so if it is accidentally left on when developers leave for the night it would turn off and not incur further expense.
    
    In either case, the NSG's applied to the subnet containing the ASE and the VNET Service Endpoints (for SQL DB and Azure Storage) would need to explicitly allow access from the subnet representing the on-premises network or the subnet containing the jumpbox.

3.  They would also like to restrict what developers can see in the database (in terms of sensitive data like gift card data), what options do they have for this?

    For the data stored in SQL Database, Fourth Coffee should consider configuring dynamic data masking on the sensitive data, enabling developers to troubleshoot without fully seeing sensitive values. They could also consider Column Level Encryption, which restrict the set of rows a developer would be allowed to see as the result of a query.

*Monitoring security health*

1.  What services would you suggest Fourth Coffee utilize in order to monitor the general health of the solution?

    Azure Monitor should be used all monitoring data from Azure services, including performance metrics and events for PaaS services, and management activities involving Azure Resource Manager (e.g., that captures when a resource is created, updated or deleted).
    
    Application Insights to instrument applications like the website and API logic to collect performance monitoring data and user analytics telemetry.
    
    Log Analytics to ingest log and metric data from Azure Monitor and App Insights, and provide tools to query, analyze and dashboard the integrated data.

2.  What service would you suggest Fourth Coffee utilize to monitor the security health of the solution? What would it provide for their solution?

    They should use Azure Security Center which will monitor their configuration and make security recommendations.
    
    For their Web Apps: Security Center will monitor for the presence of a web application firewall (WAF) and collect logs from the WAF.
    
    For their SQL DB: They should turn on auditing and threat detection on their SQL Database as threats will show up in Azure Security Center. Security Center will also flag any databases that do not have Transparent Data Encryption enabled.

## Checklist of preferred objection handling

1.  Can we really set it up so our developers' applications have access to the connection strings, keys and other secrets at run time, without enabling the developers themselves to access this sensitive data?

    Yes, for example, you can configure a situation where the web.config of the app refers to a secret (like a connection string) in Key Vault, but to get the value at runtime requires an AAD principal (e.g., it is an application service principal) that is not that of a developer.

2.  We've been told over and over again that Azure's services, like Azure SQL Database and Azure Storage, must always have a public endpoint. Is that really true?

    This used to be true. There is new functionality called VNET Service Endpoints that enables you to remove public endpoints for Azure SQL Database and Azure Storage and instead only have endpoints available within NSG secured VNET subnets that you indicate.

3.  We heard the announcement about Managed Service Identity. We recognize it is in preview now, but we would like to understand how it would improve the security of the solution you are recommending to us.

    Managed Service Identity (MSI) creates an Azure service instance in Azure Active Directory that you can then manage like any other service principal. Each MSI enabled service has an internal mechanism to acquire its authentication token from AAD that it can present when identifying itself to the downstream systems and services it will call. For example, an Azure Function with an MSI will first make a request to a local endpoint to get the authentication token and then it can call Key Vault using that token to acquire a secret like a connection string. This feature is currently in preview and there is a limited set of services that provide a Managed Service Identity: Azure Virtual Machines, Azure App Service, Azure Functions, and Azure Data Factory v2.

4.  We need to be certain that all of our data is encrypted when it is stored on disk, is that possible with the PaaS services you are recommending?

    Yes. Azure Storage and Azure SQL Database provide the ability to toggle on transparent data encryption that encrypts data as it is written to the underlying storage. Cosmos DB and Search do not expose a mechanism to disable encryption, but all data is also automatically encrypted when written using Microsoft managed keys.

## Customer quote (to be read back to the attendees at the end)

"I'm thrilled to open up these new gift card capabilities to our loyal customers, knowing that they are built with productivity boosting solutions on a foundation that is secure."

Victoria Gray, CEO of Fourth Coffee

