# PingOne MFA for PingFederate Quickstart

## Pre-requisites
- P14C account or a P1MFA account
- Ping devops keys
- Docker installed and optionally docker-compose

## Features
PingFederate server profile which configures P14C for 1FA (using html form adapter and P14C PCV), and 2FA (using the new P14C MFA Adapter).

- Postman collection to set up an MFA environment.
- Configurable PingFederate server profile to configure end-to-end flow with the P1MFA IdP Adapter.
- Android Push Notification setup ([see here](android-push-setup.md))

## P14C Configuration

### Option 1 - Create P1MFA/P14C environment using Postman

A postman collection is provided to help you get you set up quickly. 

1. Create a Worker App client/connection in your P14C parent environment.
    - Name: Postman Collection
    - App Type: Worker
    - Leave everything else as default.
2. Import the [postman collection](postman_setup_p1mfa.json) into Postman.
3. Configure the collection variables (right click collection and select Edit).
Set the following collection variables:

| Variable Name | Description | Value
| --- | --- | ---
| parentEnvID | The Environment ID of the parent environment. | Found in P14C -> Settings -> Environment -> Properties.
| licenseType | The license type applied to your environment. | Found in P14C -> Settings -> Environment -> Properties.
| adminAppID | The Client ID value of the P1MFA/P14C worker application. | Worker app configured in your parent environment.
| adminAppSecret | The Client Secret value of the P1MFA/P14C worker application. | Worker app configured in your parent environment.
| apiPath | The base URL of the P1MFA/P14C API endpoint. | https://api.pingone.com/v1 or https://api.pingone.asia/v1 or https://api.pingone.eu/v1
| authPath | The base URL of the P1MFA/P14C AuthZ endpoint. | https://auth.pingone.com or https://auth.pingone.asia or https://auth.pingone.eu
| smsNumber | A valid phone number string can be provided in international format (includes a leading + character), or in any format as long as the region (country code) where the number originates can be determined. | Your mobile number.
| emailAddress | A valid email address | Your email address.
        
4. Configure an empty Postman environment.
5. Execute the Postman requests in sequence.
6. Collect environment details by running the last request "Get Environment Details" to configure later in override.env.
    - environmentId -> P14C_ENVIRONMENTID
    - pingfederate-connection-client_id -> P14C_PFCONNECTION_CLIENTID
    - pingfederate-connection-client_secret -> P14C_PFCONNECTION_CLIENTSECRET
    - application-client_id -> P14C_APPLICATION_CLIENTID
    - application-client_secret -> P14C_APPLICATION_CLIENTSECRET
    - Note down the test users (default: p1mfauser/2FederateM0re!)

### Option 2 - Manually create P1MFA/P14C environment

Alternatively you can run the following manual steps:

1. Configure an MFA only Authentication Policy.
     - Example name: "MFA-Only-Policy".
2. Create a PingFederate connection (P14C -> Connections -> Product Platform -> PingFederate).
     - Admin Role: MFA, PCV, and provisioning
     - Save the client ID and client secret as "PingOne Connection client details".
3. Create an application OAuth client in P14C.
     - Native app
       - Example name: PF Adapter End User Client
       - Scope: openid+profile
       - Response Types: ID Token, Token
       - Grant Type: Implicit
       - Policy: MFA-Only-Policy
     - Save the client ID and client secret as "PingOne application client details".
4. [Create a user](https://apidocs.pingidentity.com/pingone/platform/v1/api/#post-create-user), [enable MFA](https://apidocs.pingidentity.com/pingone/platform/v1/api/#put-update-user-mfa-enabled), add MFA devices ([sms](https://apidocs.pingidentity.com/pingone/platform/v1/api/#post-create-mfa-user-device-sms)|[email](https://apidocs.pingidentity.com/pingone/platform/v1/api/#post-create-mfa-user-device-email)).

## Launch docker and test

1. Pull this git project.
    - git clone https://github.com/ttranatping/p1mfa-serverprofile.git
2. Navigate to the p1mfa-serverprofile folder.
3. Configure override.env providing client details for the native and worker app.
    - Instantiate from override.env.template.
        - cp override.env.template override.env
    - At minimum, update the following:

| Environment Variable | Description | "Get Environment Details" value
| --- | --- | ---
| P14C_ENVIRONMENT_ID | The ID of the environment of the P1MFA instance (e.g. created using Postman) | environmentId
| P14C_PFCONNECTION_CLIENTID | The PF Connection Client ID (Postman: 3 Create Applications / PF Connection Client). | pingfederate-connection-client_id
| P14C_PFCONNECTION_CLIENTSECRET | The PF Connection Client Secret (Postman: 3 Create Applications / PF Connection Client). | pingfederate-connection-client_secret
| P14C_APPLICATION_CLIENTID | The Application Client ID (Postman: 3 Create Applications / PF Application Client). | application-client_id
| P14C_APPLICATION_CLIENTSECRET | The Application Client Secret (Postman: 3 Create Applications / PF Application Client). | application-client_secret

4. Start the docker container using one of the two methods:
    - docker-compose up -d
    - docker run -p 9031:9031 -p 9999:9999 -it --env-file override.env --env-file ~/.pingidentity/devops --rm  "pingidentity/pingfederate:edge"
5. Start the sample OAuth2 flow: 
    - https://localhost:9031/as/authorization.oauth2?client_id=sampleclient&response_type=token
6. Log in with a P14C user that has MFA enabled.
    - if you set up P14C/P1MFA using Postman:
      - the default user is: p1mfauser/2FederateM0re!
      - the default email is: p1mfauser@mailinator.com (you can check the OTP at mailinator.com).
    
