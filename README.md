# authorization
Authorization workflow

Authorization
                     Kong, Gateway
                Oauth




    Keycloak is an open-source identity and access management solution, while Kong Gateway is a popular open-source API gateway. OAuth is a popular authentication and authorization protocol that is often used with these technologies.     
                 
—--------------------------------------------------------------------------------------------------------------------------------------------------------------- 

OVERVIEW
When using Keycloak and Kong Gateway together with OAuth, there are a few steps you can follow to enable load balancing upstream.
Configure Keycloak as the OAuth provider: Set up Keycloak to act as the OAuth provider, so that it can handle authentication and authorization requests.
Configure Kong Gateway: Set up Kong Gateway to use Keycloak as the OAuth provider, so that it can authenticate and authorize requests to your APIs.
Enable load balancing: To enable load balancing, you will need to configure your Kong Gateway to use an upstream server. This will allow Kong to distribute incoming requests across multiple backend servers.
Configure the upstream server: Configure the upstream server to handle incoming requests from Kong Gateway. This may involve setting up additional servers, load balancing, and other optimizations.
Test and monitor: Once everything is set up, test your system to ensure that everything is working as expected. You should also monitor your system to ensure that it is performing well and that there are no issues with the load balancing or other aspects of the system.
Overall, using Keycloak and Kong Gateway together with OAuth can provide a powerful and flexible authentication and authorization solution for your APIs. By enabling load balancing upstream, you can ensure that your system can handle large volumes of traffic and that it is highly available and performant.

Prerequisite

Here are the requirements for using Docker Compose to set up Keycloak and Kong Gateway with OAuth and load balancing upstream:

Docker: You will need to have Docker installed on your local system. Docker is a platform for building, shipping, and running applications in containers.

Docker Compose: You will also need to have Docker Compose installed on your local system. Docker Compose is a tool for defining and running multi-container Docker applications.

Keycloak image: You will need to use a Keycloak Docker image that is compatible with the version of Keycloak you want to use. You can find pre-built images on Docker Hub, or you can build your own image from the Keycloak source code.

Kong Gateway image: You will also need to use a Kong Gateway Docker image that is compatible with the version of Kong Gateway you want to use. You can find pre-built images on Docker Hub, or you can build your own image from the Kong Gateway source code.

UI - git repo, Clone frontend from Github 
API - git repo, Clone backendfrom Github 
Database


		

















LOCAL SETUP 


After up all services, you can see check docker logs, 
Here we have a frontend running in 3000 ports.
and we scale backend service into 3 ports ( 3311, 3312, 3313 )		

1) KEYCLOAK SETUP
ADD REALM
Log in to Keycloak with administrative privileges.
Click on the "Add realm" button on the left-hand side of the screen.
Enter name “Experimental ”


ADD CLIENT



Enter "myapp" in the "Name" field and click on the "Create" button.
On the left-hand side of the screen, click on the "Clients" tab, then click on the "Create" button.
Enter a name for your client, such as "myapp", and select "confidential" as the client type.
Under the "Access" tab, set "Valid Redirect URIs" to the callback URL of your application. For example, if your application is running on localhost, you could set it to "http://localhost:3000/*".
Under the "Credentials" tab, generate a new client secret and make a note of it.




	
ADD USER
Enter name and email with first name.. and set Email verified as true
Then click save button
	
Click on Role Mapping and give basic role access to that user 



Go to the credentials tab and set the password.. 
Then click on reset password

2) KONG SETUP
	Kong with plugins for CORS, OIDC, and upstream load balancing
Here are the steps you can follow:
ADD UPSTREAM
Create Upstream service name called “demo” , Hash on value “cookie”
and hash fallback “none”


set user define value for Hash on cookie as like “user_type”


Now, click on the “Targets page” of the upstream add all backend ports. 
( 3311,3312,3313 )

ADD SERVICE
click on Services -> add new Services 
create name as “upstream”
set host as “demo” (Upstream for load balance)

ADD PLUGINS
we have to add two plugins in this process OIDC , CORS
For OIDC run this curl
replace client_secret as <your client secret > 
client_id as <your client id>
		
curl --location 'http://localhost:8001/plugins' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'name=oidc' \
--data-urlencode 'config.client_id=kong' \
--data-urlencode 'config.client_secret=mvhPx5FgcBcjGc1hwjlM2mV1wnTrsxd8' \
--data-urlencode 'config.bearer_only=yes' \
--data-urlencode 'config.realm=experimental' \
--data-urlencode 'config.introspection_endpoint=http://192.168.29.92:8180/realms/experimental/protocol/openid-connect/token/introspect' \
--data-urlencode 'config.discovery=http://192.168.29.92:8180/auth/realms/experimental/.well-known/openid-configuration'
	
For cors plugin goto plugins->ADD GLOBAL PLUGINS->SECURITY
Now in this list, click on CORS->ADD PLUGINS
Then in ADD FORM set Header as Authorization, methods as GET and POST
Click on save changes


After adding both plugin, kong will display both active plugins in UI


3) UI
SETUP
Clone frontend code from git -> frontend 
Open the root directory and run “docker build -t frontend  . “
RUN
Then Run docker image by below code “docker run -p 3000:3000 frontend”
		This will up frontend services in local port 3000



TEST
Note : you have to start keycloak service, otherwise frontend will show 
“This site can’t be reached”


Config Details
     Go to -> cg_account_summary_frontend\.env

REACT_APP_KEYCLOAK_REALM=experimental (Reals name)
REACT_APP_KEYCLOAK_CLIENTID=myapp (Client id)
REACT_APP_KEYCLOAK_SSO_URL=http://192.168.29.92:8180/ 
(Keycloak Oauth URL)
REACT_APP_KEYCLOAK_REALM=experimental (Reals name)
REACT_APP_KEYCLOAK_CLIENTID=myapp (Client id)
REACT_APP_ACCOUNT_SUMMARY_API_BASE_URL=http://192.168.29.92:8000/
	(Target Kong to Resolve api and load balance )







4) API
SETUP
Clone frontend code from git -> backend
Open the root directory and run “docker build -t backend . “
RUN
Then Run docker image by below code “docker run -p 8088:8088 backend ”
		This will up frontend services in local port 3000
TEST
	Download docker-compose from this link
	https://drive.google.com/file/d/10DaoU3nx-_z5Bz5g22J6MZz074E5C2ZB
Config Details
DB_HOST = "localhost" ( DB host name )
DB_NAME = "demo_db" ( DB name )
DB_USER = "root" ( DB privilege User name )
DB_PASSWORD = "*******" ( DB User Password )
Integration Testing
1) UI TESTING
1) Open http://localhost:3000
		2) Click Login with Keycloak
		3) Give username, password click on login
		4) After successful login, we can access produced page, using header token
5) If We open the URL http://localhost:3000/?rsf_id=491

2) API TESTING
1) After all integration api will expose in local port ex:8000
		2) Download postman collection from below g-link 		https://drive.google.com/file/d/10DaoU3nx-_z5Bz5g22J6MZz074E5C2ZB/view?usp=sharing
		3) set postman environment variables 
		4) Then test get,post methods 

