# Android FCM Push Notification Setup

## Pre-requisites
- PingFederate and P1MFA/P14C is set up and configured.
    - Ideally you have email/sms OTP working.
    - This can be performed manually or via Postman scripts and Docker. See [here](README.md) for more details.
- A Google account.
- Android Studio set up with the [P1MFA Mobile App](https://github.com/pingidentity/pingone-customers-mobile-sdk-android) loaded and ready.

## Create a Firebase Project

1. Launch the [Firebase Console](https://console.firebase.google.com/) with your Google account.
2. Add a Project
    - Enter a Project Name e.g. "My Company MFA App".
3. Create a new Android App configuration.
    - Enter a unique Package Name and record it down. E.g. com.mycompany.p1mfa.app
    - Enter in an App Nickname. E.g. My Company Authenticator
4. Download google-services.json
5. Open the new Project e.g. "My Company MFA App".
6. Edit the new Android app e.g. "My Company Authenticator".
7. Under the "Cloud Messaging" tab, locate and record the Server Key value. This will be used later to set up P1MFA/P14C.

## Configure P1MFA/P14C to enable Android Push Notification

### Option 1: Configure with Postman
1. Edit the provided [Postman Collection](postman_setup_p1mfa.json) variables and set the following with the details from your FCM application:
    - androidFCMPackageName
    - androidFCMServerKey
2. Execute the Postman requests under "3 Create Applications\PF Adapter EndUser\Enable Android Push Notifications"

### Option 2: Configure manually
1. Edit your Native application e.g. "PF Adapter EndUser".
    - Under the Authenticator tab, apply the following details with the details from your FCM application:
        - Package Name
        - Server Key
2. Edit your MFA Only Policy and enable Push Notifications.

## Set up your Android application

1. Launch the Android Studio P1MFA project.
2. Copy google-services.json into the "app" folder.
3. Edit build.gradle (under the "app" folder).
    - Update: 
        applicationId "com.mycompany.p1mfa.app"
4. Click menu: File -> Sync Project with Gradles.
5. Build the project under menu: Build -> Rebuild Project.
6. If you haven't configured an Android Virtual Device (AVD):
    - Click menu: Tools -> AVD Manager
    - Add a new Virtual Device (e.g. Pixel 3)
    - When asked, select API Level 29 or greater.
7. In some dev environments, Android Studio may not let you configure RAM settings appropriately. E.g. changing a value for RAM does not take effect.
    - You may need to edit config.ini on disk. See this [stackoverflow thread](https://stackoverflow.com/questions/49454328/android-emulator-ram-not-extending-for-google-play-api-27) for instructions.
    - You may wish to increase this value to something more representative of the device e.g. 4096.
8. Run the application by clicking menu: Run -> Run 'app'.

## Pair your device

1. Run the [MFA Pairing Key POST API](https://apidocs.pingidentity.com/pingone/platform/v1/api/#mfa-pairing-keys) to create a new Pairing Key.
    - Alternatively, run the provided [Postman script](postman_setup_p1mfa.json) under "2 Create Users\Pair mobile" to generate a pairing key.
    - The pairing key is returned from the "code" claim and is a 14 digit number that looks like "01100397496298".
2. Enter the pairing key into your phone and click "Pair".
3. The next time you log into an application (e.g. [this url](https://localhost:9031/as/authorization.oauth2?client_id=sampleclient&response_type=token)), it should give you the option to authenticate with Push.
    
