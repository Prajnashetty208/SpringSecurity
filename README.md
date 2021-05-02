Spring Cloud Security

Data Security
System Security
Network Security
Application Security


Application Security

Authentication

You are indeed the same person as you claim to be


Authorization

Do you have permissions to perform the job


Principal

An Authenticated user/app/resource




Authentication

Username and password
Fingerprint
Retina
Lock and Key
Chip and Pin
Swipe


OAuth2

OAuth2 is a framework to delegate part of the responsibility to a system to perform the Job


Steps for Oauth Integration

The client application onboards with the Oauth Server
The client will provide all the details while onboarding
The Oauth server will return back APP_ID and APP_SECRET which should be stored in the client application
The user will login to the client application
The client application will redirect the user to the OAuth2 Server
The client will present his credentials to the Oauth2 server
The OAuth2 server will validate the credentials and if successful, will redirect to the callback url with the Authorization code
The Auth code is not secure because it is transmitted through the front channel
The client application then presents the auth code along with APP_ID and APP_SECRET to the OAuth2 server
The  OAuth2 server will return ACCESS_TOKEN and REFRESH_TOKEN in response through the back channel
Once the ACCESS_TOKEN is received by the client application, it can perform the tasks without the user presence
The client application will present the ACCESS_TOKEN to the Resource Server
The Resource server will validate the ACCESS_TOKEN with the OAuth2 server
The validatity of ACCESS_TOKEN is limited to 10 minutes
The client application will fetch a new ACCESS_TOKEN from the OAuth2 server by presenting the REFRESH_TOKEN

The REFRESH_TOKEN is valid until it is purged and has to be stored securely by the client application


OAuth2 Grant Types

Authorization code grant flow
Client credentials
Password grant flow
Implicit grant flow


OAuth2 server

Spring Cloud OAuth2 server
Auth0
AWS Cognito
Okta


Lab 1 - Setup OAuth2 server

Create a Developer account in developer.okta.com
Add Users
Add Groups
Create application
Note down the below properties

client id- 0oa3eaj576gLDYwsh5d6
client secret- q02EUBXBN9JEgFfxhpNH_oeQ3p3FwJZoKnaEknxe
okta domain name - dev-9729512.okta.com
redirect URI - http://localhost:9222/login
Issuer URI - https://dev-9729512.okta.com/oauth2/edukart

Okta urls

Frame the auth url with the above configuration
OpenId:

<issuer-url>/v1/authorize?client_id=<client-id>&response_type=code&scope=openid,profile&redirect_uri=http%3A%2F%2Flocalhost%3A9222/login&state=state-296bc9a0-a2a2-4a57-be1a-d0e2fd9bb601


Create a class called OktaController and annotate the class with RestController


Add a GetMapping annotation to retrive the code and state



@RestController
public class OktaLoginController {

    @GetMapping(value = "/login")
    public String helloWorld(@RequestParam("code") String code, @RequestParam("state") String state) {
        if (state.equals("dummy-url-for-testing")){
            System.out.println("The request is not tampered");
            return "code:: "+ code + " state:: "+ state;
        }else {
            throw new IllegalArgumentException("Invalid code");
        }
    }
}

Run the server on 9222port



Access Token  can be accessed with the below curl command
curl --request POST \
  --url https://dev-9729512.okta.com/oauth2/edukart/v1/token \
  --header 'accept: application/json' \
  --header 'authorization: Basic MG9hM2VhajU3NmdMRFl3c2g1ZDY6cTAyRVVCWEJOOUpFZ0ZmeGhwTkhfb2VRM3AzRndKWm9LbmFFa254ZQ==' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'grant_type=authorization_code&redirect_uri=http%3A%2F%2Flocalhost%3A4200&code=XjYrJIPFbMCJEnj9_F3nQ8tvVugJqcUFldDINwmn5Cc'

Verify the access token with jwt.io
