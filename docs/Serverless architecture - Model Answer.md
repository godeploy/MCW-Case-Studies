---
lab:
    title: 'Serverless architecture - Model Answer'
---


# Case Study - Serverless architecture - Model Answer

## Preferred solution

_High-level architecture_

1. Without getting into the details (the following sections will address these), diagram your initial vision for handling the top-level requirements for the license plate processing serverless components, OCR capabilities, data export workflow, and monitoring plus DevOps.

   ![image](https://user-images.githubusercontent.com/25365143/136542835-f359ac25-2449-4088-9228-7ece7d556f2a.png)


   The solution begins with vehicle photos being uploaded to an Azure Data Lake Storage (ADLS) Gen2 container as they are captured. Each image is stored in a subfolder named after the photo booth's Id so the Id can be added to the processed license plate data in the database. An Event Grid subscription is created against the data lake storage blob created event, calling the photo processing Azure Function endpoint (on the side of the diagram), which in turn sends the photo to the Cognitive Services Computer Vision API OCR service to extract the license plate data. If processing succeeds and the license plate number is returned, the function submits a new Event Grid event, along with the data, to an Event Grid topic with an event type called "savePlateData." However, if the processing was unsuccessful, the function submits an Event Grid event to the topic with an event type called "queuePlateForManualCheckup." Two separate functions are configured to trigger when new events are added to the Event Grid topic. Each function filters on a specific event type, saving the relevant data to the appropriate Cosmos DB collection for the outcome, using the Cosmos DB output binding. A Logic App that runs on an hourly interval executes an Azure Function via its HTTP trigger, responsible for obtaining new license plate data from Cosmos DB and exporting it to a new CSV file saved to the data lake. If no new license plate records are found to export, the Logic App sends an email notification to the Customer Service department via their Office 365 subscription. Application Insights is used to monitor all Azure Functions in real-time as data is being processed through the serverless architecture. This real-time monitoring allows Contoso to observe dynamic scaling first-hand and configure alerts when certain events take place. GitHub Actions is used to manage the DevOps processes and the CI/CD workflow. Azure Key Vault is used to securely store secrets, such as connection strings and access keys. Key Vault is accessed by the Function Apps through an access policy within Key Vault, assigned to each Function App's system-assigned managed identity.

   > **Note**: The preferred solution is only one of many possible, viable approaches.

_License plate processing serverless components_

1. Which Azure messaging service would you recommend using to orchestrate event-driven activities between the serverless components?

   Use Event Grid, which is an eventing backplane that enables reliable event-driven, reactive programming. It uses a publish-subscribe model where publishers, like an Azure Data Lake Storage (ADLS) Gen2 container and Azure Functions, emit events and have no expectation about which events are handled. Subscribers decide which events they want to take using filters to select only the incoming events they wish to process intelligently. For example, use Event Grid to instantly trigger the serverless function to run image analysis each time a new photo is added to the data lake's images container. Event Grid also works with custom topics, allowing you to pass data into the topic from the photo processing function, and have another function that handles saving successful license plate processing data to the database, and a different function that saves information about photos that need to be manually reviewed when no license plate number is found. This is done by setting the event type to a custom value that the downstream functions filter to process the correct event data.

   Event Grid will support Contoso's future expansion efforts through its ability to fan-out events. This means that you can subscribe to multiple endpoints to the same event to send copies of the event to as many places as needed. This allows Contoso to add new event subscribers that perform different processing functions without overhauling the eventing backplane. Event Grid costs \$0.60 per million operations (\$0.30 during preview), and the first 100,000 operations per month are free. Operations are defined as event ingress, advanced match, delivery attempt, and management calls. More details can be found on the [pricing page](https://azure.microsoft.com/pricing/details/event-grid/).

2. What Azure service would you suggest Contoso use to execute custom business logic code when an event is triggered?

   Azure Functions, which is Azure's de facto serverless compute service that enables you to run your custom business logic code-on-demand without the need to provision or manage infrastructure, dynamically scales to meet demand and enables you to pay only for the compute resources used. It is important to note that while every PaaS service also supports scalability, the scaling is typically completely dynamic and handled by the vendor when using serverless computing services. Instead of needing to define metrics (such as high memory utilization or CPU percentage) that initiate scaling in a PaaS service and the scaling procedure, these details are handled for you behind the scenes with serverless. The vendor will allocate additional computing resources to your function to meet demand and deallocate those resources when they are no longer needed.

3. Which pricing tier for the service would you recommend that would automatically scale to handle demand while charging only for work performed?

   Azure Functions comes in three primary offerings, consumption, dedicated, and premium. Understanding the differences between them is essential, as choosing one over the other dictates your application's behavior and how you're billed.

   For Contoso, the Azure Functions consumption plan is recommended. Under this plan, their Function Apps will automatically be scaled by the platform to meet demand without any configuration needed on their part. No developer action is required. CPU and memory resource allocation is automatically handled by adding new processing instances.

   If, over time, they are experiencing a high volume of requests or seeing many long-running requests, the premium plan provides added advantages while still allowing them to benefit from serverless features. In the premium plan, instances of the Azure Functions host are added and removed based on the number of incoming events, just like with the consumption plan.

   The other plan option for Function Apps is the dedicated plan, which uses a dedicated App Service hosting plan, meaning it is not serverless. Automatic scaling is available under this plan, though it is not as dynamic. You supply the metrics or schedule that kick off the scaling procedures, though, with this plan, you have the option of both scaling up to a higher tier with more powerful VMs, as well as out to a higher number of instances. The App Service hosting plan is a good choice for long-running functions, as there is no limit on the function execution time, as opposed to the maximum 10-minute execution time limit imposed by the consumption plan. It is also suitable for functions that require fast start-up time after an idle period. This plan reserves at least one dedicated instance that is always allocated to the Function App and supports using the Always On setting to prevent the instance from going idle. However, in this scenario, none of the functions are particularly long-running, and the Consumption plan meets the requirement that charges are only incurred when work is performed. This is because the Consumption plan does not hold any instances in reserve at all times.

   You can learn more by reading the [Azure Functions hosting options](https://docs.microsoft.com/azure/azure-functions/functions-scale) article in the Azure documentation.

4. How do you ensure that downstream components, such as machine learning APIs, databases, and file stores, are not overloaded by the potential high load created when your serverless components dynamically scale?

   There are a few ways to address this issue. The first step is to review your downstream services to see any published rate limits, controlling how often you can call the service. For instance, if you are using the free tier of a Cognitive Services API, you are limited to 10 requests per minute, at a maximum of 5k requests per month. If you anticipate exceeding these limits, consider upgrading to a higher tier that allows for more requests at a higher interval. The second step is to perform a load test on your architecture, using a service such as Azure DevOps, and observe the collected telemetry during the load test for any failures in your downstream components. If the resource-constrained element is an HTTP-based service, look for HTTP response codes, such as 429 and 503, which could indicate throttling due to exceeding the number of requests allowed through policy or service limits. Once you have taken note of any such limitations, there are two methods by which you can reduce the stress on affected downstream services. The first approach is to configure the concurrency settings on your triggers by setting the maxConcurrentRequests value in the host.json file to the maximum number of HTTP functions that will be executed in parallel. This implements rate limiting on the function itself to help alleviate the stress on affected downstream components. The second approach is to implement a resiliency strategy in code by using a retry pattern with exponential back-off or a circuit breaker pattern that prevents your function from repeatedly executing an operation that is likely to fail. Your best option might be to use a combination of these two patterns to handle different types of errors and status codes with varying fault handling strategies. Review the Azure Architecture center's article on transient fault handling for more information: <https://docs.microsoft.com/azure/architecture/best-practices/transient-faults>.

5. What Azure service would you recommend for storing the license plate data? Consider options that automatically scale to meet demand and offer bindings to other serverless components that simplify connecting to and storing data within the data store.

   Use Azure Cosmos DB, a globally distributed, massively scalable, multi-model database service that includes native integration with Azure Functions. You can use output bindings directly to Cosmos DB from your functions, simplifying the steps needed to persist your incoming license plate processing data.

_License plate OCR_

1. What service would you recommend Contoso use to conduct object character recognition (OCR) processing to extract the license plate number from each photo as it enters the system?

   Since Contoso has stated that they have no in-house expertise in machine learning or data science, the most straightforward approach would be to use the Cognitive Services Computer Vision API and its built-in OCR capabilities. Because the API is an HTTP REST-based service, all that is needed is a simple POST containing the photo with a license plate, and it will return any detected text characters. It is important to note that, at this time, the Computer Vision API is not optimized for license plate recognition (LPR), which may be added in the future. While license plate text will be recognized in some cases, additional processing of the returned data may need to be performed to strip out State/Province names, as well as invalid characters.

2. How would you integrate the OCR service into your license plate processing flow?

   The Computer Vision API can be integrated by having the image processing function make a REST request containing the photo to the registered service endpoint. It is highly recommended that a resiliency strategy be implemented for the call if the service is imposing rate limits on your requests during high traffic periods. The return data is in JSON format, containing the discovered text and bounding boxes indicating their position within the photo.

_Data export workflow_

1. What Azure service would you recommend to create an automated workflow that runs on a regular interval to export processed license plate data and send alerts as needed?

   Logic Apps would help integrate an automated workflow without worrying about building hosting, scalability, availability, and management. You would add a recurrence trigger to the logic app that fires on a defined schedule. The logic app would execute the Azure function that collects new processed license plate data and exports it to a CSV file in Azure Storage.

2. Which other services would you integrate into your workflow?

    Contoso has stated that they use Office 365 services for their email. Logic Apps can use the Office 365 Outlook connector to send emails to any number of recipients. Start by adding a condition after the call to the Azure function. This condition needs to evaluate the response code sent by the function to determine whether the email step is fired. If the code is not equal to 200 (OK status), then the email should be sent with information in the body crafted based on the response code. For instance, the function should return a 204 (NoContent) status code if no license plate data was found that needs to be exported. This would indicate that no photos were processed in the period since the previous trigger interval, possibly meaning a situation out of the norm that should be investigated more closely.

_Extensible serverless analytics_

1. Assuming they would like to be able to plug-in more solutions that respond to the event when a license plate has been successfully extracted from an image, how would you extend your solution using Event Grid? Be specific on the system topics, custom topics, and subscriptions at play.

   As the following diagram illustrates, Contoso could design their solution to be extensible using an Event Grid Custom Topic. Whenever the license plate image is successfully processed, the Azure Function that saves the license plate data to Cosmos DB could instead be extended to enqueue an event into a Custom Topic. The enqueued custom topic would have an event type of "LicensePlate.Processed," a data payload of the image path (in ADLS), the extracted license plate text, and the image's timestamp was captured. Other Event Grid subscriptions could later be created against this topic, listening for the aforementioned event types.

   With the aforementioned custom topic in place, you could achieve both near-real-time and batch analytics are possible by configuring a new subscription that uses an Event Hubs instance as the handler.

   You would configure Event Hubs Capture to write the ingested messages periodically (defined in terms of the volume of messages in MB or the length of a sliding window) to Azure Storage. From ADLS Gen2, you could use Azure Data Lake Analytics to batch process the data, join it with other lookup data or perform other transformations. Azure Data Lake Analytics allocates resources to your jobs automatically and is completely serverless.

   Similarly, from the same Event Hubs instance, you could connect Azure Stream Analytics to apply standing SQL queries to analyze the time-series data (such as by computing aggregates over windows of time) in near-real-time and output the result to several Azure services like SQL Database or Power BI.

   ![image](https://user-images.githubusercontent.com/25365143/136542860-a5885736-83b7-4041-beb3-cf359f86e0ff.png)

2. What pipeline would you plug-into an Event Grid subscription listening for license plate events that could be used to provide real-time and batch analytics as a serverless solution?

   The previous diagram illustrates this solution. With the custom topic mentioned above, you could achieve near-real-time and batch analytics by configuring a new subscription that uses an Event Hubs instance as the handler.

   You would configure Event Hubs Capture to write the ingested messages periodically (defined in terms of the volume of messages in MB or the length of a sliding window) to a container in ADLS Gen2. From ADLS Gen2, you could use Azure Data Lake Analytics to batch process the data, join it with other lookup data or perform other transformations. Azure Data Lake Analytics allocates resources to your jobs automatically and is completely serverless.

   Similarly, from the same Event Hubs instance, you could connect Azure Stream Analytics to apply standing SQL queries to analyze the time-series data (such as by computing aggregates over windows of time) in near-realtime. Stream Analytics can output the result to several Azure services like SQL Database or Power BI.

_Monitoring and DevOps_

1. What tools and services would you recommend Contoso use to develop the serverless components locally, synchronize with a source code repository, and implement continuous deployment?

   The Azure Functions Core Tools is a local version of the Azure Functions runtime that can run on your local computer for debugging during your development process or run in a local, isolated environment. The core tools use the same runtime that powers Functions in Azure, and it can be installed using the NodeJS package manager (npm). Though the core tools can be used from the command line or with several code editors, the best integration is with Visual Studio.

   You can facilitate automated deployments of your function app through App Service continuous integration. Use one of the integrated services for your source code repository, such as BitBucket, Dropbox, GitHub, and Azure DevOps, enabling a workflow whereby a deployment to Azure is triggered when your function code is pushed to your repository. This continuous delivery setup deploys the entire Function App, not a single function. If a function needs to change independently, it is best to deploy it as a separate Function App.

   [GitHub Actions](https://docs.github.com/actions) provide a way to automate, customize, and execute your software development workflows right in your repository. You can discover, create, and share actions to perform any job you'd like, including CI/CD, and combine activities in a completely customized workflow.

2. How would you monitor all the executing serverless components in real-time from a single dashboard?

   Use Azure Application Insights. Azure Functions provides built-in integration with Application Insights that makes it easy to monitor your functions. Use the Live Metrics Stream feature of Application Insights to view the incoming requests, outgoing requests, overall health, allocated server information, and sample telemetry in near-real-time. This will help you observe how your functions scale under load and allow you to spot any potential bottlenecks or problematic components through a single interactive interface.

3. Does your monitoring solution support exploring historical telemetry and configuring alerts?

   Yes, Application Insights makes it easy to configure alerts triggered when specific metrics conditions are met, such as server response time, activity log service, type, status/level, or even when exact search text is found in the log files. Current and historical telemetry and log data are retained by Application Insights, depending on the retention policies set forth by your selected service plan.

## Checklist of preferred objection handling

1. We are concerned about how individual serverless components will be able to "talk" to each other and reliably pass messages through the pipeline.

   Azure Functions support multiple triggers, including several Azure services and basic yet flexible trigger types such as HTTP and webhook triggers. In our proposed solution, we use Event Grid bindings that allow events to trigger certain functions. These bindings enable other functions to send data to the reliable queue through an Event Grid topic and designate a specific event type. Hence, functions that need to react to particular events only respond to the filtered events they care about.

2. Will a serverless architecture that can infinitely scale put us at risk for substantial monthly bills?

   Azure Functions comes in three primary offerings, consumption, dedicated, and premium. Understanding the differences between them is essential, as choosing one over the other dictates your application's behavior and how you're billed.

   The consumption plan is Microsoft's "serverless" model. In this model,  your code reacts to events, effectively scales out to meet load, scales down when code isn't running, and you're billed only for what you use. One of the primary benefits of using serverless components in a consumption plan is that billing is based on resources consumed or the actual time your code is running. For many event-driven workflows, sub-second billing will save more money in the long run compared to paying for constantly running resources, whether they are used or not. When using Azure Functions, short-lived and sporadic use will benefit most from the consumption plan. Most serverless components also provide options to specify upper limits on things like concurrent executions and other rate-limiting options.

   However, if you are consistently getting a lot of traffic or have many long-running operations, the dedicated or premium tiers might be more cost-effective. The dedicated plan involves renting control of a virtual machine and a dedicated Azure App Service plan. This control means you can do whatever you like on that machine. It's always available and might make more sense financially if you have functions that need to run 24/7. The premium plan (sometimes referred to as Elastic Premium plan) provides the following benefits to your functions:

   - Avoid cold starts with perpetually warm instances.
   - Virtual network connectivity.
   - Unlimited execution duration, with 60 minutes guaranteed.
   - Premium instance sizes: one core, two core, and four core instances.
   - More predictable pricing, compared with the Consumption plan.
   - High-density app allocation for plans with multiple function apps.

   When you use the premium plan, instances of the Azure Functions host are added and removed based on the number of incoming events, just like with the Consumption plan. You can also deploy multiple function apps to the same Premium plan, and it allows you to configure the size characteristics of your compute instance.

   You can learn more by reading the [Azure Functions hosting options](https://docs.microsoft.com/azure/azure-functions/functions-scale) article in the Azure documentation.

3. How do we make sure that erroneous image processing does not make specific toll bills fall through the cracks or, even worse, send an invoice to the wrong person?

   Erroneous image processing is handled in our solution by creating a manual verification queue when the OCR process fails, or the result's confidence level is low. For example, we can filter out invalid characters and use an empty result to indicate low confidence in the predicted license plate text. More advanced machine learning models may be employed with algorithms specifically tuned to license plate recognition (LPR). Part of the result would include a confidence level, which the system could use to decide whether the photo needs manual verification.

4. We are considering creating an API to allow our customers to retrieve information about their bills and photos that captured their vehicle but are concerned about unauthorized access and the possibility of a denial of service attack due to an excessive number of requests. Is there an Azure service that would enable us to implement a secure API that we can also monitor and manage, protecting our APIs from unauthorized access and excessive requests?

   Given the nature of Contoso's serverless architecture, they can quickly expand functionality by adding more Azure functions. For instance, they can add a new function with an HTTP trigger that runs when a client hits the endpoint. This function can process GET requests with a customer Id or custom identifier, which will return information for that customer's vehicles and any captured photos.

   If Contoso would like to add a layer of protection, they should consider using [Azure API Management](https://docs.microsoft.com/azure/api-management/api-management-key-concepts) (APIM). This service provides a consistent surface area for API endpoints you wish to expose to the public, including the new Azure function that will serve customers' information.

   APIM offers several pricing tiers. Contoso should consider using the new consumption tier, allowing API Management to be used in a serverless fashion, with instant provisioning, automated scaling, built-in high availability, and pay-per-action pricing. This consumption model is an excellent match for their Azure Functions consumption plan, enabling Contoso to pay for only the resources they use.

   There are many benefits to using an API management, including:

   - **Security**: the API manager layer verifies the incoming requests' [JWT](https://jwt.io/) token against their authority URL, such as an Azure Active Directory B2C Authority URL. This is accomplished via an inbound policy that intercepts each call:

     ```xml
     <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
         <openid-config url="<--your_own_authorization_url-->" />
         <audiences>
             <audience><-- your_own_app_id --></audience>
         </audiences>
     </validate-jwt>
     ```

   - **Documentation**: API management provides developers writing applications against Contoso APIs with a complete development portal for documentation and testing.

   - **Usage Stats**: Usage stats on all API calls (and report failures) are available, making it convenient to assess the API performance.

   - **Rate Limiting**: You can configure rate limiting on APIs based on IP origin, access, etc. This feature helps prevent DDOS (distributed denial-of-service) attacks and can provide different tiers of access based on users.

   **Please note** that, in the case of Azure Functions, while the APIs are front-ended with an API manager (and hence shielded, protected, and rate-limited), the APIs are still publicly available, meaning that a DDOS attack or other attacks can still happen against the bare APIs if someone discovers them in the wild. One way to reduce this exposure is to ensure that calls to the function always go through APIM. Also, Contoso should configure their Azure Function App to [accept requests only from the IP address of your APIM instance](https://docs.microsoft.com/azure/app-service/app-service-ip-restrictions). After doing this, they should set the HTTP-triggered function authentication level to `anonymous`.

5. What is our best option to protect application secrets, such as connection strings, from being viewed by unauthorized users in the portal?

   Azure Key Vault can be used to securely store and tightly control access to tokens, passwords, certificates, API keys, and other secrets. Also, secrets stored in Azure Key Vault are centralized, giving the added benefits of only needing to update secrets in one place, such as an application key value after recycling the key for security purposes. Contoso can store their secrets in Azure Key Vault, then configure their Function Apps to securely connect to Azure Key Vault by performing the following steps:

   1. Create a key vault and add secrets to it.
   2. Create a system-assigned managed identity for each Azure Function App or Web App that needs to read from the vault.
   3. Create an access policy in Key Vault with the "Get" secret permission assigned to each application's identity.

   After these steps are completed, Contoso can update their application settings for their connected Function Apps and Web Apps, using Key Vault references for each secret in the form of `@Microsoft.KeyVault(SecretUri={referenceString})`. `{referenceString}` is the **SecretUri** URI of a secret in Key Vault, including a version. For example, `@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/ec96f02080254f109c51a1f14cdb1931)`. The secret's value will automatically be extracted from Key Vault by the App Service and passed to the deployed application code, just like any other application setting.

6. There appears to be some functional overlap between Azure Functions and Logic Apps. Can you help us to understand better when to use each and for what?

   Both Functions and Logic Apps are Azure services that enable serverless workloads capable of using complex orchestration steps to accomplish tasks. The primary differentiator between them is that Azure Functions is a serverless compute service, whereas Azure Logic Apps provides serverless workflows. You can mix and match services when you build an orchestration, calling functions from logic apps and calling logic apps from functions.

   Functions emphasize code-first versus the design-first approach offered by logic apps. For Azure Functions, you write code in various languages, like C#, Node.js, Python, Java, and PowerShell, and you can add orchestration using the [Durable Functions](https://docs.microsoft.com/azure/azure-functions/durable/durable-functions-overview?tabs=csharp) extension to create stateful workflows. You can secure Azure Functions using Authorization keys that are automatically generated and implement additional security measures using the dedicated and premium tiers. For Logic Apps, you create orchestrations using a graphical user interface (GUI) or editing configuration files. Logic Apps makes implementing security very simple. Every connector includes the authentication components needed to connect to the target service, such as the Office 365 Outlook connector requires an Office 365 account.

7. During our research on serverless architectures, we have read about the possibility of increased latency due to cold starts. Will this be an issue with Azure Functions?

   Cold start is a significant discussion point for serverless architectures and is a point of ambiguity for many users of Azure Functions. Concerning Azure Functions, a cold start is only a concern when your Functions use the serverless consumption model. When using Azure Functions in the dedicated or premium plans, the Functions host is always running, which means that cold start is not an issue.

   By cold start, we are broadly describing the phenomenon that applications that have not been used take longer to start up. In the context of Azure Functions, latency is the total time a user must wait for their function to execute. This time runs from when an event happens to trigger a function until it completes responding to the event. More precisely, a cold start is an increase in latency for Functions that haven’t been called recently.

   Cold start happens due to the serverless nature of the Azure Functions consumption plan. When you execute function code, the following steps must occur to run your code:

   - Azure must allocate your application to a server with capacity.
   - The Functions runtime must then start up on that server.
   - Your code then needs to execute.

   To make this experience better for users, instead of starting from scratch every time, Microsoft has implemented a way to keep a pool of servers warm and draw workers from that pool. This pool means that idle workers preconfigured with the Functions runtime are up and running and available at any point in time. These “pre-warmed sites” provide a measurable improvement on cold start times.

   Other suggestions to improve this are to use only generally available (GA) languages when writing your function code and focus on writing efficient, lightweight code with minimal dependencies. When you deploy your code, your dependencies are added as files to your application. All code needed by your app eventually gets loaded into memory, which takes longer with a larger application. So, if you have many dependencies, you may experience a longer cold start due to increased time for I/O operations from Azure Files and the increased time needed to load the app into memory.

   If cold start times are still an issue, consider using Azure Functions in the dedicated or premium plans, as a cold start is not an issue.

## Customer quote 

"Thanks to Azure's serverless components, and the ease in which we can use them, we have been able to rapidly build a cost-effective and robust solution to replace our manual license plate recognition process. The first-class monitoring tools we can use with our new architecture have helped us confidently move forward and competently meet the high demands of our expanding customer base."

--- Abby Burris, CIO, Contoso, ltd.
