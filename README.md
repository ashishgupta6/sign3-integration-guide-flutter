# Sign3 SDK Integration Guide for Flutter

The Sign3 SDK is a fraud prevention toolkit designed to assess device security, detecting potential risks such as rooted devices, VPN connections, or remote access, and much more. By providing insights into the device's safety, it enhances security measures against fraudulent activities and ensures a robust protection system.

## Adding Sign3 SDK to Your Project

### Add the latest version to your `pubspec.yaml` file.
1. We continuously enhance our fraud library with new features, bug fixes, and security updates. To stay protected against evolving fraud risks, we recommend updating to the latest SDK version.
   - Visit [latest_version](https://pub.dev/packages/fraud_sdk_flutter) for the latest version and check the Changelog for more details.

     ```dependency
     dependencies:
       flutter_sign3fraud: ^1.0.19
     ```

### Create `.env` file

1. In the root directory of your Flutter project, create a .env file
   - Add the following content to the `.env` file:
     
     ``` credential
     SIGN3_USERNAME=provided in credential doc
     SIGN3_PASSWORD=provided in credential doc
     SIGN3_REPO_URL=https://sign3.jfrog.io/artifactory/intelligence-generic-local/
      ```

### Using Project Level Gradle Dependency
1. **Add Sign3 SDK to the Dependency Block**
   - In the Android folder of your Flutter project, open the project-level `build.gradle` file, add the following line, and sync the project. You can collect the **username** and **password** from the credentials document.

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
            def sign3RepoUrl = envProperties['SIGN3_REPO_URL'] ?: ""
            def sign3Username = envProperties['SIGN3_USERNAME'] ?: ""
            def sign3Password = envProperties['SIGN3_PASSWORD'] ?: ""
            if (!sign3RepoUrl.isEmpty() && sign3RepoUrl.startsWith("https://") && !sign3Username.isEmpty() && !sign3Password.isEmpty()) {
                maven {
                    url sign3RepoUrl
                    credentials {
                        username = sign3Username
                        password = sign3Password
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
1. Create an `Application` class in the Android folder of your Flutter project, initialize the Sign3 SDK in the `onCreate()` method of the class, and add the Application class to your `AndroidManifest.xml` file.
2. Use the ClientID and Client Secret shared with the credentials document.

### For Java
```java
import io.flutter.app.FlutterApplication;
import android.os.Build;
import com.sign3.intelligence.fraud_sdk_flutter.OptionsBuilder;
import com.sign3.intelligence.fraud_sdk_flutter.Sign3IntelligencePlugin;

public class YourApplicationClassName extends FlutterApplication {
    @Override
    public void onCreate() {
        super.onCreate();
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            Sign3IntelligencePlugin sign3IntelligencePlugin = new Sign3IntelligencePlugin();
            if (sign3IntelligencePlugin.stop()) return;
            OptionsBuilder.INSTANCE.build(
                    "<SIGN3_CLIENT_ID>",
                    "<SIGN3_CLIENT_SECRET>",
                    OptionsBuilder.ENV_DEV); // For Prod: Options.ENV_PROD, For Dev: Options.ENV_DEV
            sign3IntelligencePlugin.initAsync(this);
        }
    }
}
```

### For Kotlin
```kotlin
import android.os.Build
import com.sign3.intelligence.fraud_sdk_flutter.OptionsBuilder
import com.sign3.intelligence.fraud_sdk_flutter.Sign3IntelligencePlugin
import io.flutter.app.FlutterApplication

class YourApplicationClassName : FlutterApplication() {
    override fun onCreate() {
        super.onCreate()
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            val sign3IntelligencePlugin = Sign3IntelligencePlugin()
            if (sign3IntelligencePlugin.stop()) return
            OptionsBuilder.build(
                clientId = "<SIGN3_CLIENT_ID>",
                secret = "<SIGN3_CLIENT_SECRET>",
                env = OptionsBuilder.ENV_DEV  // For Prod: Options.ENV_PROD, For Dev: Options.ENV_DEV
            )
            sign3IntelligencePlugin.initAsync(this)
        }
    }
}
```

### For AndroidManifest.xml
``` android menifest
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application
        android:name=".YourApplicationClassName"
        android:label="YourAppName"
        android:icon="@mipmap/ic_launcher">
        <!-- Add other components like activities here -->
    </application>

</manifest>
```

## App Permission
   1. Add the following permissions in the Manifest file in the Android folder.
   2. Optional permissions are recommended to achieve higher accuracy.

```java
<uses-permission android:name="android.permission.INTERNET" />
<!-- optional -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<!-- Below mentioned optional permissions are taken to calculate sim information to the device  -->
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
```
<br>

## Get Session ID

1. The Session ID is the unique identifier of a user's app session and serves as a reference point when retrieving the device result for that session.
2. The Session ID follows the OS lifecycle management, in line with industry best practices. This means that a user's session remains active as long as the device maintains it, unless the user terminates the app or the device runs out of memory and has to kill the app.
 
 ```dart
Future<void> getSessionID() async {
  var sessionId = await Sign3Intelligence.getSessionId();
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
UpdateOptions getUpdatedOptions() {
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
      .setPhoneNumber("<phone_number>")
      .setUserId("<user_id>")
      .setPhoneInputType(PhoneInputType.GOOGLE_HINT)
      .setOtpInputType(OtpInputType.AUTO_FILLED)
      .setUserEventType(UserEventType.TRANSACTION)
      .setMerchantId("<merchant_id>")
      .setAdditionalAttributes(additionalAttributes)
      .build();
  return updateOptions;
}

Future<void> updateOptions() async {
  await Sign3Intelligence.updateOptions(getUpdatedOptions());
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

- **requestId**: A unique identifier for the specific request.
- **newDevice**: Indicates if the device is new.
- **deviceId**: A unique identifier for the device.
- **vpn**: Indicates whether a VPN is active on the device.
- **proxy**: Indicates whether a proxy server is in use.
- **emulator**: Indicates if the app is running on an emulator.
- **remoteAppProviders**: Indicates whether any remote applications are installed on the device.
- **mirroredScreen**: Indicates if the device's screen is being mirrored.
- **cloned**: Indicates if the user is using a cloned instance of the app.
- **geoSpoofed**: Indicates if the device's location is being faked.
- **rooted**: Indicates if the device has been modified for root access.
- **sessionRiskScore**: A score representing the risk level of the session.
- **hooking**: Indicates if the app has been altered by malicious code.
- **factoryReset**: Indicates if a suspicious factory reset has been performed.
- **appTampering**: Indicates if the app has been modified in an unauthorized way.


