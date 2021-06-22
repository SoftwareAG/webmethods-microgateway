webMethods Microgateway
========================

webMethods Microgateway is a lightweight distributed proxy that gives control over a microservices landscape by enforcing policies which perform authentication, traffic monitoring, and traffic management near the service end points. The lightweight nature of the Microgateway allows a flexible deployment to avoid gaps or bottlenecks in the policy enforcement. Microgateway enables microservices to communicate with each other directly without the need for API Gateway. This eases the traffic overload on API Gateway and reduces the latency in the communication round trip. The required protection policies can be enforced on the Microgateway to establish a secure communication channel between the microservices. 

![](attachments/Microgateway_Int0.png)

The above diagram shows microservices implementing the booking related API of the imaginary company NodeTours. The shown microservice mesh or microservice landscape consists of three services. Consumer applications access the APIs via a central API Gateway that performs the enforcement of policies. The problem here is that each microservice exposes an endpoint where no policy enforcement is done.  Moreover, considering the microservices are interacting with each other all this traffic needs to be routed through the single API Gateway. This leads to additional network latency and the API Gateway potentially becomes a bottleneck. The diagram below shows a deployment where the API Gateway is replaced with a set of Microgateways that are added to the microservices themselves. Such a "sidecar" deployment does not leave any gaps and avoids bottlenecks in the policy enforcement. 

![](attachments/Microgateway_Int.png)

API Gateway Integration
----------------------------

The Microgateway’s responsibility is focused on a single microservice or a small number of microservices. To manage a microservice landscape an API Gateway is needed. It offers the UI for configuring policies and for system configurations. Moreover, it is responsible for monitoring the traffic across the microservice landscape. The following figures shows how Microgateways are interacting with an API Gateway.

![](attachments/Microgateway_APIGWInt.png)

API Gateway vs Microgateway
----------------------------

API Gateway is used to expose services that provides complete end to end capability. API Gateways are generally designed as centralized gateways hosted on cloud or on-premise. Microgateways are designed as decentralized gateways that can hold single or few APIs. The flexible deployment nature of Microgateway makes it ideal for low latency small footprint solutions. Microgateways are best suited for microservices architectures with greater flexibility of horizontal scaling. 

Components of Microgateway
---------------------------

The Microgateway service runs within its own Java runtime environment and is controlled by a simple command line interface that supports basic lifecycle operations like start and stop. The configuration of the service consists of system settings and assets that can be provisioned from a running API Gateway or can be provisioned through a filesystem. The provisioned assets include Application, API, Policy, and Alias definitions. The Microgateway service exposes an administrator REST API to query the status, the system setting, and the provisioned assets. Microgateway has a service to enforce the policies on REST APIs. 

Get Microgateway
--------------------

Software AG Microgateway is part of webMethods API Management platform (link - https://www.softwareag.com/en_corporate/platform/integration-apis/api-management.html). webMethods API Gateway can be provisioned in multiple ways based on the users needs. 

Docker - Microgateway docker image can be pulled from the docker hub. https://hub.docker.com/_/softwareag-microgateway-trial
After the checkout, check the "How to use" page which lists the detailed steps for spinning up a container with the docker image. Please follow the steps and get your instance started in few minutes.

On-Premise Installation – Microgateway on-premise installation can be downloaded and installed from Software AG empower. link - https://empower.softwareag.com/

Getting Started
-------------------------

Let's deploy the microgateway on an API to see how it is provisioned and how it works. This tutorial uses the the [Studio Ghibli API](https://github.com/janaipakos/ghibliapi) which lists information about the animes created by Studio Ghibli. You can try listing all their films with the below Postman call and its swagger definition can be found [here](https://github.com/SoftwareAG/webmethods-microgateway/blob/main/attachments/StudioGhibliAPIswagger.yaml).
    
    GET https://ghibliapi.herokuapp.com/films/

And a specific film can be listed with its id, for example:
    
    GET https://ghibliapi.herokuapp.com/films/2baf70d1-42bb-4437-b551-e5fed5a87abe

1. Login to docker hub and sign up for Software AG Microgateway Trial image using [this link](https://hub.docker.com/_/softwareag-microgateway-trial)

2. Pull the microgateway image using the below command

		docker pull store/softwareag/microgateway-trial:10.7

3. Download [this archive](https://github.com/SoftwareAG/webmethods-microgateway/blob/main/attachments/StudioGhibliAPI.zip) to your computer and place it in ~/archive folder or another folder you prefer. It contains the archive files to define the API in the microgateway and apply a data masking policy for two different scopes

4. Now, let's provision the microgateway container with the following command. (Make sure to use the folder you specified in the previous step instead of ~/archive before colon)
    
	    docker run -d -v ~/archive:/opt/softwareag/Microgateway/archives -p 9090:4485 -e mcgw_archive_file=/opt/softwareag/Microgateway/archives/StudioGhibliAPI.zip --name microgateway store/softwareag/microgateway-trial:10.7
    
    This command volume mounts the ~/archive folder to the /opt/softwareag/Microgateway/archives folder inside the container so that the archive file is accessible from inside the container, starts the gateway on port 9090, and tells it to use the given archive file. 

5. Once the container is up and running, you can see the microgateway status and the assets controlled by it with the following curl commands

		curl http://localhost:9090/rest/microgateway/status
	
    	curl http://localhost:9090/rest/microgateway/assets 

6. Now let's see if the microgateway works. Let's invoke the microgateway endpoint with the following Postman call

    	GET http://localhost:9090/gateway/StudioGhibliAPI/1.0/films/

    You will see in the response body that the original title in Japanese spelling is masked by the microgateway

7. Let's try making a call to a specific film, for example:

    	GET http://localhost:9090/gateway/StudioGhibliAPI/1.0/films/2baf70d1-42bb-4437-b551-e5fed5a87abe

    This time, you will see that the Japanese spelling of the title is displayed, but its spelling with Roman characters is masked. Using the microgateway you can define different scopes for resources and methods, and set different policies for each scope.

To take advantage of all the policies that are supported by the microgateway, you can download and set up The API Gateway and download the asset archives from it. 
 
 ### Using The Microgateway with the API Gateway ###

Prerequisite: [Install API Gateway](https://github.com/SoftwareAG/webmethods-api-gateway), download the Petstore swagger from https://petstore.swagger.io/v2/swagger.json and create REST API in API Gateway called Petstore.

1. Apply any of the supported policies by the microgateway on the Petstore API from the Gateway.

2. Start Microgateway container using the below command

		docker run -p 9090:4485 \ 
		--env mcgw_api_gateway_url=http://sample.com:5555/rest/apigateway \ 
		--env mcgw_api_gateway_user=Administrator \ 
		--env mcgw_api_gateway_password=password \ 
		--env mcgw_downloads_apis=Petstore \ 
		--name microgateway store/softwareag/microgateway-trial:10.7

    This command connects the microgateway to the API Gateway with the given username and password and pulls the Petstore API and its associated policies to the microgateway. 

4. Below curl commands can be used to verify the status of Microgateway, its assets and the applied policies

		curl http://localhost:9090/rest/microgateway/status 

		curl http://localhost:9090/rest/microgateway/assets 

5. You can invoke the microgateway endpoint of the API and see the result of the applied policies

### Stopping the Microgateway Container ###

Stop the Microgateway container using the docker stop command

docker stop -t90 microgateway

The docker stop is parameterized with the number of seconds required for a graceful shutdown of the Microgateway container.

Note: The docker stop does not destroy the state of the Microgateway. On restarting the Docker container all assets that have been created or configured are available again.

Policies in Microgateway
-------------------------

Microgateway supports a comprehensive collection of policies you can configure in the API Gateway and provision to Microgateway:
* Transport: Microgateway supports Enable HTTP/HTTPs policy in transport. 
* Identify and Access: Policies supported by Microgateway for Identify and Access are: 
	* Authorize user - Microgateway authorizes users against copy of IntegrationServer\instances\default\config\users.cnf which is created during Microgateway start-up. This policy should be used together with an authentication policy (for example, Require HTTP Basic Authentication). Authorize user policy validates the incoming request against list of users, groups or access profiles. 
	* Identify and Authorize application - This policy allows client applications to access APIs depending on the identification type specified. Identification types supported are API Key, Hostname, HTTP Basic Authentication, IP address range, OAuth2 token, JWT, OpenID connect, SSL Certificate and Payload element.
	Microgateway is created using the required applications pulled during start-up. Any changes to applications are synched using polling mechanism.  To avoid the consumption of a considerable amount of memory and CPU, the API Provider provides certain configurations for polling the applications. Ex: list of application ids, registered applications of the APIs in Microgateway or all global applications.
* Request Processing: Policies supported in request processing are:
	* Request transformation - Allows to configure several transformations on the request messages from clients into a format required by the native API before it is submitted to the native API.
	* Validate API specification - This policy validates incoming request against schema, query parameters, path parameters, content-types, and HTTP headers.
	* Data Masking - Data masking is a technique of masking sensitive fields. Data masking is applied at application level. It mandates Identify and Access policy to be configured to identify an application. If no application is specified masking is applied on all requests. Masking criteria can be applied to XPath, JSONPath, and Regex expressions based on the content-type. This policy can also be applied on API scopes. 
* Routing: Microgateway supports routing policies as below:
	* Straight-through routing
	* Content-based routing
	* Context-based routing
	* Out-bound Authentication - Transport
* Traffic Monitoring - Microgateway supports the following Traffic Monitoring policies:
	* Log Invocation - In log invocation policy Microgateway supports the below properties. Store Request, Store Response, Compress Payload data, Log Generation Frequency and Destination (API Gateway and Elasticsearch).
	* Monitor Service Level Agreement - This policy monitors a set of run-time performance conditions like availability, average response time, fault count, maximum response time, success count and total request count for one or more specified applications. This policy can be used to configure SLAs for each API or application or combination of both. If there is a breach in any of the parameters, an event notification (Monitor event) is sent to the configured destination. In a single policy, multiple action configurations behave as AND condition. The OR condition can be achieved by configuring multiple policies. Microgateway supports API Gateway and Elasticsearch as destinations. This policy action monitors run-time performance conditions within a single Microgateway instance. 
	* Monitor Service Performance - This policy is like the Monitor Service Level Agreement policy but enforces the run-time performance conditions at the API level. Parameters like success count, fault count, and total request count are immediate monitoring parameters and are evaluated as soon as the limit is breached. For the rest of the aggregated monitoring parameters, the evaluation happens after the configured interval. If there is a breach in any of the parameters, an event notification (Monitor event) is sent to the configured destination. Destinations supported are API Gateway and Elasticsearch. In a single policy, multiple action configurations behave as AND condition. The OR condition can be achieved by configuring multiple policies. This policy only monitors run-time performance conditions within a single Microgateway instance. 
	* Throttling Traffic Optimization - This policy limits the number of API invocations during a specific time interval. This policy only limits the number of API invocations within a single Microgateway instance. For destinations API Gateway and Elasticsearch is supported.
* Response Processing - Microgateway supports the below response processing policies: 
	* Response Transformation - Microgateway supports the following parameter types for the response transformation policy. response.payload, response.headers, response.statusCode and response.statusMessage
	* Validate API Specification - This policy validates outgoing response against schema, content-types, and HTTP headers.
	* CORS - The Cross-Origin Resource Sharing (CORS) mechanism is used to secure cross-domain requests and data transfers between browsers and web servers. The CORS standard works by adding new HTTP headers that allow servers to describe the set of origins that are permitted to read that information. The CORS response specifications that are supported by Microgateway are Allow Origins, Max age, Allowed Methods, Allow Headers, Allow Credentials and Expose Headers. 
	* Data Masking - Data masking is a technique of masking sensitive fields. Data masking is applied at application level. It mandates Identify and Access policy to be configured to identify an application. If no application is specified masking is applied on all responses. Masking criteria can be applied to XPath, JSONPath, and Regex expressions based on the content-type. This policy can also be applied on API scopes. 
* Error Handling - Microgateway supports Conditional Error Processing and Data Masking in Error Handling policies. 
* API Scopes - A scope represents a logical grouping of REST resources, methods, or both in an API. An API can have a set of declared scopes. API scopes are configured in API Gateway and provisioned to Microgateway. Only the following policies are supported for API Scopes in Microgateway.
	* Identify and Access policy: Identify and Authorize Application
	* Traffic Monitoring policies: Log Invocation, Throttling Traffic optimization

References
-------------------------------------------

https://tech.forums.softwareag.com/t/running-microgateway-10-5-in-a-windows-container/237522
https://tech.forums.softwareag.com/t/getting-started-with-webmethods-microgateway-dockerhub-image/237332
https://tech.forums.softwareag.com/t/microgateway-instance-based-provisioning/239201
https://tech.forums.softwareag.com/t/microgateway-configuration/239202
https://tech.forums.softwareag.com/t/managing-microgateways/239203
https://tech.forums.softwareag.com/t/microgateway-integration-with-service-registry-and-api-portal/237259

Please also make sure to check out our API Gateway and have a look at the [API Gateway GitHub repository](https://github.com/SoftwareAG/webmethods-api-gateway).

-------------------------------------------
These tools are provided as-is and without warranty or support. They do not constitute part of the Software AG product suite. Users are free to use, fork and modify them, subject to the license agreement. While Software AG welcomes contributions, we cannot guarantee to include every contribution in the master project.

Contact us at [TECHcommunity](mailto:technologycommunity@softwareag.com?subject=Github/SoftwareAG) if you have any questions.
