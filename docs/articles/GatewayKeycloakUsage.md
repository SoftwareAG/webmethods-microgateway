# API Gateway and Microgateway with Keycloak Authentication Server

The following picture describes the architecture how API Gateway and Microgateway is using OAuth2 authentication with an Authentication Server, in this case the keycloak server.

![control flow scope](https://github.com/andreas-h-schmidt/samples/blob/main/pictures/arc.jpg?raw=tue)


1. Login as a registered user to keycloak and acquire a token (per REST request). It must know the user client ID and its client secret.
1. Keycloak delivers a token for the specified user client
1. The user performs the request to API Gateway with the token inside the Authorization HTTP header with Bearer mode
1. The OAuth2 policy of the API performs the introspection request to validate the client’s token 
1. Keycloak returns a JSON document and tells if the token is valid (true) or not (false)
1. For a valid token the processing continues resulting in success (response 200), for invalid token the request terminates immediately with response 401.

## Keycloak Server

To try a keycloak server it is convenient to establish him as a docker image. Just pull the server from DockerHub and start the container. The following examples uses “daefermion1” as host.

```sh
login to daefermion1
docker pull jboss/keycloak
docker run -d -p 8080:8080 -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin 
       --name keycloak jboss/keycloak 
```

If the server is up, then just login as admin into the Administration console (http://daefermion1:8080/auth). Then create the necessary assets:

#### Create Realm
Create a realm with “Master -> Create realm”, for example “myrealm”. All further created parts will be part of the realm

#### Create User
Create a user with its password, for example “myuser”.  Then select the Credentials tab and specify its password. Those users are necessary for acquiring tokens.

![create realm](https://github.com/andreas-h-schmidt/samples/blob/main/pictures/kc1.jpg?raw=tue)

![create user](https://github.com/andreas-h-schmidt/samples/blob/main/pictures/kc2.jpg?raw=tue)

To verify the login of the user, open keycloak’s account console:\
http://daefermion1:8080/auth/realms/myrealm/account \
Complete password confirmation if not already done. Then the user is fully activated.

#### Create Clients

Clients are used for authentication requests to the server. The clients are identified with its ID and a secret, which is calculated by keycloak. Create two clients:

* API Gateway client to validate the token coming from a user request:
  * Client ID:		apigw
  * Client Protocol:		openid-connect
  * Access Type:		confidential
  * Valid redirect URI:	http://localhost:5555/*


* User client for retrieving an access token
  * Client ID:		myapp
  * Client Protocol:		openid-connect
  * Access Type:		confidential
  * Valid redirect URI:	http://localhost:5555/*

For selecting the confidential Access Type, you’ll find the Client Secret under the Credentials tab.

![create client](https://github.com/andreas-h-schmidt/samples/blob/main/pictures/kc3.jpg?raw=tue)

![create client](https://github.com/andreas-h-schmidt/samples/blob/main/pictures/kc4.jpg?raw=tue)

![client sectet](https://github.com/andreas-h-schmidt/samples/blob/main/pictures/kc5.jpg?raw=tue)


## API Gateway

### Create Authentication Server entry
Define the keycloak server as external authentication server:
_Administration -->  Security  -->  JWT/OAuth_

![API Gateway Auth Server sectet](https://github.com/andreas-h-schmidt/samples/blob/main/pictures/ag1.jpg?raw=tue)


Take over the values, created at keycloak server into the external authentication server configuration.

#### Introspection

_Issuer_\
Specify the created realm (“myrealm”) as URL in the “Issuer” field:\
**http://daefermion1:8080/auth/realms/myrealm**

_JWKS URI_\
Specify the according keycloak URL for possible SSL usage:\
**http://daefermion1:8080/auth/realms/myrealm/protocol/openid-connect/certs**

_Introspection endpoint_\
The introspection endpoint is used by the application to verify the specified token which will be delivered with user requests. It mainly delivers a json document with the active field, saying true or false, together with scope settings:\
**http://daefermion1:8080/auth/realms/myrealm/protocol/openid-connect/token/introspect**

_Client ID_\
Specify the Gateway client, present on keycloak together with its Client Secret.\
**apigw**

#### Metadata
Define the metadata URLs used by further actions, e.g. API Portal

_Authorize URL_\
**http://daefermion1:8080/auth/realms/myrealm/protocol/openid-connect/auth**

_Access Token URL_\
**http://daefermion1:8080/auth/realms/myrealm/protocol/openid-connect/token**

_Refresh Token URL_\
**http://daefermion1:8080/auth/realms/myrealm/protocol/openid-connect/token**

#### Scopes
Scope: Define according scope names present on keycloak which will be mapped with the API having the OAuth2 policy. Ensure that the scope names are matching.

### Create API with OAuth2 policy
![create api](https://github.com/andreas-h-schmidt/samples/blob/main/pictures/ag2.jpg?raw=tue)

### Create Application
Define an Application referring to keycloak Authentication server
![create app](https://github.com/andreas-h-schmidt/samples/blob/main/pictures/ag3.jpg?raw=tue)

Select the Authentication Server by its name and specify the User client name from the keycloak server as Client id. Specify the API, which has got the OAuth2 policy.

### Define a scope
Menu  -->  OAuth/OpenID
![create scope](https://github.com/andreas-h-schmidt/samples/blob/main/pictures/ag4.jpg?raw=tue)

## Requests in detail

#### Try a Token
At first let keycloak generate a token that can be used for the request to the API. As parameter the user client id with its secret together with the configured keycloak user and password is required for acquiring a token

```
POST http://daefermion1:8080/auth/realms/myrealm/protocol/openid-connect/token
	
Header:
Access-Type: application/x-www-form-urlencoded

Payload:
client_id         myapp
client_secret     secret-of-myapp
grant_type        password
username          myuser
password          mypass
```
The resulting payload contains a JSON document with the token.

This token can then be inserted as bearer-token into API requests, for example:
```
GET http://localhost:5555/gateway/BayernOAuth2/1.0/bezirke
	
Header:
Authorization: Bearer <token>
```

#### Try the introspection
You may also try the introspection request and see how keycloak validates. API-Gateway resp. Microgateway issues this request to the keycloak authentication server. The resulting payload contains a JSON document with the validation result.

```
POST http://daefermion1:8080/auth/realms/myrealm/protocol/openid-connect/token/introspect
	
Header:
Access-Type:  	application/x-www-form-urlencoded
Accept:	     	application/json
Authorization:	Basic  apigw / secret-of-apigw

Payload:
token_type_hint	access_token
token		<token>
```
For a successful validation it returns:
```
{
    scope:   "myscope roles",	
    active”: true
}
```


## Microgateway
For Microgateway there is a similar handling. It just needs the configuration of the Authentication server and the involved assets used with API Gateway. The introspection request to validate the token will be done by Microgateway itself.

#### Required assets and objects for Microgateway:

-	API with OAuth2 policy
-	Application with the Authentication section (keycloak name and User Client ID)
-	Gateway Scope
-	Authentication server alias (via yml file)

#### Use Microgateway accessing API Gateway

Prepare steps:
1. Prepare Authentication server, Scope, API and Application as for API Gateway (see above)
1. Start Microgateway with the download-settings option that all the necessary authentication server configurations are pulled from API Gateway.

Execute:
```bash
microgateway start -p 7070 
             -gw localhost:5555 -gwu Administrator -gwp pwd
             -ds true -apis BayernOAuth2
```

#### Use Microgateway with archive
To completely decouple Microgateway from API Gateway at runtime, start Microgateway with prepared export data:

Prepare steps:
1. Export the mapped scope with the “Include Application” option. This ensures that all required assets are within the archive:
_Administration  -->  OAuth/OpenID scopes  -->  keycloak-scope  -->  Export  -->  scopeArchive.zip_

2. Download the API Gateway configuration to a yml file
```
microgateway downloadSettings -gw localhost:5555 -gwu Administrator -gwp pwd -out config.yml
```

Execute:
```
microgateway start -p 7070 -c config.yml -a scopeArchive.zip
```

#### Check all the settings
Because there are so many assets and configuration settings in effect, you’ll have the ability to validate all required values, server, scope, policy with:
```
microgateway assets -p 7070 -v
```
```
{
    "apis": [{
        "apiName": "BayernOAuth2",
        "apiVersion": "1.0",
		...
		
    }],
    "applications": [{
        "name": "KeycloakApp",
        ...
    }],
    ...
    "authserver": [
      {
        "name": "Keycloak",
        "description": "Keycloak server on daefermion1",
        "clientId": "apigw",
        "remoteIntrospection": "http://daefermion1:8080/...",
        "scopes": "myscope",
        "mappedScopes": "myscope"
      }
    ]
}
```
Because Check that the API with the application is registered and the authserver entry is valid, especially the mappedScope information.

#### Try a token
Perform the same steps as shown in the API Gateway section. You need only replace the API Gateway port (5555) with the Microgateway port (7070).
```
POST http://daefermion1:8080/auth/realms/myrealm/protocol/openid-connect/token
	
Header:
Access-Type: application/x-www-form-urlencoded

Payload:
client_id         myapp
client_secret     secret-of-myapp
grant_type        password
username          myuser
password          mypass
```

Return:
```
{
    "access_token": "eyJhbGciOiJSUzI1NiI...
    "expires_in": 900,
    ...

```

Execute the request:
```
GET http://localhost:7070/gateway/BayernOAuth2/1.0/bezirke
	
Header:
Authorization: Bearer eyJhbGciOiJSUzI1NiI...
```
