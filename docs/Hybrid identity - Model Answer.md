---
lab:
    title: 'Hybrid identity - Model Answer'
---

# Hybrid identity - Model Answer

## Preferred solution

*Requirements Recap*

1.  Remote users must be able to sign into their devices by using their Active Directory credentials.

2.  Existing Active Directory user sign-in hours and password policies must be preserved (although allowed password values could be further restricted). 

3.  User sign-in experience should be simplified by minimizing the number of sign-in prompts and limiting the use passwords in lieu of more secure authentication methods. 

4.  User device configuration should be simplified by leveraging a mobile device management solution and roaming user-specific settings across multiple devices.

5.  Control access of users to applications and resources by relying on a combination of multiple conditions, including users group membership, state of the users' devices, and dynamically evaluated risk based on heuristics and globally collected security related telemetry.

6.  Users must be allowed to reset their own passwords. 

7.  Designated users should be able to temporarily elevate their privileges to manage other user accounts. All elevation events must be edited.

8.  Contoso remote users must be able to access on-premises Windows Integrated Authentication-based applications.

9.  Fabrikam users must be able to access on-premises Windows Integrated Authentication-based applications.

10.  Commercial applications developed by Contoso programmers must be made available to external customers with minimum overhead associated with identity management.

11.  Resiliency must be maximized whenever possible.  Users must not lose authentication capabilities or the ability to access on-premises applications.

12.  Infrastructure requirements must be minimized


*Architecting a hybrid identity solution*

1.  Using the features of Azure Active Directory and the requirements from the customer, how would you design a Hybrid Identity solution?

    The solution illustrates a wide range of benefits of implementing the hybrid identity model. In the current state:

      - Contoso is using a single-domain Active Directory forest using a non-routable DNS domain name.

      - Contoso has not implemented any cloud-based services, including an Azure AD tenant and an Azure subscription.

    Implementing the hybrid identity model will allows Contoso to take advantage of such technologies and capabilities as: 

      - Passthrough authentication with Seamless Single Sign-On

      - Multi-Factor Authentication

      - Self-Service Password Reset

      - Azure AD Password Protection

      - Hybrid Azure AD join

      - Windows Hello for Business

      - Microsoft Intune automatic enrollment

      - Azure AD Conditional Access

      - Azure AD Application Proxy

      - Azure AD B2B

      - Azure AD B2C

2.  How does your design account for customer objectives and objections?

    The preferred solution relies on pass-through authentication with seamless single sign-on to provide integration between on-premises Active Directory environment and an Azure AD tenant. That integration, implemented by virtue of installing Azure AD Connect on an on-premises Windows Server with direct connectivity to Active Directory domain controllers and Azure AD, drives a number of other configuration choices, including Hybrid Azure AD join, Self-Service Password Reset with password writeback, Azure AD Password Protection for Windows Server Active Directory, Azure AD Multi-Factor Authentication, Azure AD Privileged Identity Management, Azure AD Conditional Access with Azure AD Identity Protection-based risk assessment, as well as Azure AD Application Proxy. Hybrid Azure AD join will additionally allow implementing Hybrid Key trust-based Windows Hello for Business on Windows 10 domain member computers, although this will require additional infrastructure changes, including installation of an internal Certification Authority. 

    In order to provide access to on-premises applications to business partners, Contoso will leverage Azure AD B2B capabilities and Azure AD Application Proxy. For in-house developed customer apps, Contoso will implement an Azure AD B2C tenant. 
    
*Architecting a hybrid authentication solution*

1.  How does your solution address all the customer requirements in regard to authentication?

    The prerequisites: 

      - Contoso will provision a new Azure Active Directory tenant with a custom, publicly routable domain name and use Azure AD Connect in order to integrate it with on-premises Active Directory environment. 

      - Contoso will purchase Azure AD Premium P2 licenses for its users, in order to provide the ability to implement: 

           - Azure AD Privileged Identity Management. This will allow designated users to temporarily elevate their privileges to manage other user accounts with auditing automatically enabled for all elevation events.

           - Azure AD Identity Protection. This will allow access control to applications and resources by relying on dynamically evaluated risk based on heuristics and globally collected security-related telemetry.

           - Conditional Access (available starting with Azure AD Premium P1). This will allow access control to applications and resources by relying on a combination of multiple conditions, including users group membership and state of the users' devices.

           - Multi-Factor Authentication (available starting with Azure AD Premium P1), including integration of MFA into Conditional Access scenarios through step up authentication.

           - Azure AD Application Proxy (available starting with Azure AD Premium P1). This will allow providing access to remote users and business partners to on-premises web applications.

           - Password Protection for Windows Server Active Directory (available starting with Azure AD Premium P1). This will allow imposing restrictions on allowed password values. 

           - Self-service password reset/change/unlock with on-premises writeback (available starting with Azure AD Premium P1). 

     ![image](https://user-images.githubusercontent.com/25365143/136541071-380eca05-a609-438d-8216-fd273058f0ec.png)

    The choice of authentication method: 

      - In order to minimize infrastructure footprint required for integration, streamline user experience, and, at the same time, ensure that any on-premises Active Directory user account restrictions, such as allowed sign-in hours must be honored, the proposed solution leverages pass-through authentication with seamless single sign-on (SSO). 

     ![image](https://user-images.githubusercontent.com/25365143/136541083-e132149a-bffc-4809-9e92-8ff4f2d42ff1.png)

      - The need for preserving on-premises Active Directory user account restrictions eliminates the possibility of relying exclusively on Azure AD password hash synchronization, even though this is the simplest way to enable authentication for on-premises directory objects in Azure AD (which also supports seamless SSO), with minimum infrastructure requirements. It is worth noting that the password hash synchronization also does not include expired and locked-out states of Active Directory user accounts.

      - Another authentication option facilitated by Azure AD Connect is based on federation, which allows for enforcing on-premises AD password policies and restrictions when authenticating Azure AD accounts, but it has a significantly larger infrastructure footprint and it is relatively complex to configure and operate. Unlike the first two, it also supports third-party multifactor authentication (including smartcards), but that was not stipulated as one of Contoso's requirements.

      - In order to minimize the use of passwords, Contoso will implement Windows Hello for Business. Windows Hello for Business replaces passwords with strong two-factor authentication on Windows 10 devices. This authentication consists of a new type of user credential that is tied to a device and uses a biometric or PIN. Windows Hello for Business lets users authenticate to an Active Directory or Azure Active Directory account. In hybrid deployments, Windows Hello for Business can leverage Hybrid Azure AD joined computers, so Contoso will start by configuring Hybrid Azure AD join for its on-premises computers. Considering that Contoso is not planning on using federated authentication, this implies the choice of Windows Hello for Business hybrid key trust deployment model. For more information regarding this topic, refer to *Configure Device Registration for Hybrid key trust Windows Hello for Business* at <https://docs.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-hybrid-key-trust-devreg>.

*Implementing a hybrid identity solution*

1.  What are the steps to implement a hybrid identity solution that will allow you to meet all the customer requirements?

    Azure AD tenant: in order to implement the hybrid identity model, Contoso will create an Azure AD tenant and purchase licenses Enterprise Mobility + Security     E5 licenses for its users. To account for the integration and single sign-on requirements, Contoso will also leverage its ownership of a publicly routable     DNS domain name and assign it as a verified, custom DNS domain name to the newly provisioned Azure AD tenant. 

    Active Directory configuration

      - UPN suffix

           - The publicly routable DNS domain name will be assigned as the suffix of the userPrincipalName attribute of all Active Directory user accounts that will be integrated with Azure AD by using Azure AD Connect. 

      - Recycle Bin

           - Contoso will also enable Active Directory Recycle Bin. It is recommended that you enable the AD Recycle Bin feature for your on-premises Active Directory domains which are synchronized to Azure AD. If you accidentally deleted an on-premises AD user object and restore it using the feature, Azure AD restores the corresponding Azure AD user object. By default, Azure AD keeps the deleted Azure AD user object in soft-deleted state for 30 days.

           - If you do not have on-premises AD Recycle Bin feature enabled, you may be required to create an AD user object to replace the deleted object. If Azure AD Connect Synchronization Service is configured to use system-generated AD attribute (such as ObjectGuid) for the Source Anchor attribute, the newly created AD user object will not have the same Source Anchor value as the deleted AD user object. When the newly created AD user object is synchronized to Azure AD, Azure AD creates a new Azure AD user object instead of restoring the soft-deleted Azure AD user object.

    Azure AD Connect configuration

      - Authentication method

           - For pass-through authentication, you need to install at least one or more (three are recommended) lightweight Authentication Agents on your on-premises computers running Windows Servers 2012 R2 or newer with TLS 1.2 enabled. The computers hosting the agents must have direct access to Active Directory domain controllers and outbound access to internet. The first agent is installed automatically on the computer hosting Azure AD Connect once you choose to use pass-through authentication. To install additional agents, you can download their setup files from the **Pass-through authentication** blade (accessible via **Azure AD Connect** blade in the **Azure Active Directory** section of the Azure portal). Installation can be performed interactively (you will be prompted to sign in with an account that has been assigned the Azure AD Global Administrator role) or via an unattended deployment script. 

      - Filtering 

           - Azure AD Connect offers a number of different filtering options that determine the scope of synchronized Active Directory objects. While organizational unit-based filtering is the most straightforward to configure option, the scope can be based on a value of individual Active Directory attributes, which offers object-level granularity. 

           - Configuring attribute-based filtering relies on declarative provisioning, which is configurable by using Synchronization Rules Editor, included in the installation of Azure AD Connect. It can be applied when importing objects from Active Directory into to the metaverse (inbound) or when exporting objects from the metaverse to Azure AD (outbound). The recommended approach involves inbound filtering because this is easiest to maintain. Outbound filtering might be required in some scenarios, such as, for example, joining objects from more than one Active Directory forest before applying the filtering logic. 

           - In inbound filtering, the scope determines which objects to synchronize or not synchronize. The scope has a group and a clause to determine when a sync rule is in scope. A group contains one or many clauses. There is a logical *AND* between multiple clauses, and a logical *OR* between multiple groups. Objects which are supposed to be synchronized to Azure AD must have the metaverse attribute **cloudFiltered** not set to a value to be synchronized. If this attribute's value is set to **TRUE**, then the object is not synchronized. It is important to note that, in general, this attribute should not be set to **FALSE**. To make sure that multiple rules can affect its value, the attribute is supposed to have the value of either **TRUE** or **NULL** (not set). There are, however, scenarios where the choice of **FALSE** is appropriate, such as, so called *positive* filtering, where you do specify which objects to include (rather than exclude) based on the value of their designated attribute. 

           - Contoso will use a combination of the organizational unit-based filtering and the *positive* filtering based on the value of the userPrincipalName attribute. In particular, user objects to be synchronized will need to have the domain suffix portion of their userPrincipalName attribute match the custom, verified DNS domain name of the Azure AD tenant. This value can be set individually on per user object level as part of staged implementation of the proposed hybrid identity solution.  

    Azure AD Conditional Access

      - A Conditional Access policy is configurable directly from the Azure portal and is intended for granting or blocking access to Azure AD integrated applications and services based on a number of criteria such as:

           - All users, a specific user, member of a group, or assigned role

           - Specific cloud application being accessed

           - Device platform

           - Device state or compliance

           - Network location or geo-located IP address (customers have the ability to define trusted networks by using named locations)

           - Client applications 

           - Sign-in risk (Requires Identity Protection)

           - Hybrid Azure AD joined device

           - Approved client application

      -  Other options include stepping up authentication by enforcing Multi-Factor Authentication and applying session-level (rather than gated) restrictions.

    Azure AD Multi-Factor Authentication

      - Implementing Azure AD Multi-Factor Authentication involves three distinct configuration steps:

           - Configuring the MFA registration method

           - Configuring the MFA authentication method

           - Configuring scenarios in which MFA is required

      - Customers must determine how users will register their authentication methods. This can be accomplished by using Conditional Access, since this approach offers more flexibility. Conditional Access policies enforce registration, requiring unregistered users to complete registration at first sign-in. While it is possible to enable Multi-Factor Authentication by modifying the user state, this effectively forces users to perform two-step verification every time they sign in and overrides Conditional Access policies. However, there are scenarios in which this approach might be preferred or required. It is, for example, necessary, if the customer's current licensing arrangements do not include Conditional Access.

      - Additionally, with Azure AD Premium P2 licensing, customers have the option of leveraging Azure AD Identity Protection to further enhance MFA capabilities by implementing an automatic MFA registration policy, as well as incorporating automated risk detection into MFA-based Conditional Access policies. Creating an MFA registration policy will prompt users to register the next time they sign in interactively. Conditional Access policies can be configured to force password changes when there is a threat of compromised identity or require MFA when a sign-in is deemed risky in response to detection of such events as: 

           - Leaked credentials

           - Sign-ins from anonymous IP addresses

           - Impossible travel to atypical locations

           - Sign-ins from unfamiliar locations

           - Sign-ins from infected devices

           - Sign-ins from IP addresses with suspicious activities

     Some of the risk detections detected by Azure Active Directory Identity Protection occur in real time and some require offline processing. Administrators can choose to block users who exhibit risky behaviors and remediate manually, require a password change, or require a multi-factor authentication as part of their Conditional Access policies.

      - Customers also need to choose the authentication methods that they want to make available for users. It is important to allow more than a single authentication method so that users have a backup method available in case their primary method is unavailable. The methods include: 

           - Notification through mobile app

           - Verification code from mobile app

           - Call to phone

           - Text message to phone

    Azure AD Self-Service Password Reset (SSPR) and Azure AD Connect password writeback

      - SSPR allows users to reset their password in a secure way using some of the same methods they use for Azure AD Multi-Factor Authentication (there are a few additional options not available with MFA. Enabling SSPR requires selecting at least one of the following options for the authentication methods (it is recommended to choose two or more authentication methods so provide users with more flexibility):

           - Mobile app notification

           - Mobile app code

           - Email

           - Mobile phone

           - Office phone

           - Security questions

     Users can only reset their password if they successfully complete required authentication challenges. A new functionality (currently in preview) offers combined registration for Azure MFA and self-service password reset (SSPR).

      - Customers can enable password writeback by using Azure AD Connect, which allows users to reset passwords of their Active Directory accounts by leveraging Azure AD Self-Service Password Reset.

    Azure AD password protection for Windows Server Active Directory

      - Azure AD Password Protection for Windows Server Active Directory allows you to eliminate easily guessed passwords, including customizable password list that you can manage directly from the Azure portal. This feature relies on the Azure AD password protection DC agent software. The agent can only validate passwords when it is installed on a domain controller running Windows Server 2012 or newer, and only for password changes that are sent to that domain controller. It is not possible to control which domain controllers are chosen by Windows client machines for processing user password changes. In order to guarantee consistent behavior and universal password protection security enforcement, the DC agent software must be installed on all domain controllers in a domain.

      - Azure AD Password Protection for Windows Server Active Directory relies additionally on the Azure AD Password Protection Proxy service, which can be installed on any domain-joined Windows Server 2012 R2 or newer, with .NET 4.7 installed and with connectivity to internet. Its primary purpose is to forward password policy download requests from domain controllers to Azure AD and to return the responses from Azure AD to the DC Agent service running on individual domain controllers.

    Smart Lockout

      - Smart lockout can be integrated with hybrid deployments, using password hash sync or pass-through authentication to protect on-premises Active Directory accounts from being locked out by attackers. By setting smart lockout policies in Azure AD appropriately, attacks can be filtered out before they reach on-premises Active Directory.

      - When using smart lockout in pass-through authentication scenarios, you need to make sure that:

           - The Azure AD lockout threshold is less than the Active Directory account lockout threshold. Set the values so that the Active Directory account lockout threshold is at least two or three times longer than the Azure AD lockout threshold.

           - The Azure AD lockout duration must be set longer than the Active Directory reset account lockout counter after duration. Note that, when using graphical interface tools (Group Policy Management Editor for Active Directory account lockout policy settings and the Azure portal for Smart Lockout settings), the Azure AD duration is set in seconds, while the AD duration is set in minutes.

      - When combined with password hash synchronization, smart lockout keeps track of the last three bad password hashes to avoid incrementing the lockout counter for the same password. This way, if someone enters the same bad password multiple times, this behavior will not cause the account to lockout. However, hash tracking functionality is not available for customers with pass-through authentication enabled as authentication happens on-premises not in the cloud.

    Azure AD Application Proxy: 

      - Azure AD provides the ability to access on-premises web applications by relying on Azure AD Application Proxy via an external URL or an internal application portal. Azure AD Application Proxy offers a single sign-on experience and consistent user interface regardless of the location of the target app. For example, Application Proxy can facilitate access to on-premises line of business (LOB) applications, Office 365, or any other SaaS-based application integrated with Azure AD. Application Proxy works with:

           - Web applications that use Integrated Windows Authentication for authentication

           - Web applications that use form-based or header-based access

           - Web APIs that you want to expose to rich applications on different devices

           - Applications hosted behind a Remote Desktop Gateway

           - Rich client apps that are integrated with the Active Directory Authentication Library (ADAL)

      - Azure AD Application Proxy consists of the following components: 

           - **Endpoint** represents a URL or an end-user portal via which remote users access the on-premises applications. Such access is first authenticated by Azure AD and then are routed through the Azure AD Application Proxy connector to the on-premises application.

           - **Azure AD tenant** performs the authentication of remote users attempting to access on-premises applications.

           - **Application Proxy service** hosted by Azure AD passes the sign-in token from the user to on-premises instances of Application Proxy Connector. 

           - **Application Proxy Connector** is a lightweight, stateless agent running on an on-premises Windows Server 2012 R2 or newer with direct connectivity to the target application. The server needs to have TLS 1.2 enabled before you install the Application Proxy connector. The connector manages communication between the on-premises application and the Application Proxy service via an outbound, persistent connection, which eliminates dependency on a perimeter network or open inbound ports on perimeter firewalls. In general, connectors can run on a Windows server that is not domain-joined. However, scenarios that require single sign-on (SSO) to applications which rely on Integrated Windows Authentication (IWA), it is necessary to use a domain-joined machine. In such scenarios, the connector machines must be domain-joined in order to perform Kerberos Constrained Delegation on behalf of the users of the published applications. This is one of the requirements that applies to the proposed solution. 

           - **Active Directory** performs authentication required to access on-premises applications. In single sign-on scenarios, the connector communicates with Active Directory to authenticate incoming access requests.

           - **On-premises applications** deliver required functionality to users once their access requests are authenticated.


*Assessing resiliency aspects of a hybrid identity solution*

1.  What are provisions that eliminate single points of failure in your design?

    - The potential single points of failure to consider in a hybrid identity design based on the components incorporated into the proposed solution include the following:

        - Network connectivity between Active Directory and Azure Active Directory

            - It is imperative that the connectivity between Active Directory domain controllers and Azure AD is highly available and performant. Without it, pass-through authentication requests will fail, preventing users from accessing Azure AD protected resources and applications.

              You can use password hash synchronization as a backup authentication method for pass-through authentication, to address scenarios in which the agents cannot validate users' credentials because Active Directory domain controllers are unavailable or unreachable. By combining password hash synchronization and pass-through authentication, users will be able to authenticate directly against Azure AD in cases where the latter fails.

              Another mitigation approach involves extending your Active Directory environment to Azure. To accomplish this, you need to establish a hybrid network connection (such as Site-to-Site VPN or ExpressRoute) between your on-premises data center and an Azure virtual network and 
deploy additional domain controllers of the on-premises Active Directory domain into that virtual network, as well as install additional pass-through authentication agents on Azure virtual machines within the same virtual network. This minimizes the possibility of network connectivity issues affecting communication between Active Directory and Azure AD. 

        - Azure AD Connect synchronization engine

            - In order to facilitate provisions that require eliminating single points of failure, in regard to the Azure AD Connect synchronization engine, customers have the option of deploying an additional server hosting the sync engine operating in the staging mode (this is one of the options available directly from the installation wizard interface).
In this mode, the sync engine imports and synchronizes data the same way as the active instance, but it does not export anything to Azure AD or AD. Since a server in the staging mode continues to receive changes from Active Directory and Azure AD, it can quickly take over the responsibilities of a failed active server. Password sync and password writeback features of Azure AD Connect are disabled while in staging mode. 

        - Azure AD Connect pass-through authentication agent

            - For pass-through authentication, you can install two or more (three are recommended) lightweight Authentication Agents on your on-premises computers running Windows Servers 2012 R2 or newer with TLS 1.2 enabled. Installing multiple Pass-through Authentication Agents ensures high availability, but not deterministic load balancing between the Authentication Agents. To determine how many Authentication Agents you need for your tenant, consider the peak and average load of sign-in requests that you expect to see on your tenant. A single Authentication Agent can handle typically between 300 to 400 authentications per second on a standard 4-core CPU, 16-GB RAM server. 

        - Azure AD Password Protection for Windows Server Active Directory

            - The main availability concern for password protection is the availability of proxy servers when the domain controllers in a forest try to download new policies or other data from Azure. Each DC Agent uses a simple round-robin-style algorithm when deciding which proxy server to call. The Agent skips proxy servers that are not responding. For most fully connected Active Directory deployments that have healthy replication of both directory and sysvol folder state, two proxy servers are enough to ensure availability. This results in timely download of new policies and other data. However, you have the option of deploying additional proxy servers.

              The design of the DC Agent software mitigates the usual problems that are associated with high availability. The DC Agent maintains a local cache of the most recently downloaded password policy. Even if all registered proxy servers become unavailable, the DC Agents continue to enforce their cached password policy. A reasonable update frequency for password policies in a large deployment is usually days, not hours or less. Effectively, brief outages of the proxy servers do not significantly impact Azure AD password protection.

        - Azure AD Application Proxy connector

            - Customers should deploy two or more connectors and organize them into connector groups, with each group handling traffic to specific applications. Connector groups not only provide high availability but also facilitate improving latency when accessing applications hosted in different regions, since it is possible to create location-based connector groups to serve only local applications.
It is important to provision sufficient number of connectors to handle the expected volume of application traffic. It is recommended that each connector group has at least two connectors to provide high availability and scale. Having three connectors accounts for maintenance windows necessary to service servers hosting the connectors. For details regarding sizing of the servers hosting connectors, refer to *Understand Azure AD Application Proxy connectors* at <https://docs.microsoft.com/en-us/azure/active-directory/manage-apps/application-proxy-connectors>.

              **Note**:  Azure AD Password Protection Proxy and Application Proxy install different versions of the Microsoft Azure AD Connect Agent Updater service. These different versions are incompatible when installed side by side, so it is not recommended to install Azure AD Password Protection Proxy and Application Proxy side by side on the same machine.

        - Backup domain controller
  
            - In order to increase availability of resources and authentication located on-premises, it is recommended to also add a backup domain controller within the hybrid infrastructure.  This backup domain controller could be a virtual machine in Azure or a physical server on-premises as long as it is able to communicate directly with the primary domain controller.  
            - This backup domain controller will have Azure AD Connect installed and configured in standby mode and can be made primary if the main domain controller goes offline.
            - The backup domain controller can also have a pass through authenticator agent installed for redundancy and also configured with the application proxy configuration to maintain resiliency for application access.

1.  What is the failover process for components that operate in the active/passive mode?

   - Network connectivity between Active Directory and Azure Active Directory

       - Fail over to password hash synchronization does not happen automatically. In order to perform a switch, you must re-run Azure AD Connect to reconfigure the authentication method. Note that this requires that you have at that point connectivity to both Active Directory domain controllers and Azure AD, which affect the viability of this option. Instead, you should consider designing your network and Active Directory infrastructure in the manner that provides sufficient resiliency and availability (for example, by deploying additional Active Directory domain controllers and pass-through authentication agents into an Azure virtual network).

   - Azure AD Connect synchronization engine

       - In case of a problem with the server hosting the active instance of the Azure AD Connect sync engine, the failover involves simply rerunning the installation wizard to change the mode of the staging instance (the option labeled *Enable staging mode. When selected, synchronization will not export any data to AD or Azure AD* appears on the **Ready to configure** page of the wizard). At that point, that instance will start to operate as the active one. It is important to remember to either deprovision the former active instance or switch it to the staging mode in order to ensure that there is only a single server which is actively exporting data.

   - Azure AD Connect passthrough authentication agent

       - Authentication requests are dynamically distributed across all available Authentication Agents, so no explicit failover is required. 

   - Azure AD Password Protection for Windows Server Active Directory

       - The architecture of Azure AD Password Protection for Windows Server Active Directory relies on two main components. The DC Agent should be installed on each domain controller, which inherently provides high availability. You also have the option to install multiple Azure AD Password Protection Proxy service agents for high availability. However, as mentioned earlier, the DC Agent maintains a local cache of the most recently downloaded password policy. Even if all registered proxy servers become unavailable, the DC Agents continue to enforce their cached password policy.

   - Azure AD Application Proxy connector

       - Application connection requests are dynamically load balanced across all available connectors, so no explicit failover is required. The connectors and the Azure AD Application Proxy service automatically handle all high availability tasks. They can be added or removed dynamically. Each time a new request arrives it is routed to one of currently available connectors. If a connector is temporarily unavailable, it is excluded from request distribution. Connectors also poll the service to determine whether there is a newer version of the connector software and, if one is found, they trigger an automatic update.

*Optimizing authentication configuration*

1.  What features will allow you to optimize authentication in your solution to satisfy the following customer requirements?

    - multi-factor authentication

    - self-service password reset

    - replacing password-based authentication with biometrics-based sign-in

    - Multi-Factor Authentication

      - Azure Multi-Factor Authentication (MFA) helps safeguard access to data and applications. It provides an additional layer of security using a second form of authentication. MFA can be used in combination with Conditional Access and Azure AD Identity Protection. It is also an essential part of configuring Self-Service Password Reset (SSPR). 

    - Self-Service Password Reset
    
      - Self-Service Password Reset (SSPR) allows users to reset their password in a secure way using the same methods they use for multi-factor authentication. Combining Self-Service Password Reset with password writeback that can be enabled by using Azure AD Connect allows users to reset passwords of their Active Directory accounts.

    - Windows Hello for Business

      - In Windows 10, you can use Windows Hello for Business to replace passwords with strong two-factor authentication. This authentication consists of a new type of user credential that is tied to a device and uses a biometric or PIN. Windows Hello for Business lets user authenticate to an Active Directory or Azure Active Directory account.

        While Windows Hello for Business can be implemented exclusively in on-premises environments, its deployment can be simplified by leveraging Azure AD and Azure AD Connect in hybrid scenarios. Considering that one of design objectives in our proposed solution was minimizing infrastructure footprint, the recommended approach in this case is to use Hybrid Azure AD joined Key Trust Deployment, which can be implemented in combination with Azure AD pass-through authentication (eliminating the need for federation servers). 

        **Note**: Windows Hello for Business with a key does not support RDP. RDP does not support authentication with a key or a self-signed certificate. RDP with Windows Hello for Business is supported with certificate-based deployments.

        One of the prerequisites for implementing Hybrid Azure AD joined Key Trust Deployment of Windows Hello for Business is registration of Windows 10 client devices in Azure Active Directory. In the proposed solution, this is performed by leveraging the functionality of Azure AD Connect, which starting with version 1.1.819.0, includes a wizard that significantly simplifies the registration process. The wizard configures the Active Directory service connection points (SCPs) for device registration.

        For the information regarding other prerequisites and the process of implementing Hybrid Azure AD joined Key Trust Deployment of Windows Hello for Business, refer to *Hybrid Azure AD joined Key Trust Deployment* at <https://docs.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/hello-hybrid-key-trust>.

    - Microsoft Authenticator authentication

      - For users that do not have the capability to use Windows Hello for Business, the Microsoft Authenticator can be used for an additional layer of password-less security.
      

*Optimizing authorization configuration*

1.  What features will allow to optimize authorization in your solution to satisfy the following customer requirements?

    - Privileged Identity Management

    - identity protection

    - Privileged Identity Management

      - Azure AD Privileged Identity Management (PIM) is a service that facilitates managing, controlling, and monitoring access to cloud resources, including Azure AD, Azure, and other Microsoft Online Services, such as Office 365 and Microsoft Intune.

      - Privileged Identity Management provides time-based and approval-based role activation to mitigate the risks of excessive, unnecessary, or misused access permissions on cloud resources, including the following capabilities:

        - Provide just-in-time privileged access to Azure AD and Azure resources

        - Assign time-bound access to resources

        - Require approval to activate privileged roles

        - Enforce multi-factor authentication to activate any roles
        
        - Ensure that a rationale is provided as part of elevation approval process 

        - Configure notifications triggered by activation of privileged roles

        - Facilitate access reviews to validate whether users still are eligible for role assignment or elevation

        - Track elevation events

          **Note**: Privileged Identity Management requires Azure AD Premium P2 licensing. 

*Optimizing access control and management of applications and devices*

1.  What features will allow optimization of access control and management of applications and devices to satisfy the following customer requirements?

    - Mobile device management

    - Conditional access

    - Remote access to on-premises applications integrated with Active Directory

    - B2B

    - B2C

    - Mobile Device Management

      - With a growing number of Windows 10 devices, Contoso will need a mobile device management solution, such as Microsoft Intune, that integrates with the proposed hybrid identity model. Microsoft Intune leverages Azure AD as its identity provider and supports automatic enrollment as devices are joined to Azure AD. It provides a wide range of device and application management features, including configuration and compliance policies, software deployment, remote administration, and support for multi-identity (which separates corporate and personal data).

        Microsoft Intune licensing is included in the Enterprise Mobility + Security (EMS) suite (along with the Azure AD Premium P2 licensing). 

    - Azure AD Conditional Access

      - As mentioned earlier, Azure AD Conditional Access policy allows customers to restrict access to resources that integrate with Azure AD based on a wide range of criteria, including Azure AD group membership, target resource, device platform and state (such as Hybrid Azure AD join), network location, client application being used to access the resource, sign-in risk (evaluated by relying on Azure AD Identity Protection), and device compliance (evaluated by using Microsoft Intune compliance policies).

    - Azure AD Application Proxy

      - As mentioned earlier, Azure AD Application Proxy provides seamless access to on-premises line of business (LOB) applications, Office 365, or any other SaaS-based application integrated with Azure AD. 

    - Azure AD B2B

      - Azure AD business-to-business (B2B) functionality allows customers to grant access to applications and services integrated with their individual Azure AD tenants to guest users from other organizations. The capabilities available to guest accounts mirror, for the most part, those available to users that belong to the same Azure AD tenant. Guest accounts are provisioned via a straightforward invitation and redemption process, allowing invitees use their own credentials to authenticate. Since the partner organizations continue to rely on their own identity management solutions, additional administrative overhead is minimized. 

        While the partner guest accounts are stored in the same Azure AD tenant as the user accounts of members of the organizations that provide access to its resources, they are easy to distinguish since their **userType** attribute is set to **Guest**. However, the process of granting access to cloud-based apps is the same. In addition, you have the option of granting guest accounts access to on-premises apps. Details depend on the authentication capabilities of the apps. 

           - SAML: if on-premises apps use SAML-based authentication, these apps can be made available to Azure AD B2B guests directly from the Azure portal by adding them to Azure AD based on the non-gallery application template and then using Azure AD Application Proxy to publish them, with Azure AD configured as the authentication source

           - Integrated Windows Authentication (IWA) and Kerberos Constrained Delegation (KCD): if on-premises apps rely on Active Directory for authentication, then they not only need to be added to Azure AD and published via the Azure AD Application Proxy, but the B2B guest users must be added as users to Active Directory. There are two methods available that can be used to create the guest user objects that are required for authorization in the on-premises directory:

               - Microsoft Identity Manager (MIM) and the MIM management agent for Microsoft Graph.

               - A PowerShell script. Using the script is a more lightweight solution that does not require MIM.

              For more information regarding this subject, refer to *Grant B2B users in Azure AD access to your on-premises applications* at <https://docs.microsoft.com/en-us/azure/active-directory/b2b/hybrid-cloud-to-on-premises>.

    - Azure AD B2C

      - Azure Active Directory B2C provides business-to-customer identity as a service. Customers can use their preferred social, enterprise, or local account identities to authenticate in order to access applications and APIs offered by your organization. B2C requires an Azure AD tenant separate from the one used by organization's users and applications (and, consequently, different from the one that is used in B2B scenarios). In order to make your applications and APIs available via an Azure AD B2C tenant, you need to register them with that tenant.

        For more information regarding this subject, refer to *Technical and feature overview of Azure Active Directory B2C* at <https://docs.microsoft.com/en-us/azure/active-directory-b2c/technical-overview>


## Checklist of preferred objection handling

1.  Our Active Directory domain is using a non-routable domain name. We cannot risk renaming it in order to implement single sign-on with Azure Active Directory. 

    **Potential Answer:** Contoso does not have to rename their Active Directory domain in order to integrate with an Azure Active Directory tenant. Such integration is possible regardless of the DNS name of the Active Directory domain. What's important in order to ensure single sign-on experience for Active Directory users accessing cloud-based resources is to ensure that there is a match between the userPrincipalName in Active Directory and Azure AD. This is the Microsoft's recommended approach. It is also possible to configure **Alternate Login ID**, which makes it possible to choose another attribute to designate the sign-in user names. The impact of this choice differs depending on the authentication method. For more information regarding **Alternate Login ID**, refer to Microsoft Docs at <https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/operations/configuring-alternate-login-id>.

2.  We have heard that it is not possible to run multiple instances of Azure AD Connect simultaneously. All identity services components in our environment must provide resiliency and support failover.

    **Potential Answer:** While Azure AD Connect cannot operate in the active/active mode, it is possible to setup an additional server hosting the sync engine operating in the staging mode. In this mode, the sync engine imports and synchronizes data the same way as the active instance, but it does not export anything to Azure AD or AD. Password sync and password writeback features of Azure AD Connect are disabled while in staging mode. Since a server in the staging mode continues to receive changes from Active Directory and Azure AD, it can quickly take over the responsibilities of a failed active server. The switch involves simply re-running the Azure AD Connect installation wizard.

3.  If we decide to integrate our Active Directory environment with Azure Active Directory, this must be performed in stages. This is likely to be complex, considering that users in each stage would be members of different Active Directory groups and their accounts might reside in different Active Directory organizational units.

    **Potential Answer:** Azure AD Connect supports a number of different filtering options that determine the scope of synchronized Active Directory objects. While organizational unit-based filtering is the most straightforward to configure option, the scope can be based on a value of individual Active Directory attributes, which offers object-level granularity. 

4. Synchronizing our Active Directory accounts with Azure AD accounts makes the former vulnerable to malicious or accidental lockouts that affect the latter. This would effectively expose our on-premises environment to external attacks. 

    **Potential Answer:** Azure AD offers the Smart Lockout functionality, which can be integrated with hybrid deployments, using password hash sync or pass-through authentication to protect on-premises Active Directory accounts from being locked out by attackers. By setting smart lockout policies in Azure AD appropriately, attacks can be filtered out before they reach on-premises Active Directory.

5.  A number of critical web applications running in our on-premises environment rely on Kerberos-based Windows Integrated Authentication. Microsoft states that Azure Active Directory does not support Kerberos. Doesn't this mean that remote users authenticating to Azure Active Directory and our business partners will not be able to properly authenticate and access these applications?

    **Potential Answer:** Azure AD provides the ability to access on-premises web applications by relying on Azure AD Application Proxy via an external URL or an internal application portal. Azure AD Application Proxy offers a single sign-on experience and consistent user interface regardless of the location of the target app. For example, Application Proxy can facilitate access to on-premises line of business (LOB) applications, Office 365, or any other SaaS-based application integrated with Azure AD. This eliminates the need for VPN infrastructure and integrates with other Azure AD features, such as Conditional Access and Multi-Factor Authentication. At the same time, there is no need to open any inbound ports on perimeter firewalls. 

    Application Proxy works with:

    - Web applications that use Integrated Windows Authentication for authentication

    - Web applications that use form-based or header-based access

    - Web APIs that you want to expose to rich applications on different devices

    - Applications hosted behind a Remote Desktop Gateway

    - Rich client apps that are integrated with the Active Directory Authentication Library (ADAL)
 
## Customer quote 

"Azure AD offers a wide range of new opportunities that will allow us to extend our identity and access management far beyond our existing on-premises Active Directory environment, facilitating support for our remote users and providing secure and easy to manage integration platform for our business partners and customers."

---Andrew Cross, CIO, Contoso
