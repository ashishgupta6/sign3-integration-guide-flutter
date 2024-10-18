# Sign3 SDK Integration Guide

The Sign3 SDK is a fraud prevention toolkit designed to assess device security, detecting potential risks such as rooted devices, VPN connections, or remote access, and much more. By providing insights into the device's safety, it enhances security measures against fraudulent activities and ensures a robust protection system.

## Create a .env file

1. In the root directory of your Flutter project, create a .env file.
2. Add the following content to the .env file:

``` credential
REPO_USERNAME=provided in credential doc
REPO_PASSWORD=provided in credential doc
REPO_URL=https://sign3.jfrog.io/artifactory/intelligence-test-local/
 ```

## Adding Sign3SDK to Your Project

### Using Project Level Gradle Dependency
1. **Add Sign3SDK to the Dependency Block**
   - In the android folder of your Flutter project, open the project-level `build.gradle` file and add the following line to the dependencies block. You can collect the **username** and **password** from the credentials document.

```groovy
def envProperties = new Properties()
def envFile = rootProject.file("../.env")
if (envFile.exists()) {
    envFile.withInputStream { stream ->
        envProperties.load(stream)
    }
    println ".env file loaded successfully."
} else {
    println "Error: .env file not found."
}

allprojects {
   repositories {
       def repositoryUrl = envProperties['REPO_URL'] ?: ""
       def username = envProperties['REPO_USERNAME'] ?: ""
       def password = envProperties['REPO_PASSWORD'] ?: ""
       if (!repositoryUrl.isEmpty() && repositoryUrl.startsWith("https://") && !repositoryUsername.isEmpty() && !repositoryPassword.isEmpty()) {
           maven {
               url repositoryUrl
               credentials {
                   username = username
                   password = password
               }
           }
       } else {
           println "Error: Invalid repository URL or missing credentials in .env file."
       }
   }
}
```
<br>

## Initializing the SDK
1. Initialize the SDK in the `onCreate()` method of your Application class.
2. Use the ClientID and Client Secret shared with the credentials document.
<br>

## Get Session ID

1. The Session ID is the unique identifier of a user's app session and serves as a reference point when retrieving the device result for that session.
2. The Session ID follows the OS lifecycle management, in line with industry best practices. This means that a user's session remains active as long as the device maintains it, unless the user terminates the app or the device runs out of memory and has to kill the app.
 
 ```dart
Future<void> getSessionID() async {
  try {
    var sessionId = await Sign3Intelligence.getSessionId();
  } catch (e) {
    // Handle the error message 
  }
}
```
<br>

## Fetch Device Intelligence Result

1. To fetch the device intelligence data refer to the following code snippet.
2. IntelligenceResponse and IntelligenceError models are exposed by the SDK.

```dart
Future<void> getIntelligence() async {
  try {
    var sign3IntelligenceResponse = await Sign3Intelligence.getIntelligence();
    // Do something with the response
  } catch (e) {
    // Handle the error message 
  }
}
```
<br>

## Optional Parameters
1.	You can add optional parameters like UserId, Phone Number, etc., at any time and update the instance of Sign3Intelligence.
2.	Once the options are updated, they get reset. Clients need to explicitly update the options again to ingest them, or else the default value of OTHERS in userEventType will be sent to the backend.
3.	You need to call **getIntelligence()** function whenever you update the options.
4.	To update the Sign3Intelligence instance with optional parameters, including additional attributes, you can use the following examples.

```dart
Future<void> updateOptions() async {
  try {
    Map<String, String> additionalAttributes = {
      "TRANSACTION_ID": "<TRANSACTION_ID>",
      "DEPOSIT": "<AMOUNT>",
      "WITHDRAWAL": "<AMOUNT>",
      "METHOD": "UPI/CARD/NET_BANKING/WALLET",
      "STATUS": "SUCCESS/FAILURE",
      "CURRENCY": "USD/INR/GBP/etc.",
      "TIMESTAMP": DateTime.now().millisecondsSinceEpoch.toString(),
    };

    UpdateOptions updateOptions = UpdateOptionsBuilder()
        .setPhoneNumber("6265257963")
        .setUserId("123456")
        .setPhoneInputType(PhoneInputType.GOOGLE_HINT)
        .setOtpInputType(OtpInputType.AUTO_FILLED)
        .setUserEventType(UserEventType.TRANSACTION)
        .setMerchantId("962328457268263")
        .setAdditionalAttributes(additionalAttributes)
        .build();
    
    await Sign3Intelligence.updateOptions(getUpdatedOptions());
  } catch (e) {
    // Handle the error message 
  }
}

Future<void> getIntelligence() async {
  try {
    var sign3IntelligenceResponse = await Sign3Intelligence.getIntelligence();
    // Do something with the response
  } catch (e) {
    // Handle the error message 
  }
}
```
<br>

## Sample Device Result Response

### Successful Intelligence Response

```response
{
  "requestId": "9ca5fa19-cc8c-4d67-ac43-ae301e50478a",
  "newDevice": false,
  "deviceId": "6c805eea-cb6a-4a1f-8896-eb7b4804556d",
  "vpn": false,
  "proxy": false,
  "emulator": true,
  "remoteAppProviders": false,
  "mirroredScreen": false,
  "cloned": false,
  "geoSpoofed": false,
  "rooted": false,
  "sessionRiskScore": 12.78,
  "hooking": true,
  "factoryReset": false,
  "appTampering": false
}

```
### Error Response

```error
{
  "requestId": "7e12d131-2d90-4529-a5c7-35f457d86ae6",
  "errorMessage": "Sign3 Server Error"
}
```
<br>

## Intelligence Response

The intelligence response includes the following keys:

- **requestId**: 
  - A unique identifier for the specific request.
  
- **newDevice**: 
  - Indicates if the device is new.
  
- **deviceId**: 
  - A unique identifier for the device.
  
- **vpn**: 
  - Indicates whether a VPN is active on the device.
  
- **proxy**: 
  - Indicates whether a proxy server is in use.
  
- **emulator**: 
  - Indicates if the app is running on an emulator.
  
- **remoteAppProviders**: 
  - Indicates whether any remote applications are installed on the device.
  
- **mirroredScreen**: 
  - Indicates if the device's screen is being mirrored.
  
- **cloned**: 
  - Indicates if the user is using a cloned instance of the app.
  
- **geoSpoofed**: 
  - Indicates if the device's location is being faked.
  
- **rooted**: 
  - Indicates if the device has been modified for root access.
  
- **sessionRiskScore**: 
  - A score representing the risk level of the session.
  
- **hooking**: 
  - Indicates if the app has been altered by malicious code.
  
- **factoryReset**: 
  - Indicates if a suspicious factory reset has been performed.
  
- **appTampering**: 
  - Indicates if the app has been modified in an unauthorized way.


