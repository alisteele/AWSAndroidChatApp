# AWSAndroidChatApp
AWS Chat App

Get Started
Overview
The AWS Mobile Android and iOS SDKs help you build high quality mobile apps quickly and easily. They provide easy access to a range of AWS services, including Amazon Cognito, AWS Lambda, Amazon S3, Amazon Kinesis, Amazon DynamoDB, Amazon Pinpoint and many more.

Set Up Your Backend
Sign up for the AWS Free Tier.

Create a Mobile Hub project by signing into the console. The Mobile Hub console provides a single location for managing and monitoring your app's cloud resources.

To integrate existing AWS resources using the SDK directly, without Mobile Hub, see Setup Options for Android or Setup Options for iOS.

Name your project, check the box to allow Mobile Hub to administer resources for you and then choose Add.

Android - JavaiOS - Swift
Choose Android as your platform and then choose Next.


Choose the Download Cloud Config and then choose Next.

The awsconfiguration.json file you download contains the configuration of backend resources that Mobile Hub enabled in your project. Analytics cloud services are enabled for your app by default.


Add the backend service configuration file to your app.

Right-click your app's res folder, and then choose New > Android Resource Directory. Select raw in the Resource type dropdown menu.


                           Image of selecting a Raw Android Resource Directory in Android Studio.
                        
From the location where configuration file, awsconfiguration.json, was downloaded in a previous step, drag it into the res/raw folder. Android gives a resource ID to any arbitrary file placed in this folder, making it easy to reference in the app.


Remember

Every time you create or update a feature in your Mobile Hub project, download and integrate a new version of your awsconfiguration.json into each app in the project that will use the update.

Your backend is now configured. Follow the next steps at Connect to Your Backend.

Connect to Your Backend
Android - JavaiOS - Swift
Prerequisites

Install Android Studio version 2.33 or higher.

Install Android SDK v7.11 (Nougat), API level 25.

Your AndroidManifest.xml must contain:

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
Add dependencies to your app/build.gradle, then choose Sync Now in the upper right of Android Studio. This libraries enable basic AWS functions, like credentials, and analytics.

dependencies {
    implementation ('com.amazonaws:aws-android-sdk-mobile-client:2.6.+@aar') { transitive = true }
}
Add the following code to the onCreate method of your main or startup activity. AWSMobileClient is a singleton that establishes your connection to and acts as an interface for your services.

import com.amazonaws.mobile.client.AWSMobileClient;

  public class YourMainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        AWSMobileClient.getInstance().initialize(this, new AWSStartupHandler() {
            @Override
            public void onComplete(AWSStartupResult awsStartupResult) {
                Log.d("YourMainActivity", "AWSMobileClient is instantiated and you are connected to AWS!");
            }
        }).execute();

        // More onCreate code ...
    }
  }

What does this do?

When AWSMobileClient is initialized, it constructs the AWSCredentialsProvider and AWSConfiguration objects which, in turn, are used when creating other SDK clients. The client then makes a Sigv4 signed network call to Amazon Cognito Federated Identities to retrieve AWS credentials that provide the user access to your backend resources. When the network interaction succeeds, the onComplete method of the AWSStartUpHandler is called.

Your app is now set up to interact with the AWS services you configured in your Mobile Hub project!

Choose the Run icon in Android Studio to build your app and run it on your device/emulator. Look for Welcome to AWS! in your Android Logcat output (choose View > Tool Windows > Logcat).


Optional: The following example shows how to retrieve the reference to AWSCredentialsProvider and AWSConfiguration objects which can be used to instantiate other SDK clients. You can use the IdentityManager to fetch the user's AWS identity id either directly from Amazon Cognito or from the locally cached identity id value.

import com.amazonaws.auth.AWSCredentialsProvider;
import com.amazonaws.mobile.auth.core.IdentityHandler;
import com.amazonaws.mobile.auth.core.IdentityManager;
import com.amazonaws.mobile.client.AWSMobileClient;
import com.amazonaws.mobile.client.AWSStartupHandler;
import com.amazonaws.mobile.client.AWSStartupResult;
import com.amazonaws.mobile.config.AWSConfiguration;

public class YourMainActivity extends Activity {

    private AWSCredentialsProvider credentialsProvider;
    private AWSConfiguration configuration;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        AWSMobileClient.getInstance().initialize(this, new AWSStartupHandler() {
            @Override
            public void onComplete(AWSStartupResult awsStartupResult) {

                // Obtain the reference to the AWSCredentialsProvider and AWSConfiguration objects
                credentialsProvider = AWSMobileClient.getInstance().getCredentialsProvider();
                configuration = AWSMobileClient.getInstance().getConfiguration();

                // Use IdentityManager#getUserID to fetch the identity id.
                IdentityManager.getDefaultIdentityManager().getUserID(new IdentityHandler() {
                    @Override
                    public void onIdentityId(String identityId) {
                        Log.d("YourMainActivity", "Identity ID = " + identityId);

                        // Use IdentityManager#getCachedUserID to
                        //  fetch the locally cached identity id.
                        final String cachedIdentityId =
                            IdentityManager.getDefaultIdentityManager().getCachedUserID();
                    }

                    @Override
                    public void handleError(Exception exception) {
                        Log.d("YourMainActivity", "Error in retrieving the identity" + exception);
                    }
                });
            }
        }).execute();

        // .. more code
    }
}
Connect to your Backend
Use the following steps to add analytics to your mobile app through AWS Pinpoint.

Android - JavaiOS - Swift
Set up AWS Mobile SDK components with the following Basic Backend Setup steps.

app/build.gradle must contain:

dependencies{
   implementation 'com.amazonaws:aws-android-sdk-pinpoint:2.6.+'
}
Instrument your app to provide basic session data for Amazon Pinpoint analytics. The Amazon Pinpoint SDK gives you full control of when your sessions are started and stopped. Your app must explicitly start and stop the sessions. The following example shows one way to handle this by instrumenting a public class that extends MultidexApplication. StartSession() is called during the OnCreate event.

Add the following to app/build.gradle:

  android {
      defaultConfig {
          ...
          multiDexEnabled = true
      }
  }
Add the following to the dependencies section of app/build.gradle:

  implementation 'com.android.support:multidex:1.0.+'
Add the following to AndroidManifest.xml:

  <application
  ..
  android:theme="@style/AppTheme"
  android:name="com.YourApplication.Application">
  ..
  </application>
Add the following to your activity:

  //. . .
  import com.amazonaws.mobileconnectors.pinpoint.PinpointManager;
  import com.amazonaws.mobileconnectors.pinpoint.PinpointConfiguration;
  //. . .

  public class MainActivity extends AppCompatActivity {

     public static PinpointManager pinpointManager;

      @Override
      public void onCreate() {

          super.onCreate();

          PinpointConfiguration pinpointConfig = new PinpointConfiguration(
                  getApplicationContext(),
                  AWSMobileClient.getInstance().getCredentialsProvider(),
                  AWSMobileClient.getInstance().getConfiguration());

          pinpointManager = new PinpointManager(pinpointConfig);

          // Start a session with Pinpoint
          pinpointManager.getSessionClient().startSession();

          // Stop the session and submit the default app started event
          pinpointManager.getSessionClient().stopSession();
          pinpointManager.getAnalyticsClient().submitEvents();
      }

  }
Build and run your app to see usage metrics in Amazon Pinpoint.

To see visualizations of the analytics coming from your app, open your project in the Mobile Hub console.

Choose Analytics on the upper right to open the Amazon Pinpoint console.


            |AMH| console link to your project in the Amazon Pinpoint console.
         
Choose Analytics from the icons on the left of the console, and view the graphs of your app's usage. It may take up to 15 minutes for metrics to become visible.


Learn more about Amazon Pinpoint.

Enable Custom App Analytics
Instrument your code to capture app usage event information, including attributes you define. Use graphs of your custom usage event data in the Amazon Pinpoint console. Visualize how your users' behavior aligns with a model you design using Amazon Pinpoint Funnel Analytics, or use stream the data for deeper analysis.

Use the following steps to implement Amazon Pinpoint custom analytics for your app.

Android - JavaiOS - Swift
    import com.amazonaws.mobileconnectors.pinpoint.analytics.AnalyticsEvent;

    public void logEvent() {
        pinpointManager.getSessionClient().startSession();
        final AnalyticsEvent event =
            pinpointManager.getAnalyticsClient().createEvent("EventName")
                .withAttribute("DemoAttribute1", "DemoAttributeValue1")
                .withAttribute("DemoAttribute2", "DemoAttributeValue2")
                .withMetric("DemoMetric1", Math.random());

        pinpointManager.getAnalyticsClient().recordEvent(event);
        pinpointManager.getSessionClient().stopSession();
        pinpointManager.getAnalyticsClient().submitEvents();
    }
Build, run, and try your app, and then view your custom events in the Events tab of the Amazon Pinpoint console (use your Mobile Hub project / Analytics > Amazon Pinpoint console / Analytics > Events). Look for the name of your event in the Events dropdown menu.

Enable Revenue Analytics
Amazon Pinpoint supports the collection of monetization event data. Use the following steps to place and design analytics related to purchases through your app.

Android - JavaiOS - Swift
import com.amazonaws.mobileconnectors.pinpoint.analytics.monetization.AmazonMonetizationEventBuilder;

public void logMonetizationEvent() {
    pinpointManager.getSessionClient().startSession();

    final AnalyticsEvent event =
        AmazonMonetizationEventBuilder.create(pinpointManager.getAnalyticsClient())
            .withFormattedItemPrice("$10.00")
            .withProductId("DEMO_PRODUCT_ID")
            .withQuantity(1.0)
            .withProductId("DEMO_TRANSACTION_ID").build();

    pinpointManager.getAnalyticsClient().recordEvent(event);
    pinpointManager.getSessionClient().stopSession();
    pinpointManager.getAnalyticsClient().submitEvents();
}

Add User Sign-in to Your Mobile App with Amazon Cognito
Enable your users to sign-in using credentials from Facebook, Google, or your own custom user directory. The AWS Mobile Hub User Sign-in feature is powered by Amazon Cognito, and the SDK provides a pre-built, configurable Sign-in UI based on the identity provider(s) you configure.

Set Up Your Backend
Prerequisite Complete the Get Started steps before your proceed.

Email & PasswordFacebookGoogle
Enable User Sign-in: Open your project in Mobile Hub console and choose the User Sign-in tile.

Configure Email and Password sign-in: Choose the feature and then select sign-in policies including: allowed login methods; multi-factor authentication; and password requirements and then choose Create user pool.


When you are done configuring providers, choose Click here to return to project details page in the blue banner at the top.


On the project detail page, choose the flashing Integrate button, and then complete the steps that guide you to connect your backend.

If your project contains apps for more than one platform, any that need to complete those steps will also display a flashing Integrate button. The reminder banner will remain in place until you have taken steps to update the configuration of each app in the project.


Follow the Set up Email & Password Login steps to connect to your backend from your app.

Setup Email & Password Login in your Mobile App
Choose your platform:

Android - JavaiOS - Swift

Use Android API level 23 or higher

The AWS Mobile SDK library for Android sign-in (aws-android-sdk-auth-ui) provides the activity and view for presenting a SignInUI for the sign-in providers you configure. This library depends on the Android SDK API Level 23 or higher.

Add or update your AWS backend configuration file to incorporate your new sign-in. For details, see the last steps in the Get Started: Set Up Your Backend section.

Add these permisions to the AndroidManifest.xml file:

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
Add these dependencies to the app/build.gradle file:

dependencies {
     // Mobile Client for initializing the SDK
     implementation ('com.amazonaws:aws-android-sdk-mobile-client:2.6.+@aar') { transitive = true }

     // Cognito UserPools for SignIn
     implementation 'com.android.support:support-v4:24.+'
     implementation ('com.amazonaws:aws-android-sdk-auth-userpools:2.6.+@aar') { transitive = true }

     // Sign in UI Library
     implementation 'com.android.support:appcompat-v7:24.+'
     implementation ('com.amazonaws:aws-android-sdk-auth-ui:2.6.+@aar') { transitive = true }
}
Create an activity that will present your sign-in screen.

In Android Studio, choose File > New > Activity > Basic Activity and type an activity name, such as AuthenticatorActivity. If you want to make this your starting activity, move the the intent filter block containing .LAUNCHER to the AuthenticatorActivity in your app's AndroidManifest.xml.

<activity android:name=".AuthenticatorActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
Update the onCreate function of your AuthenticatorActivity to call AWSMobileClient. This component provides the functionality to resume a signed-in authentication session. It makes a network call to retrieve the AWS credentials that allow users to access your AWS resources and registers a callback for when that transaction completes.

If the user is signed in, the app goes to the NextActivity, otherwise it presents the user with the AWS Mobile ready made, configurable sign-in UI. NextActivity Activity class a user sees after a successful sign-in.

import android.app.Activity;
import android.os.Bundle;

import com.amazonaws.mobile.auth.ui.SignInUI;
import com.amazonaws.mobile.client.AWSMobileClient;
import com.amazonaws.mobile.client.AWSStartupHandler;
import com.amazonaws.mobile.client.AWSStartupResult;

public class AuthenticatorActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_authenticator);

        // Add a call to initialize AWSMobileClient
        AWSMobileClient.getInstance().initialize(this, new AWSStartupHandler() {
            @Override
            public void onComplete(AWSStartupResult awsStartupResult) {
                SignInUI signin = (SignInUI) AWSMobileClient.getInstance().getClient(AuthenticatorActivity.this, SignInUI.class);
                signin.login(AuthenticatorActivity.this, NextActivity.class).execute();
            }
        }).execute();
    }
}
Choose the Run icon in Android Studio to build your app and run it on your device/emulator. You should see our ready made sign-in UI for your app. Checkout the next steps to learn how to customize your UI.


API References

AWSMobileClient

A library that initializes the SDK, constructs CredentialsProvider and AWSConfiguration objects, fetches the AWS credentials, and creates a SDK SignInUI client instance.

Auth UserPools

A wrapper Library for Amazon Cognito UserPools that provides a managed Email/Password sign-in UI.

Auth Core

A library that caches and federates a login provider authentication token using Amazon Cognito Federated Identities, caches the federated AWS credentials, and handles the sign-in flow.

Setup Facebook Login in your Mobile App
Android - JavaiOS - Swift

Use Android API level 23 or higher

The AWS Mobile SDK library for Android sign-in (aws-android-sdk-auth-ui) provides the activity and view for presenting a SignInUI for the sign-in providers you configure. This library depends on the Android SDK API Level 23 or higher.

Add or update your AWS backend configuration file to incorporate your new sign-in. For details, see the last steps in the Get Started: Set Up Your Backend section.

Add the following permissions and Activity to your AndroidManifest.xml file:

<!-- ... -->

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

<!-- ... -->

<activity
    android:name="com.facebook.FacebookActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="@string/fb_login_protocol_scheme" />
    </intent-filter>
</activity>

<!-- ... -->

<meta-data android:name="com.facebook.sdk.ApplicationId" android:value="@string/facebook_app_id" />

<!-- ... -->
Add these dependencies to your app/build.gradle file:

dependencies {
  // Mobile Client for initializing the SDK
  implementation ('com.amazonaws:aws-android-sdk-mobile-client:2.6.+@aar') { transitive = true }

  // Facebook SignIn
  implementation 'com.android.support:support-v4:24.+'
  implementation ('com.amazonaws:aws-android-sdk-auth-facebook:2.6.+@aar') { transitive = true }

  // Sign in UI
  implementation 'com.android.support:appcompat-v7:24.+'
  implementation ('com.amazonaws:aws-android-sdk-auth-ui:2.6.+@aar') { transitive = true }
}
In strings.xml, add string definitions for your Facebook App ID and login protocol scheme.The value should contain your Facebook AppID in both cases, the login protocol value is always prefaced with fb.

<string name="facebook_app_id">1231231231232123123</string>
<string name="fb_login_protocol_scheme">fb1231231231232123123</string>
Create an activity that will present your sign-in screen.

In Android Studio, choose File > New > Activity > Basic Activity and type an activity name, such as AuthenticatorActivity. If you want to make this your starting activity, move the the intent filter block containing .LAUNCHER to the AuthenticatorActivity in your app's AndroidManifest.xml.

<activity android:name=".AuthenticatorActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
Update the onCreate function of your AuthenticatorActivity to call AWSMobileClient. This component provides the functionality to resume a signed-in authentication session. It makes a network call to retrieve the AWS credentials that allow users to access your AWS resources and registers a callback for when that transaction completes.

If the user is signed in, the app goes to the NextActivity, otherwise it presents the user with the AWS Mobile ready made, configurable sign-in UI. NextActivity Activity class a user sees after a successful sign-in.

import android.app.Activity;
import android.os.Bundle;

import com.amazonaws.mobile.auth.ui.SignInUI;
import com.amazonaws.mobile.client.AWSMobileClient;
import com.amazonaws.mobile.client.AWSStartupHandler;
import com.amazonaws.mobile.client.AWSStartupResult;

public class AuthenticatorActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_authenticator);

        // Add a call to initialize AWSMobileClient
        AWSMobileClient.getInstance().initialize(this, new AWSStartupHandler() {
            @Override
            public void onComplete(AWSStartupResult awsStartupResult) {
                SignInUI signin = (SignInUI) AWSMobileClient.getInstance().getClient(AuthenticatorActivity.this, SignInUI.class);
                signin.login(AuthenticatorActivity.this, NextActivity.class).execute();
            }
        }).execute();
    }
}
Choose the Run icon in Android Studio to build your app and run it on your device/emulator. You should see our ready made sign-in UI for your app. Checkout the next steps to learn how to customize your UI.


API References

AWSMobileClient

A library that initializes the SDK, constructs CredentialsProvider and AWSConfiguration objects, fetches the AWS credentials, and creates a SDK SignInUI client instance.

Auth UserPools

A wrapper Library for Amazon Cognito UserPools that provides a managed Email/Password sign-in UI.

Auth Core

A library that caches and federates a login provider authentication token using Amazon Cognito Federated Identities, caches the federated AWS credentials, and handles the sign-in flow.

Setup Google Login in your Mobile App
Android - JavaiOS - Swift

Use Android API level 23 or higher

The AWS Mobile SDK library for Android sign-in (aws-android-sdk-auth-ui) provides the activity and view for presenting a SignInUI for the sign-in providers you configure. This library depends on the Android SDK API Level 23 or higher.

Add or update your AWS backend configuration file to incorporate your new sign-in. For details, see the last steps in the Get Started: Set Up Your Backend section.

Add these permissions to your AndroidManifest.xml file:

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
Add these dependencies to your app/build.gradle file:

dependencies {
    // Mobile Client for initializing the SDK
    implementation ('com.amazonaws:aws-android-sdk-mobile-client:2.6.+@aar') { transitive = true }

    // Google SignIn
    implementation 'com.android.support:support-v4:24.+'
    implementation ('com.amazonaws:aws-android-sdk-auth-google:2.6.+@aar') { transitive = true }

    // Sign in UI Library
    implementation 'com.android.support:appcompat-v7:24.+'
    implementation ('com.amazonaws:aws-android-sdk-auth-ui:2.6.+@aar') { transitive = true }
}
Create an activity that will present your sign-in screen.

In Android Studio, choose File > New > Activity > Basic Activity and type an activity name, such as AuthenticatorActivity. If you want to make this your starting activity, move the the intent filter block containing .LAUNCHER to the AuthenticatorActivity in your app's AndroidManifest.xml.

<activity android:name=".AuthenticatorActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
Update the onCreate function of your AuthenticatorActivity to call AWSMobileClient. This component provides the functionality to resume a signed-in authentication session. It makes a network call to retrieve the AWS credentials that allow users to access your AWS resources and registers a callback for when that transaction completes.

If the user is signed in, the app goes to the NextActivity, otherwise it presents the user with the AWS Mobile ready made, configurable sign-in UI. NextActivity Activity class a user sees after a successful sign-in.

import android.app.Activity;
import android.os.Bundle;

import com.amazonaws.mobile.auth.ui.SignInUI;
import com.amazonaws.mobile.client.AWSMobileClient;
import com.amazonaws.mobile.client.AWSStartupHandler;
import com.amazonaws.mobile.client.AWSStartupResult;

public class AuthenticatorActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_authenticator);

        // Add a call to initialize AWSMobileClient
        AWSMobileClient.getInstance().initialize(this, new AWSStartupHandler() {
            @Override
            public void onComplete(AWSStartupResult awsStartupResult) {
                SignInUI signin = (SignInUI) AWSMobileClient.getInstance().getClient(AuthenticatorActivity.this, SignInUI.class);
                signin.login(AuthenticatorActivity.this, MainActivity.class).execute();
            }
        }).execute();
    }
}
Choose the Run icon in Android Studio to build your app and run it on your device/emulator. You should see our ready made sign-in UI for your app. Checkout the next steps to learn how to customize your UI.


API References

AWSMobileClient

A library that initializes the SDK, constructs CredentialsProvider and AWSConfiguration objects, fetches the AWS credentials, and creates a SDK SignInUI client instance.

Auth UserPools

A wrapper Library for Amazon Cognito UserPools that provides a managed Email/Password sign-in UI.

Auth Core

A library that caches and federates a login provider authentication token using Amazon Cognito Federated Identities, caches the federated AWS credentials, and handles the sign-in flow.

Enable Sign-out
Android - JavaiOS - Swift
To enable a user to sign-out of your app, register a callback for sign-in events by adding a SignInStateChangeListener to IdentityManager. The listener captures both onUserSignedIn and onUserSignedOut events.

IdentityManager.getDefaultIdentityManager().addSignInStateChangeListener(new SignInStateChangeListener() {
    @Override
    // Sign-in listener
    public void onUserSignedIn() {
        Log.d(LOG_TAG, "User Signed In");
    }

    // Sign-out listener
    @Override
    public void onUserSignedOut() {

        // return to the sign-in screen upon sign-out
       showSignIn();
    }
});
To initiate a sign-out, call the signOut method of IdentityManager.

IdentityManager.getDefaultIdentityManager().signOut();
For a fuller example, see Sign-out a Signed-in User in the How To section.

Customize Your Sign-In UI
By default, the SDK presents sign-in UI for each sign in provider you enable in your Mobile Hub project (Email and Password, Facebook, Google) with a default look and feel. It knows which provider(s) you chose by reading the awsconfiguration.json file you downloaded.

To override the defaults, and modify the behavior, look, and feel of the sign-in UI, create an AuthUIConfiguration object and set the appropriate properties.

Android - JavaiOS - Swift
Create and configure an AuthUIConfiguration object and set its properties.

To present the Email and Password user SignInUI, set userPools to true.

To present Facebook or Google user SignInUI, add signInButton(FacebookButton.class) or signInButton(GoogleButton.class).

To change the logo, use the logoResId.

To change the background color, use backgroundColor.

To cancel the sign-in flow, set .canCancel(true).

To change the font in the sign-in views, use the fontFamily method and pass in the string that represents a font family.

To draw the backgroundColor full screen, use fullScreenBackgroundColor.

import android.app.Activity;
import android.graphics.Color;
import android.os.Bundle;

import com.amazonaws.mobile.auth.facebook.FacebookButton;
import com.amazonaws.mobile.auth.google.GoogleButton;
import com.amazonaws.mobile.auth.ui.AuthUIConfiguration;
import com.amazonaws.mobile.auth.ui.SignInUI;

import com.amazonaws.mobile.client.AWSMobileClient;
import com.amazonaws.mobile.client.AWSStartupHandler;
import com.amazonaws.mobile.client.AWSStartupResult;

public class YourMainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        AWSMobileClient.getInstance().initialize(this, new AWSStartupHandler() {
            @Override
            public void onComplete(final AWSStartupResult awsStartupResult) {
                AuthUIConfiguration config =
                    new AuthUIConfiguration.Builder()
                        .userPools(true)  // true? show the Email and Password UI
                        .signInButton(FacebookButton.class) // Show Facebook button
                        .signInButton(GoogleButton.class) // Show Google button
                        .logoResId(R.drawable.mylogo) // Change the logo
                        .backgroundColor(Color.BLUE) // Change the backgroundColor
                        .isBackgroundColorFullScreen(true) // Full screen backgroundColor the backgroundColor full screenff
                        .fontFamily("sans-serif-light") // Apply sans-serif-light as the global font
                        .canCancel(true)
                        .build();
                SignInUI signinUI = (SignInUI) AWSMobileClient.getInstance().getClient(YourMainActivity.this, SignInUI.class);
                signinUI.login(YourMainActivity.this, YourNextActivity.class).authUIConfiguration(config).execute();
            }
        }).execute();
    }
}

How to Integrate Your Existing Bucket

Just Getting Started?

Use streamlined steps to install the SDK and integrate Amazon S3.

Or, use the contents of this page if your app will integrate existing AWS services.

The following steps include:

Set up short-lived credentials for accessing your AWS resources using a Cognito Identity Pool.

Create an AWS Mobile configuration file that ties your app code to your bucket.

To configure a new Amazon S3 bucket, see .

Set Up Your Backend
If you already have a Cognito Identity Pool and have its unauthenticated IAM role set up with read/write permissions on the S3 bucket, you can skip to Get Your Bucket Name and ID.

Create or Import the Amazon Cognito Identity Pool
Go to Amazon Cognito Console and choose Manage Federated Identities.

Choose Create new Identity pool on the top left of the console.

Type a name for the Identity pool, select Enable access to unauthenticated identities under the Unauthenticated Identities section, and then choose Create pool on the bottom right.

Expand the View Details section to see the two roles that are to be created to enable access to your bucket. Copy and keep the Unauthenticated role name, in the form of Cognito_<IdentityPoolName>Unauth_Role, for use in a following configuration step. Choose Allow on the bottom right.

In the code snippet labeled Get AWSCredentials displayed by the console, copy the Identity Pool ID and the Region for use in a following configuration step.

Set up the required Amazon IAM permissions
Go to Amazon IAM Console and choose Roles.

Choose the unauthenticated role whose name you copied in a previous step.

Choose Attach Policy, select the AmazonS3FullAccess policy, and then choose Attach Policy to attach it to the role.


Note

The AmazonS3FullAccess policy will grant users in the identity pool full access to all buckets and operations in Amazon S3. In a real app, you should restrict users to only have access to the specific resources they need. For more information, see Amazon S3 Security Considerations.

Get Your Bucket Name and ID
Go to Amazon S3 Console and select the bucket you want to integrate.

Copy and keep the bucket name value from the breadcrumb at the top of the console, for use in a following step.

Copy and keep the bucket's region, for use in a following step.

Connect to Your Backend
Create the awsconfiguration.json file
Create a file with name awsconfiguration.json with the following contents:

{
    "Version": "1.0",
    "CredentialsProvider": {
        "CognitoIdentity": {
            "Default": {
                "PoolId": "COGNITO-IDENTITY-POOL-ID",
                "Region": "COGNITO-IDENTITY-POOL-REGION"
            }
        }
    },
    "IdentityManager" : {
      "Default" : {

      }
    },
    "S3TransferUtility": {
        "Default": {
            "Bucket": "S3-BUCKET-NAME",
            "Region": "S3-REGION"
        }
    }
}
Make the following changes to the configuration file.

Replace the COGNITO-IDENTITY-POOL-ID with the identity pool ID.

Replace the COGNITO-IDENTITY-POOL-REGION with the region the identity pool was created in.

Replace the S3-BUCKET-NAME with the name of your bucket.

Replace the S3-REGION with the region your bucket was created in.

Add the awsconfiguration.json file to your app
Android - JavaiOS - SwiftiOS - Swift
Right-click your app's res folder, and then choose New > Android Resource Directory. Select raw in the Resource type dropdown menu.


                        Image of selecting a Raw Android Resource Directory in Android Studio.
                     
Drag the awsconfiguration.json you created into the res/raw folder. Android gives a resource ID to any arbitrary file placed in this folder, making it easy to reference in the app.

Add the SDK to your App
Android - JavaiOS - Swift
Set up AWS Mobile SDK components as follows:

Add the following to app/build.gradle:

dependencies {
   implementation ('com.amazonaws:aws-android-sdk-mobile-client:2.6.+@aar') { transitive = true }
   implementation 'com.amazonaws:aws-android-sdk-s3:2.6.+'
   implementation 'com.amazonaws:aws-android-sdk-cognito:2.6.+'
}
Perform a Gradle Sync to download the AWS Mobile SDK components into your app

Add the following to AndroidManifest.xml:

<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

<application ... >

   <!- . . . ->

   <service android:name="com.amazonaws.mobileconnectors.s3.transferutility.TransferService" android:enabled="true" />

   <!- . . . ->

</application>
For each Activity where you make calls to perform user file storage operations, import the following packages.

import com.amazonaws.mobile.config.AWSConfiguration;
import com.amazonaws.mobileconnectors.s3.transferutility.*;
Add the following code to the onCreate method of your main or startup activity. This will establish a connection with AWS Mobile. AWSMobileClient is a singleton that will be an interface for your AWS services.

import com.amazonaws.mobile.client.AWSMobileClient;

public class YourMainActivity extends Activity {
 @Override
 protected void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);

     AWSMobileClient.getInstance().initialize(this).execute();
  }
}
Implement Storage Operations
Once your backend is setup and connected to your app, use the following steps to upload and download a file using the SDK's transfer utility.

Upload a File
Android - JavaiOS - Swift
To upload a file to an Amazon S3 bucket, use AWSMobileClient to get the AWSConfiguration and AWSCredentialsProvider, then create the TransferUtility object. AWSMobileClient expects an activity context for resuming an authenticated session and creating the credentials provider.

The following example shows using the TransferUtility `in the context of an Activity. If you are creating :code:`TransferUtility from an application context, you can construct the AWSCredentialsProvider and pass it into TransferUtility to use in forming the AWSConfiguration object.. The TransferUtility will check the size of file being uploaded and will automatically switch over to using multi-part uploads if the file size exceeds 5 MB.

import android.app.Activity;
import android.util.Log;

import com.amazonaws.mobile.client.AWSMobileClient;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferUtility;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferState;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferObserver;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferListener;
import com.amazonaws.services.s3.AmazonS3Client;

import java.io.File;

public class YourActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        AWSMobileClient.getInstance().initialize(this).execute();
        uploadWithTransferUtility();
    }

    private void uploadWithTransferUtility() {

        TransferUtility transferUtility =
            TransferUtility.builder()
                .context(getApplicationContext())
                .awsConfiguration(AWSMobileClient.getInstance().getConfiguration())
                .s3Client(new AmazonS3Client(AWSMobileClient.getInstance().getCredentialsProvider()))
                .build();

        TransferObserver uploadObserver =
            transferUtility.upload(
                "s3Folder/s3Key.txt",
                new File("/path/to/file/localFile.txt"));

        // Attach a listener to the observer to get state update and progress notifications
        uploadObserver.setTransferListener(new TransferListener() {

            @Override
            public void onStateChanged(int id, TransferState state) {
                if (TransferState.COMPLETED == state) {
                    // Handle a completed upload.
                }
            }

            @Override
            public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
                float percentDonef = ((float) bytesCurrent / (float) bytesTotal) * 100;
                int percentDone = (int)percentDonef;

                Log.d("YourActivity", "ID:" + id + " bytesCurrent: " + bytesCurrent
                        + " bytesTotal: " + bytesTotal + " " + percentDone + "%");
            }

            @Override
            public void onError(int id, Exception ex) {
                // Handle errors
            }

        });

        // If you prefer to poll for the data, instead of attaching a
        // listener, check for the state and progress in the observer.
        if (TransferState.COMPLETED == uploadObserver.getState()) {
            // Handle a completed upload.
        }

        Log.d("YourActivity", "Bytes Transferrred: " + uploadObserver.getBytesTransferred());
        Log.d("YourActivity", "Bytes Total: " + uploadObserver.getBytesTotal());
    }
}
Download a File
Android - JavaiOS - Swift
To download a file from an Amazon S3 bucket, use AWSMobileClient to get the AWSConfigurationand AWSCredentialsProvider to create the TransferUtility object. AWSMobileClient expects an activity context for resuming an authenticated session and creating the .

The following example shows using the TransferUtility in the context of an Activity. If you are creating TransferUtility from an application context, you can construct the AWSCredentialsProvider and pass it into TransferUtility to use in forming the AWSConfiguration object.

import android.app.Activity;
import android.util.Log;

import com.amazonaws.mobile.client.AWSMobileClient;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferUtility;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferState;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferObserver;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferListener;
import com.amazonaws.services.s3.AmazonS3Client;

import java.io.File;

public class YourActivity extends Activity {

    public void dowloadData() {
        AWSMobileClient.getInstance().initialize(this, new AWSStartupHandler() {
            @Override
            public void onComplete() {
                downloadWithTransferUtility();
            }
        }).execute();
    }

    public void downloadWithTransferUtility() {

        TransferUtility transferUtility =
            TransferUtility.builder()
                    .context(getApplicationContext())
                    .awsConfiguration(AWSMobileClient.getInstance().getConfiguration())
                    .s3Client(new AmazonS3Client(AWSMobileClient.getInstance().getCredentialsProvider()))
                    .build();

        TransferObserver downloadObserver =
            transferUtility.download(
                    "s3Folder/s3Key.txt",
                    new File("/path/to/file/localFile.txt"));

        // Attach a listener to the observer to get state update and progress notifications
        downloadObserver.setTransferListener(new TransferListener() {

            @Override
            public void onStateChanged(int id, TransferState state) {
                if (TransferState.COMPLETED == state) {
                    // Handle a completed upload.
                }
            }

            @Override
            public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
                    float percentDonef = ((float)bytesCurrent/(float)bytesTotal) * 100;
                    int percentDone = (int)percentDonef;

                    Log.d("MainActivity", "   ID:" + id + "   bytesCurrent: " + bytesCurrent + "   bytesTotal: " + bytesTotal + " " + percentDone + "%");
            }

            @Override
            public void onError(int id, Exception ex) {
                // Handle errors
            }

        });

        // If you prefer to poll for the data, instead of attaching a
        // listener, check for the state and progress in the observer.
        if (TransferState.COMPLETED == downloadObserver.getState()) {
            // Handle a completed upload.
        }

        Log.d("YourActivity", "Bytes Transferrred: " + downloadObserver.getBytesTransferred());
        Log.d("YourActivity", "Bytes Total: " + downloadObserver.getBytesTotal());
    }
}
Next Steps
For further information about TransferUtility capabilities, see Transfer Files and Data Using TransferUtility and Amazon S3.

For sample apps that demonstrate TransferUtility capabilities, see Android S3 TransferUtility Sample and iOS S3 TransferUtility Sample.
Transfer Files and Data Using TransferUtility and Amazon S3

Just Getting Started?

Use streamlined steps to install the SDK and integrate Amazon S3.

Or, use the contents of this page if your app will integrate existing AWS services.

This page explains how to implement upload and download functionality and a number of additional storage use cases.

The examples on this page assume you have added the the AWS Mobile SDK to your mobile app. To create a new cloud storage backend for your app, see Add User File Storage.


Best practice

If you use the transfer utility multipart upload feature, take advantage of automatic cleanup features by seting up the AbortIncompleteMultipartUpload action in your Amazon S3 bucket life cycle configuration.

Upload a File
Android - JavaiOS - Swift
The following example shows how to upload a file to an Amazon S3 bucket.

Use AWSMobileClient to get the AWSConfiguration and AWSCredentialsProvider, then create the TransferUtility object. AWSMobileClient expects an activity context for resuming an authenticated session and creating the credentials provider.

The following example shows using the transfer utility in the context of an Activity. If you are creating transfer utility from an application context, you can construct the CredentialsProvider and AWSConfiguration object and pass it into TransferUtility. The TransferUtility will check the size of file being uploaded and will automatically switch over to using multi part uploads if the file size exceeds 5 MB.

import android.app.Activity;
import android.util.Log;

import com.amazonaws.mobile.client.AWSMobileClient;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferUtility;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferState;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferObserver;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferListener;
import com.amazonaws.services.s3.AmazonS3Client;

import java.io.File;

public class YourActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        AWSMobileClient.getInstance().initialize(this).execute();
        uploadWithTransferUtility();
    }

    private void uploadWithTransferUtility() {

        TransferUtility transferUtility =
            TransferUtility.builder()
                .context(getApplicationContext())
                .awsConfiguration(AWSMobileClient.getInstance().getConfiguration())
                .s3Client(new AmazonS3Client(AWSMobileClient.getInstance().getCredentialsProvider()))
                .build();

        TransferObserver uploadObserver =
            transferUtility.upload(
                "s3Folder/s3Key.txt",
                new File("/path/to/file/localFile.txt"));

        // Attach a listener to the observer to get state update and progress notifications
        uploadObserver.setTransferListener(new TransferListener() {

            @Override
            public void onStateChanged(int id, TransferState state) {
                if (TransferState.COMPLETED == state) {
                    // Handle a completed upload.
                }
            }

            @Override
            public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
                float percentDonef = ((float) bytesCurrent / (float) bytesTotal) * 100;
                int percentDone = (int)percentDonef;

                Log.d("YourActivity", "ID:" + id + " bytesCurrent: " + bytesCurrent
                        + " bytesTotal: " + bytesTotal + " " + percentDone + "%");
            }

            @Override
            public void onError(int id, Exception ex) {
                // Handle errors
            }

        });

        // If you prefer to poll for the data, instead of attaching a
        // listener, check for the state and progress in the observer.
        if (TransferState.COMPLETED == uploadObserver.getState()) {
            // Handle a completed upload.
        }

        Log.d("YourActivity", "Bytes Transferrred: " + uploadObserver.getBytesTransferred());
        Log.d("YourActivity", "Bytes Total: " + uploadObserver.getBytesTotal());
    }
}
Download a File
Android - JavaiOS - Swift
The following example shows how to download a file from an Amazon S3 bucket. We use AWSMobileClient to get the AWSConfiguration and AWSCredentialsProvider to create the TransferUtility object. AWSMobileClient expects an activity context for resuming an authenticated session and creating the credentials provider.

This example shows using the transfer utility in the context of an Activity. If you are creating transfer utility from an application context, you can construct the CredentialsProvider and AWSConfiguration object and pass it into TransferUtility.

import android.app.Activity;
import android.util.Log;

import com.amazonaws.mobile.client.AWSMobileClient;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferUtility;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferState;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferObserver;
import com.amazonaws.mobileconnectors.s3.transferutility.TransferListener;
import com.amazonaws.services.s3.AmazonS3Client;

import java.io.File;

public class YourActivity extends Activity {

    @Override
      protected void onCreate(Bundle savedInstanceState) {
          AWSMobileClient.getInstance().initialize(this).execute();
          downloadWithTransferUtility();
    }

    public void downloadWithTransferUtility() {

        TransferUtility transferUtility =
              TransferUtility.builder()
                    .context(getApplicationContext())
                    .awsConfiguration(AWSMobileClient.getInstance().getConfiguration())
                    .s3Client(new AmazonS3Client(AWSMobileClient.getInstance().getCredentialsProvider()))
                    .build();

        TransferObserver downloadObserver =
              transferUtility.download(
                    "s3Folder/s3Key.txt",
                    new File("/path/to/file/localFile.txt"));

        // Attach a listener to the observer to get notified of the
        // updates in the state and the progress
        downloadObserver.setTransferListener(new TransferListener() {

           @Override
           public void onStateChanged(int id, TransferState state) {
              if (TransferState.COMPLETED == state) {
                 // Handle a completed upload.
              }
           }

           @Override
           public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
                 float percentDonef = ((float)bytesCurrent/(float)bytesTotal) * 100;
                 int percentDone = (int)percentDonef;

                 Log.d("MainActivity", "   ID:" + id + "   bytesCurrent: " + bytesCurrent + "   bytesTotal: " + bytesTotal + " " + percentDone + "%");
           }

           @Override
           public void onError(int id, Exception ex) {
              // Handle errors
           }

        });

        // If you do not want to attach a listener and poll for the data
        // from the observer, you can check for the state and the progress
        // in the observer.
        if (TransferState.COMPLETED == downloadObserver.getState()) {
            // Handle a completed upload.
        }

        Log.d("YourActivity", "Bytes Transferrred: " + downloadObserver.getBytesTransferred());
        Log.d("YourActivity", "Bytes Total: " + downloadObserver.getBytesTotal());
    }
}
Track Transfer Progress
Android - JavaiOS - Swift
With the TransferUtility, the download() and upload() methods return a TransferObserver object. This object gives access to:

The state, as an enum

The total bytes currently transferred

The total bytes remaining to transfer, to aid in calculating progress bars

A unique ID that you can use to keep track of distinct transfers

Given the transfer ID, the TransferObserver object can be retrieved from anywhere in your app, even if the app was terminated during a transfer. It also lets you create a TransferListener, which will be updated on state or progress change, as well as when an error occurs.

To get the progress of a transfer, call setTransferListener() on your TransferObserver. This requires you to implement onStateChanged, onProgressChanged, and onError. For example:

You can also query for TransferObservers with either the getTransfersWithType(transferType) or getTransfersWithTypeAndState(transferType, transferState) method. You can use TransferObservers to determine what transfers are underway, what are paused and handle the transfers as necessary.

TransferObserver transferObserver = download(MY_BUCKET, OBJECT_KEY, MY_FILE);
transferObserver.setTransferListener(new TransferListener(){

    @Override
    public void onStateChanged(int id, TransferState state) {
        // do something
    }

    @Override
    public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
        int percentage = (int) (bytesCurrent/bytesTotal * 100);
        //Display percentage transfered to user
    }

    @Override
    public void onError(int id, Exception ex) {
        // do something
    }
});
The transfer ID can be retrieved from the TransferObserver object that is returned from upload or download function.

// Gets id of the transfer.
int transferId = transferObserver.getId();
Pause a Transfer
Android - JavaiOS - Swift
Transfers can be paused using the pause(transferId) method. If your app is terminated, crashes, or loses Internet connectivity, transfers are automatically paused.

The transferId can be retrieved from the TransferObserver object as described in Track Transfer Progress.

To pause a single transfer:

transferUtility.pause(idOfTransferToBePaused);
To pause all uploads:

transferUtility.pauseAllWithType(TransferType.UPLOAD);
To pause all downloads:

transferUtility.pauseAllWithType(TransferType.DOWNLOAD);
To pause all transfers of any type:

transferUtility.pauseAllWithType(TransferType.ANY);
Resume a Transfer
Android - JavaiOS - Swift
In the case of a loss in network connectivity, transfers will automatically resume when network connectivity is restored. If the app crashed or was terminated by the operating system, transfers can be resumed with the resume(transferId) method.

The transferId can be retrieved from the TransferObserver object as described in Track Transfer Progress.

To resume a single transfer:

transferUtility.resume(idOfTransferToBeResumed);
To resume all uploads:

transferUtility.resumeAllWithType(TransferType.UPLOAD);
To resume all downloads:

transferUtility.resumeAllWithType(TransferType.DOWNLOAD);
To resume all transfers of any type:

transferUtility.resumeAllWithType(TransferType.ANY);
Cancel a Transfer
Android - JavaiOS - Swift
To cancel an upload, call cancel() or cancelAllWithType() on the TransferUtility object.

The transferId can be retrieved from the TransferObserver object as described in Track Transfer Progress.

To cancel a single transfer, use:

transferUtility.cancel(idToBeCancelled);
To cancel all transfers of a certain type, use:

transferUtility.cancelAllWithType(TransferType.DOWNLOAD);
Background Transfers
The SDK supports uploading to and downloading from Amazon S3 while your app is running in the background.

Android - JavaiOS - Swift
No additional work is needed to use this feature. As long as your app is present in the background a transfer that is in progress will continue.

Advanced Transfer Methods
Topics

Transfer with Object Metadata
Transfer with Access Control List
Transfer Utility Options
Transfer with Object Metadata
Android - JavaiOS - Swift
To upload a file with metadata, use the ObjectMetadata object. Create a ObjectMetadata object and add in the metadata headers and pass it to the upload function.

import com.amazonaws.services.s3.model.ObjectMetadata;

ObjectMetadata myObjectMetadata = new ObjectMetadata();

//create a map to store user metadata
Map<String, String> userMetadata = new HashMap<String,String>();
userMetadata.put("myKey","myVal");

//call setUserMetadata on our ObjectMetadata object, passing it our map
myObjectMetadata.setUserMetadata(userMetadata);
Then, upload an object along with its metadata:

TransferObserver observer = transferUtility.upload(
  MY_BUCKET,        /* The bucket to upload to */
  OBJECT_KEY,       /* The key for the uploaded object */
  MY_FILE,          /* The file where the data to upload exists */
  myObjectMetadata  /* The ObjectMetadata associated with the object*/
);
To download the meta, use the S3 getObjectMetadata method. For more information, see the API Reference.

Transfer with Access Control List
Android - JavaiOS - Swift
To upload a file with Access Control List, use the CannedAccessControlList object. The CannedAccessControlList specifies the constants defining a canned access control list. For example, if you use CannedAccessControlList.PublicRead , this specifies the owner is granted Permission.FullControl and the GroupGrantee.AllUsers group grantee is granted Permission.Read access.

Then, upload an object along with its ACL:

TransferObserver observer = transferUtility.upload(
  MY_BUCKET,                          /* The bucket to upload to */
  OBJECT_KEY,                         /* The key for the uploaded object */
  MY_FILE,                            /* The file where the data to upload exists */
  CannedAccessControlList.PublicRead  /* Specify PublicRead ACL for the object in the bucket. */
);
Transfer Utility Options
Android - JavaiOS - Swift
You can use the TransferUtilityOptions object to customize the operations of the TransferUtility.

TransferThreadPoolSize This parameter will let you specify the number of threads in the thread pool for transfers. By increasing the number of threads, you will be able to increase the number of parts of a mulit-part upload that will be uploaded in parallel. By default, this is set to 2 * (N + 1), where N is the number of available processors on the mobile device. The minimum allowed value is 2.

TransferUtilityOptions options = new TransferUtilityOptions();
options.setTransferThreadPoolSize(8);

TransferUtility transferUtility = TransferUtility.builder()
    // Pass-in S3Client, Context, AWSConfiguration/DefaultBucket Name
    .transferUtilityOptions(options)
    .build();
TransferServiceCheckTimeInterval The TransferUtility monitors each on-going transfer by checking its status periodically. If a stalled transfer is detected, it will be automatically resumed by the TransferUtility. The TransferServiceCheckTimeInterval option allows you to set the time interval between the status checks. It is specified in milliseconds and set to 60,000 by default.

TransferUtilityOptions options = new TransferUtilityOptions();
options.setTransferServiceCheckTimeInterval(2 * 60 * 1000); // 2-minutes

TransferUtility transferUtility = TransferUtility.builder()
    // Pass-in S3Client, Context, AWSConfiguration/DefaultBucket Name
    .transferUtilityOptions(options)
    .build();
More Transfer Examples
Topics

Downloading to a File
Uploading Binary Data to a File
Downloading Binary Data to a File
This section provides descriptions and abbreviated examples of the aspects of each type of transfer that are unique. For information about typical code surrounding the following snippets see Track Transfer Progress.

Downloading to a File
The following code shows how to download an Amazon S3 Object to a local file.

Android - JavaiOS - Swift
TransferObserver downloadObserver =
    transferUtility.download(
          "s3Folder/s3Key.txt",
          new File("/path/to/file/localFile.txt"));

downloadObserver.setTransferListener(new TransferListener() {

     @Override
     public void onStateChanged(int id, TransferState state) {
        if (TransferState.COMPLETED == state) {
           // Handle a completed download.
        }
     }

     @Override
     public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
           float percentDonef = ((float)bytesCurrent/(float)bytesTotal) * 100;
           int percentDone = (int)percentDonef;

           Log.d("MainActivity", "   ID:" + id + "   bytesCurrent: " + bytesCurrent + "   bytesTotal: " + bytesTotal + " " + percentDone + "%");
     }

     @Override
     public void onError(int id, Exception ex) {
        // Handle errors
     }

});
Uploading Binary Data to a File
Android - JavaiOS - Swift
Use the following code to upload binary data to a file in Amazon S3.

TransferObserver uploadObserver =
        transferUtility.upload(
              "s3Folder/s3Key.bin",
              new File("/path/to/file/localFile.bin"));

uploadObserver.setTransferListener(new TransferListener() {

     @Override
     public void onStateChanged(int id, TransferState state) {
        if (TransferState.COMPLETED == state) {
           // Handle a completed upload.
        }
     }

     @Override
     public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
           float percentDonef = ((float)bytesCurrent/(float)bytesTotal) * 100;
           int percentDone = (int)percentDonef;

           Log.d("MainActivity", "   ID:" + id + "   bytesCurrent: " + bytesCurrent + "   bytesTotal: " + bytesTotal + " " + percentDone + "%");
     }

     @Override
     public void onError(int id, Exception ex) {
        // Handle errors
     }

});
Downloading Binary Data to a File
The following code shows how to download a binary file.

Android - JavaiOS - Swift
TransferObserver downloadObserver =
    transferUtility.download(
          "s3Folder/s3Key.bin",
          new File("/path/to/file/localFile.bin"));

downloadObserver.setTransferListener(new TransferListener() {

     @Override
     public void onStateChanged(int id, TransferState state) {
        if (TransferState.COMPLETED == state) {
           // Handle a completed download.
        }
     }
     @Override
     public void onProgressChanged(int id, long bytesCurrent, long bytesTotal) {
           float percentDonef = ((float)bytesCurrent/(float)bytesTotal) * 100;
           int percentDone = (int)percentDonef;

           Log.d("MainActivity", "   ID:" + id + "   bytesCurrent: " + bytesCurrent + "   bytesTotal: " + bytesTotal + " " + percentDone + "%");
     }

     @Override
     public void onError(int id, Exception ex) {
        // Handle errors
     }

});
Limitations
Android - JavaiOS - Swift
If you expect your app to perform transfers that take longer than 50 minutes, use AmazonS3Client instead of TransferUtility.

TransferUtility generates Amazon S3 pre-signed URLs to use for background data transfer. Using Amazon Cognito Identity, you receive AWS temporary credentials. 
The credentials are valid for up to 60 minutes. Generated Amazon S3 pre-signed URLs cannot last longer than that time. Because of this limitation, the Amazon 
S3 Transfer Utility enforces 50 minute transfer timeouts, leaving a 10 minute buffer before AWS temporary credentials are regenerated. After 50 minutes, you receive a transfer failure.

Amazon S3 Pre-Signed URLs: For Background Transfer
If you are working with large file transfers, you may want to perform uploads and downloads in the background. To do this, you need to create a background session using NSURLSession and then transfer your objects using pre-signed URLs.

The following sections describe pre-signed S3 URLs. To learn more about NSURLSession, see Using NSURLSession.

Pre-Signed URLs
By default, all Amazon S3 resources are private. If you want your users to have access to Amazon S3 buckets or objects, you can assign appropriate permissions with an IAM policy.

Alternatively, you can use pre-signed URLs to give your users access to Amazon S3 objects. A pre-signed URL provides access to an object without requiring AWS security credentials or permissions.

When you create a pre-signed URL, you must provide your security credentials, specify a bucket name, an object key, an HTTP method, and an expiration date and time. The pre-signed URL is valid only for the specified duration.

Build a Pre-Signed URL
The following example shows how to build a pre-signed URL for an Amazon S3 download in the background.

iOS - SwiftiOS - Objective-C
AWSS3PreSignedURLBuilder.default().getPreSignedURL(getPreSignedURLRequest).continueWith { (task:AWSTask<NSURL>) -> Any? in
    if let error = task.error as? NSError {
        print("Error: \(error)")
        return nil
    }

    let presignedURL = task.result
    print("Download presignedURL is: \(presignedURL)")

    let request = URLRequest(url: presignedURL as! URL)
    let downloadTask: URLSessionDownloadTask = URLSession.shared.downloadTask(with: request)
    downloadTask.resume()

    return nil
}
The preceding example uses GET as the HTTP method: AWSHTTPMethodGET. For an upload request to Amazon S3, we would need to use a PUT method and also specify a content type.

iOS - SwiftiOS - Objective-C
getPreSignedURLRequest.httpMethod = .PUT
let fileContentTypeStr = "text/plain"
getPreSignedURLRequest.contentType = fileContentTypeStr
Here's an example of building a pre-signed URL for a background upload to S3.

iOS - SwiftiOS - Objective-C
let getPreSignedURLRequest = AWSS3GetPreSignedURLRequest()
getPreSignedURLRequest.bucket = "myBucket"
getPreSignedURLRequest.key = "myFile.txt"
getPreSignedURLRequest.httpMethod = .PUT
getPreSignedURLRequest.expires = Date(timeIntervalSinceNow: 3600)

//Important: set contentType for a PUT request.
let fileContentTypeStr = "text/plain"
getPreSignedURLRequest.contentType = fileContentTypeStr

AWSS3PreSignedURLBuilder.default().getPreSignedURL(getPreSignedURLRequest).continueWith { (task:AWSTask<NSURL>) -> Any? in
    if let error = task.error as? NSError {
        print("Error: \(error)")
        return nil
    }

    let presignedURL = task.result
    print("Download presignedURL is: \(presignedURL)")

    var request = URLRequest(url: presignedURL as! URL)
    request.cachePolicy = .reloadIgnoringLocalCacheData
    request.httpMethod = "PUT"
    request.setValue(fileContentTypeStr, forHTTPHeaderField: "Content-Type")

    let uploadTask: URLSessionTask = URLSession.shared.uploadTask(with: request, fromFile: URL(fileURLWithPath: "your/file/path/myFile.txt"))
    uploadTask.resume()

    return nil
}

Integrate Your Existing NoSQL Table

Just Getting Started?

Use streamlined steps to install the SDK and integrate features.

The Get Started section of this guide allows you to create new resources and complete the steps described on this page in minutes. If you want to import existing resources or create them from scratch, the contents of this page will walk you through the steps you need.

The following steps and examples are based on a simple bookstore app. The app tracks the books that are available in the bookstore using an Amazon DynamoDB table.

Set up Your Backend
To manually configure an Amazon DynamoDB table that you can integrate into your mobile app, use the following steps.

Topics

Create an New Table and Index
Set Up an Identity Pool
Set Permissions
Apply Permissions
Create an New Table and Index
If you already have an Amazon DynamoDB table and know its region, you can skip to Set Up an Identity Pool.

To create the Books table:

Sign in to the Amazon DynamoDB Console.

Choose Create Table.

Type Books as the name of the table.

Enter ISBN in the Partition key field of the Primary key with String as their type.

Check the Add sort key box , then type Category in the provided field and select String as the type.

Clear the Use default settings checkbox and choose + Add Index.

In the Add Index dialog type Author with String as the type.

Check the Add sort key checkbox and enter Title as the sort key value, with String as its type.

Leave the other values at their defaults. Choose Add index to add the Author-Title-index index.

Set the Minimum provisioned capacity for read to 10, and for write to 5.

Choose Create.Amazon DynamoDB will create your database.

Refresh the console and choose your Books table from the list of tables.

Open the Overview tab and copy or note the Amazon Resource Name (ARN). You need this for the next procedure.

Set Up an Identity Pool
To give your users permissions to access your table you'll need an identity pool from Amazon Cognito. That pool has two default IAM roles, one for guest (unauthenticated), and one for signed-in (authenticated) users. The policies you design and attach to the IAM roles determine what each type of user can and cannot do.

Import an existing pool or create a new pool for your app.

Set Permissions
Attach the following IAM policy to the unauthenticated role for your identity pool. It allows the user to perform the actions on two resources (a table and an index) identified by the ARN of your Amazon DynamoDB table.

{
    "Statement": [{
        "Effect": "Allow",
        "Action": [
            "dynamodb:DeleteItem",
            "dynamodb:GetItem",
            "dynamodb:PutItem",
            "dynamodb:Scan",
            "dynamodb:Query",
            "dynamodb:UpdateItem",
            "dynamodb:BatchWriteItem"
        ],
        "Resource": [
            "arn:aws:dynamodb:us-west-2:123456789012:table/Books",
            "arn:aws:dynamodb:us-west-2:123456789012:table/Books/index/*"
        ]
    }]
}
Apply Permissions
Apply this policy to the unauthenticated role assigned to your Amazon Cognito identity pool, replacing the Resource values with the correct ARN for the Amazon DynamoDB table:

Sign in to the IAM console.

Choose Roles and then choose the "Unauth" role that Amazon Cognito created for you.

Choose Attach Role Policy.

Choose Custom Policy and then Choose Select.

Type a name for your policy and paste in the policy document shown above, replacing the Resource values with the ARNs for your table and index. (You can retrieve the table ARN from the Details tab of the database; then append /index/* to obtain the value for the index ARN.

Choose Apply Policy.

Connect to Your Backend
Topics

Create Your AWS Configuration File
Add the AWS Config File
Add the SDK to your App
Add Data Models to Your App
Create Your AWS Configuration File
Your app is connected to your AWS resources using an awsconfiguration.json file which contains the endpoints for the services you use.

Create a file with name awsconfiguration.json with the following contents:

{
  "Version": "1.0",
  "CredentialsProvider": {
    "CognitoIdentity": {
      "Default": {
        "PoolId": "COGNITO-IDENTITY-POOL-ID",
        "Region": "COGNITO-IDENTITY-POOL-REGION"
      }
    }
  },
  "IdentityManager": {
    "Default": {}
  },
  "DynamoDBObjectMapper": {
    "Default": {
      "Region": "DYNAMODB-REGION"
    }
  }
}
Make the following changes to the configuration file.

Replace the DYNAMODB-REGION with the region the table was created in.


Need to find your table's region?

Go to Amazon DynamoDB Console. and choose the Overview tab for your table. The Amazon Resource Name (ARN) item shows the table's ID, which contains its region.

For example, if your pool ID is arn:aws:dynamodb:us-east-1:012345678901:table/nosqltest-mobilehub-012345678-Books, then your the table's region value would be us-east-1.

The configuration file value you want is in the form of: "Region": "REGION-OF-YOU-DYNAMODB-ARN". For this example:

"Region": "us-east-1"
Replace the COGNITO-IDENTITY-POOL-ID with the identity pool ID.

Replace the COGNITO-IDENTITY-POOL-REGION with the region the identity pool was created in.


Need to find your pool's ID and region?

Go to Amazon Cognito Console and choose Manage Federated Identities, then choose your pool and choose Edit identity pool. Copy the value of Identity pool ID.

Insert this region value into the following form to create the value you need for this integration.

"Region": "REGION-PREFIX-OF-YOUR-POOL-ID".
For example, if your pool ID is us-east-1:01234567-yyyy-0123-xxxx-012345678901, then your integration region value would be:

"Region": "us-east-1"
Add the AWS Config File
To make the connection between your app and your backend services, add the configuration file.

Android - JavaiOS - SwiftiOS - Swift
Right-click your app's res folder, and then choose New > Android Resource Directory. Select raw in the Resource type dropdown menu.


                        Image of selecting a Raw Android Resource Directory in Android Studio.
                     
Drag the awsconfiguration.json you created into the res/raw folder. Android gives a resource ID to any arbitrary file placed in this folder, making it easy to reference in the app.

Add the SDK to your App
Use the following steps to add AWS Mobile NoSQL Database to your app.

Android - JavaiOS - Swift
Set up AWS Mobile SDK components with the following Set Up Your Backend steps.

app/build.gradle must contain:

 dependencies{

     // Amazon Cognito dependencies for user access to AWS resources
     implementation ('com.amazonaws:aws-android-sdk-mobile-client:2.6.+@aar') { transitive = true }

     // AmazonDynamoDB dependencies for NoSQL Database
     implementation 'com.amazonaws:aws-android-sdk-ddb-mapper:2.6.+'

     // other dependencies . . .
 }
Add the following permissions to AndroidManifest.xml.

<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
Create an AWSDynamoDBMapper client in the call back of your call to instantiate AWSMobileClient. This will ensure that the AWS credentials needed to connect to Amazon DynamoDB are available, and is typically in onCreate function of of your start up activity.

import com.amazonaws.mobile.client.AWSMobileClient;
import com.amazonaws.mobile.client.AWSStartupHandler;
import com.amazonaws.mobile.client.AWSStartupResult;

import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.DynamoDBMapper;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClient;

public class MainActivity extends AppCompatActivity {

    // Declare a DynamoDBMapper object
    DynamoDBMapper dynamoDBMapper;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // AWSMobileClient enables AWS user credentials to access your table
        AWSMobileClient.getInstance().initialize(this, new AWSStartupHandler() {

            @Override
            public void onComplete(AWSStartupResult awsStartupResult) {

                    // Add code to instantiate a AmazonDynamoDBClient
                    AmazonDynamoDBClient dynamoDBClient = new AmazonDynamoDBClient(AWSMobileClient.getInstance().getCredentialsProvider());
                    this.dynamoDBMapper = DynamoDBMapper.builder()
                        .dynamoDBClient(dynamoDBClient)
                        .awsConfiguration(
                        AWSMobileClient.getInstance().getConfiguration())
                        .build();

            }
        }).execute();

        // Other functions in onCreate . . .
    }
}

Important

Use Asynchronous Calls to DynamoDB

Since calls to DynamoDB are synchronous, they don't belong on your UI thread. Use an asynchronous method like the Runnable wrapper to call DynamoDBObjectMapper in a separate thread.

Runnable runnable = new Runnable() {
     public void run() {
       //DynamoDB calls go here
     }
};
Thread mythread = new Thread(runnable);
mythread.start();
Add Data Models to Your App
To connect your app to your table create a data model object in the following form. In this example, the model is based on the Books table you created in a previous step. The partition key (hash key) is called ISBN and the sort key (rangekey) is called Category.

Android - JavaiOS - Swift
In the Android Studio project explorer right-click the folder containing your main activity, and choose New > Java Class. Type the Name you will use to refer to your data model. In this example the name would be BooksDO. Add code in the following form.

package com.amazonaws.models.nosql;

import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.DynamoDBAttribute;
import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.DynamoDBHashKey;
import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.DynamoDBIndexHashKey;
import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.DynamoDBIndexRangeKey;
import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.DynamoDBRangeKey;
import com.amazonaws.mobileconnectors.dynamodbv2.dynamodbmapper.DynamoDBTable;

import java.util.List;
import java.util.Map;
import java.util.Set;

@DynamoDBTable(tableName = "Books")

public class BooksDO {
    private String _isbn;
    private String _category;
    private String _title;
    private String _author;

    @DynamoDBHashKey(attributeName = "ISBN")
    @DynamoDBAttribute(attributeName = "ISBN")
    public String getIsbn() {
        return _isbn;
    }

    public void setIsbn(final String _isbn) {
        this._isbn = _isbn;
    }

    @DynamoDBRangeKey (attributeName = "Category")
    @DynamoDBAttribute(attributeName = "Category")
    public String getCategory() {
        return _category;
    }

    public void setCategory(final String _category) {
        this._category= _category;
    }

    @DynamoDBIndexHashKey(attributeName = "Author", globalSecondaryIndexName = "Author")
    public String getAuthor() {
        return _author;
    }

    public void setAuthor(final String _author) {
        this._author = _author;
    }

    @DynamoDBIndexRangeKey(attributeName = "Title", globalSecondaryIndexName = "Title")
    public String getTitle() {
        return _title;
    }

    public void setTitle(final String _title) {
        this._title = _title;
    }

}
Perform CRUD Operations
The fragments below consume the BooksDO data model class created in a previous step.

Topics

Create (Save) an Item
Read (Load) an Item
Update an Item
Delete an Item
Create (Save) an Item
Use the following code to create an item in your NoSQL Database table.

Android - JavaiOS - Swift
public void createBooks() {
    final com.amazonaws.models.nosql.BooksDO booksItem = new com.amazonaws.models.nosql.BooksDO();

    booksItem.setIsbn("ISBN1");
    booksItem.setAuthor("Frederick Douglas");
    booksItem.setTitle("Escape from Slavery");
    booksItem.setCategory("History");

    new Thread(new Runnable() {
        @Override
        public void run() {
            dynamoDBMapper.save(booksItem);
            // Item saved
        }
    }).start();
}
Read (Load) an Item
Use the following code to read an item in your NoSQL Database table.

Android - JavaiOS - Swift
public void readBooks() {
    new Thread(new Runnable() {
        @Override
        public void run() {

            com.amazonaws.models.nosql.BooksDO booksItem = dynamoDBMapper.load(
                    com.amazonaws.models.nosql.BooksDO.class,
                    "ISBN1",       // Partition key (hash key)
                    "History");    // Sort key (range key)

            // Item read
             Log.d("Books Item:", booksItem.toString());
        }
    }).start();
}
Update an Item
Use the following code to update an item in your NoSQL Database table.

Android - JavaiOS - Swift
public void updateBooks() {
    final com.amazonaws.models.nosql.BooksDO booksItem = new com.amazonaws.models.nosql.BooksDO();


    booksItem.setIsbn("ISBN1");
    booksItem.setCategory("History");
    booksItem.setAuthor("Frederick M. Douglas");
    //  booksItem.setTitle("Escape from Slavery");

    new Thread(new Runnable() {
        @Override
        public void run() {


            // Using .save(bookItem) with no Title value makes that attribute value equal null
            // The .Savebehavior shown here leaves the existing value as is
            dynamoDBMapper.save(booksItem, new DynamoDBMapperConfig(DynamoDBMapperConfig.SaveBehavior.UPDATE_SKIP_NULL_ATTRIBUTES));

            // Item updated
        }
    }).start();
}
Delete an Item
Use the following code to delete an item in your NoSQL Database table.

Android - JavaiOS - Swift
public void deleteBooks() {
    new Thread(new Runnable() {
        @Override
        public void run() {

            com.amazonaws.models.nosql.BooksDO booksItem = new com.amazonaws.models.nosql.BooksDO();
            booksItem.setIsbn("ISBN1");       //partition key
            booksItem.setCategory("History"); //range key

            dynamoDBMapper.delete(booksItem);

            // Item deleted
        }
    }).start();
}
Perform a Query
A query operation enables you to find items in a table. You must define a query using both the hash key (partition key) and range key (sort key) attributes of a table. You can filter the results by specifying the attributes you are looking for. For more information about DynamoDBQueryExpression, see the AWS Mobile SDK for Android API reference.

The following example code shows querying for books with partition key (hash key) ISBN and sort key (range key) Category beginning with History.

Android - JavaiOS - Swift
public void queryBook() {

     new Thread(new Runnable() {
         @Override
         public int hashCode() {
             return super.hashCode();
         }

         @Override
         public void run() {
             com.amazonaws.models.nosql.BooksDO book = new com.amazonaws.models.nosql.BooksDO();
             book.setIsbn("ISBN1");       //partition key
             book.setCategory("History"); //range key


             Condition rangeKeyCondition = new Condition()
                     .withComparisonOperator(ComparisonOperator.BEGINS_WITH)
                     .withAttributeValueList(new AttributeValue().withS("History"));
             DynamoDBQueryExpression queryExpression = new DynamoDBQueryExpression()
                     .withHashKeyValues(book)
                     .withRangeKeyCondition("Category", rangeKeyCondition)
                     .withConsistentRead(false);

             PaginatedList<BooksDO> result = dynamoDBMapper.query(com.amazonaws.models.nosql.BooksDO.class, queryExpression);

             Gson gson = new Gson();
             StringBuilder stringBuilder = new StringBuilder();

             // Loop through query results
             for (int i = 0; i < result.size(); i++) {
                 String jsonFormOfItem = gson.toJson(result.get(i));
                 stringBuilder.append(jsonFormOfItem + "\n\n");
             }

             // Add your code here to deal with the data result
             Log.d("Query results: ", stringBuilder.toString());

             if (result.isEmpty()) {
                 // There were no items matching your query.
             }
         }
     }).start();
 }
 
 Amazon DynamoDB Object Mapper API
Topics

Overview
Setup
Instantiate the Object Mapper API
CRUD Operations
Perform a Scan
Perform a Query
Additional Resources
Overview
Amazon DynamoDB is a fast, highly scalable, highly available, cost-effective, non-relational database service. Amazon DynamoDB removes traditional scalability limitations on data storage while maintaining low latency and predictable performance.

The AWS Mobile SDK for iOS provides both low-level and high-level libraries for working with Amazon DynamoDB.

The high-level library described in this section provides Amazon DynamoDB object mapper which lets you map client-side classes to tables. Working within the data model defined on your client you can write simple, readable code that stores and retrieves objects in the cloud.

The dynamodb-low-level-client provides useful ways to perform operations like conditional writes and batch operations.

Setup
To set your project up to use the AWS SDK for iOS dynamoDBObjectMapper, take the following steps.

Setup the SDK, Credentials, and Services
To integrate dynamoDBObjectMapper into a new app, follow the steps described in Get Started to install the AWS Mobile SDK for iOS.

For apps that use an SDK version prior to 2.6.0, follow the steps on setup-options-for-aws-sdk-for-ios to install the AWS Mobile SDK for iOS. Then use the steps on cognito-auth-identity-for-ios-legacy to configure user credentials, and permissions.

Instantiate the Object Mapper API
In this section:

Topics

Import the AWSDynamoDB APIs
Create Amazon DynamoDB Object Mapper Client
Define a Mapping Class
Import the AWSDynamoDB APIs
Add the following import statement to your project.

iOS - SwiftiOS - Objective-C
import AWSDynamoDB
Create Amazon DynamoDB Object Mapper Client
Use the AWSDynamoDBObjectMapper to map a client-side class to your database. The object mapper supports high-level operations like creating, getting, querying, updating, and deleting records. Create an object mapper as follows.

iOS - SwiftiOS - Objective-C
dynamoDBObjectMapper = AWSDynamoDBObjectMapper.default()
Object mapper methods return an AWSTask object. for more information, see Working with Asynchronous Tasks.

Define a Mapping Class
An Amazon DynamoDB database is a collection of tables, and a table can be described as follows:

A table is a collection of items.

Each item is a collection of attributes.

Each attribute has a name and a value.

For the bookstore app, each item in the table represents a book, and each item has four attributes: Title, Author, Price, and ISBN.

Each item (Book) in the table has a Primary key, in this case, the primary key is ISBN.

To directly manipulate database items through their object representation, map each item in the Book table to a Book object in the client-side code, as shown in the following code. Attribute names are case sensitive.

iOS - SwiftiOS - Objective-C
import AWSDynamoDB

class Book : AWSDynamoDBObjectModel, AWSDynamoDBModeling  {
    var Title:String?
    var Author:String?
    var Price:String?
    var ISBN:String?

    class Amazon DynamoDBTableName() -> String {
        return "Books"
    }

    class func hashKeyAttribute() -> String {
        return "ISBN"
    }
}
Note

As of SDK version 2.0.16, the AWSDynamoDBModel mapping class is deprecated and replaced by AWSDynamoDBObjectModel. For information on migrating your legacy code, see awsdynamodb-model.

To conform to the AWSDynamoDBModeling protocol, implement dynamoDBTableName, which returns the name of the table, and hashKeyAttribute, which returns the name of the primary key. If the table has a range key, implement + (NSString *)rangeKeyAttribute.

CRUD Operations
In this section:

Topics

Save an Item
Retrieve an Item
Update an Item
Delete an Item
The Amazon DynamoDB table, mapping class, and object mapper client enable your app to interact with objects in the cloud.

Save an Item
The save: method saves an object to Amazon DynamoDB, using the default configuration. As a parameter, save: takes a an object that inherits from AWSDynamoDBObjectModel and conforms to the AWSDynamoDBModeling protocol. The properties of this object will be mapped to attributes in Amazon DynamoDB table.

To create the object to be saved take the following steps.

Define the object and it's properties to match your table model.

iOS - SwiftiOS - Objective-C
let myBook = Book()
myBook?.ISBN = "3456789012"
myBook?.Title = "The Scarlet Letter"
myBook?.Author = "Nathaniel Hawthorne"
myBook?.Price = 899 as NSNumber?
Pass the object to the save: method.

iOS - SwiftiOS - Objective-C
dynamoDBObjectMapper.save(myBook).continueWith(block: { (task:AWSTask<AnyObject>!) -> Any? in
     if let error = task.error as? NSError {
         print("The request failed. Error: \(error)")
     } else {
         // Do something with task.result or perform other operations.
     }
 })
Save Behavior Options
The AWS Mobile SDK for iOS supports the following save behavior options:

AWSDynamoDBObjectMapperSaveBehaviorUpdate

This option does not affect unmodeled attributes on a save operation. Passing a nil value for the modeled attribute removes the attribute from the corresponding item in Amazon DynamoDB. By default, the object mapper uses this behavior.

AWSDynamoDBObjectMapperSaveBehaviorUpdateSkipNullAttributes

This option is similar to the default update behavior, except that it ignores any null value attribute(s) and does not remove them from an item in Amazon DynamoDB.

AWSDynamoDBObjectMapperSaveBehaviorAppendSet

This option treats scalar attributes (String, Number, Binary) the same as the AWSDynamoDBObjectMapperSaveBehaviorUpdateSkipNullAttributes option. However, for set attributes, this option appends to the existing attribute value instead of overriding it. The caller must ensure that the modeled attribute type matches the existing set type; otherwise, a service exception occurs.

AWSDynamoDBObjectMapperSaveBehaviorClobber

This option clears and replaces all attributes, including unmodeled ones, on save. Versioned field constraints are be disregarded.

The following code provides an example of setting a default save behavior on the object mapper.

iOS - SwiftiOS - Objective-C
let updateMapperConfig = AWSDynamoDBObjectMapperConfiguration()
updateMapperConfig.saveBehavior = .updateSkipNullAttributes
Use updateMapperConfig as an argument when calling save:configuration:.

Retrieve an Item
Using an object's primary key, in this case, ISBN, we can load the corresponding item from the database. The following code returns the Book item with an ISBN of 6543210987.

iOS - SwiftiOS - Objective-C
dynamoDBObjectMapper.load(Book.self, hashKey: "6543210987" rangeKey:nil).continueWith(block: { (task:AWSTask<AnyObject>!) -> Any? in
     if let error = task.error as? NSError {
         print("The request failed. Error: \(error)")
     } else if let resultBook = task.result as? Book {
         // Do something with task.result.
     }
     return nil
 })
The object mapper creates a mapping between the Book item returned from the database and the Book object on the client (here, resultBook). Access the title at resultBook.Title.

Since the Books database does not have a range key, nil was passed to the rangeKey parameter.

Update an Item
To update an item in the database, just set new attributes and save the objects. The primary key of an existing item, myBook.ISBN in the Book object mapper example, cannot be changed. If you save an existing object with a new primary key, a new item with the same attributes and the new primary key are created.

Delete an Item
To delete a table row, use the remove: method.

iOS - SwiftiOS - Objective-C
 let bookToDelete = Book()
 bookToDelete?.ISBN = "4456789012";

dynamoDBObjectMapper.remove(bookToDelete).continueWith(block: { (task:AWSTask<AnyObject>!) -> Any? in
     if let error = task.error as? NSError {
         print("The request failed. Error: \(error)")
     } else {
         // Item deleted.
     }
 })
Perform a Scan
A scan operation retrieves in an undetermined order.

The scan:expression: method takes two parameters: the class of the resulting object and an instance of AWSDynamoDBScanExpression, which provides options for filtering results.

The following example shows how to create an AWSDynamoDBScanExpression object, set its limit property, and then pass the Book class and the expression object to scan:expression:.

iOS - SwiftiOS - Objective-C
 let scanExpression = AWSDynamoDBScanExpression()
 scanExpression.limit = 20

dynamoDBObjectMapper.scan(Book.self, expression: scanExpression).continueWith(block: { (task:AWSTask<AnyObject>!) -> Any? in
     if let error = task.error as? NSError {
         print("The request failed. Error: \(error)")
     } else if let paginatedOutput = task.result {
         for book in paginatedOutput.items as! Book {
             // Do something with book.
         }
     }
 })
Filter a Scan ~~~~~~~~~~~~-

The output of a scan is returned as an AWSDynamoDBPaginatedOutput object. The array of returned items is in the items property.

The scanExpression` method provides several optional parameters. Use ``filterExpression and expressionAttributeValues to specify a scan result for the attribute names and conditions you define. For more information about the parameters and the API, see AWSDynamoDBScanExpression.

The following code scans the Books table to find books with a price less than 50.

iOS - SwiftiOS - Objective-C
 let scanExpression = AWSDynamoDBScanExpression()
 scanExpression.limit = 10
 scanExpression.filterExpression = "Price < :val"
 scanExpression.expressionAttributeValues = [":val": 50]

dynamoDBObjectMapper.scan(Book.self, expression: scanExpression).continueWith(block: { (task:AWSTask<AnyObject>!) -> Any? in
   if let error = task.error as? NSError {
       print("The request failed. Error: \(error)")
   } else if let paginatedOutput = task.result {
       for book in paginatedOutput.items as! Book {
           // Do something with book.
       }
   }
 })
You can also use the projectionExpression` property to specify the attributes to retrieve from the ``Books table. For example adding scanExpression.projectionExpression = @"ISBN, Title, Price"; in the previous code snippet retrieves only those three properties in the book object. The Author property in the book object will always be nil.

Perform a Query
The query API enables you to query a table or a secondary index. The query:expression: method takes two parameters: the class of the resulting object and an instance of AWSDynamoDBQueryExpression.

To query an index, you must also specify the indexName. You must specify the hashKeyAttribute if you query a global secondary with a different hashKey. If the table or index has a range key, you can optionally refine the results by providing a range key value and a condition.

The following example illustrates querying the Books index table to find all books whose author is "John Smith", with a price less than 50.

iOS - SwiftiOS - Objective-C
 let queryExpression = AWSDynamoDBQueryExpression()
 queryExpression.indexName = "Author-Price-index"

 queryExpression.keyConditionExpression = @"Author = :authorName AND Price < :val";
 queryExpression.expressionAttributeValues = @{@":authorName": @"John Smith", @":val": @50};

dynamoDBObjectMapper.query(Book.self, expression: queryExpression).continueWith(block: { (task:AWSTask<AnyObject>!) -> Any? in
     if let error = task.error as? NSError {
           print("The request failed. Error: \(error)")
     } else if let paginatedOutput = task.result {
         for book in paginateOutput.items as! Book {
             // Do something with book.
         }
     }
     return nil
 })
In the preceding example, indexName is specified to demonstrate querying an index. The query expression is specified using keyConditionExpression and the values used in the expression using expressionAttributeValues.

You can also provide filterExpression and projectionExpression in AWSDynamoDBQueryExpression. The syntax is the same as that used in a scan operation.

For more information, see AWSDynamoDBQueryExpression.

Migrating AWSDynamoDBModel to AWSDynamoDBObjectModel

As of SDK version 2.0.16, the AWSDynamoDBModel mapping class is deprecated and replaced by AWSDynamoDBObjectModel.The deprecated AWSDynamoDBModel used NSArray 
to represent multi-valued types (String Set, Number Set, and Binary Set); it did not support Boolean, Map, or List types. The new AWSDynamoDBObjectModel uses NSSet 
for multi-valued types and supports Boolean, Map, and List. For the Boolean type, you create an NSNumber 
using [NSNumber numberWithBool:YES] or using the shortcuts @YES and @NO. For the Map type, create using NSDictionary. For the List type, create using NSArray.

Android: Execute Code On Demand with AWS Lambda

Just Getting Started?

Use streamlined steps to install the SDK and integrate features.

Or, use the contents of this page if your app will integrate existing AWS services.

Overview
AWS Lambda is a compute service that runs your code in response to events and automatically manages the compute resources for you, making it easy to build applications that respond quickly to new information. The AWS Mobile SDK for Android enables you to call Lambda functions from your Android mobile apps.

The tutorial below explains how to integrate AWS Lambda with your app.

Setup
Prerequisites
You must complete all of the instructions on the Android: Setup Options for the SDK page before beginning this tutorial.

Create a Lambda Function in the AWS Console
For this tutorial, let's use a simple "echo" function that returns the input. Follow the steps described at Amazon Lambda Getting Started, replacing the function code with the code below:

exports.
handler = function(event, context) {
     console.log("Received event");
     context.succeed("Hello "+ event.firstName + "using " + context.clientContext.deviceManufacturer);
  }
Set IAM Permissions
The default IAM role policy grants your users access to Amazon Mobile Analytics and Amazon Cognito Sync. To use AWS Lambda in your application, you must configure the IAM role policy so that it allows your application and your users access to AWS Lambda. The IAM policy in the following steps allows the user to perform the actions shown in this tutorial on a given AWS Lambda function identified by its Amazon Resource Name (ARN). To find the ARN go to the Lambda Console and click the Function name.

To set IAM Permissions for AWS Lambda:

Navigate to the IAM Console and click Roles in the left-hand pane.

Type your identity pool name into the search box. Two roles will be listed: one for unauthenticated users and one for authenticated users.

Click the role for unauthenticated users (it will have unauth appended to your Identity Pool name).

Click the Create Role Policy button, select Custom Policy, and then click the Select button.

Enter a name for your policy and paste in the following policy document, replacing the function’s Resource value with the ARN for your function (click your function’s Function name in the AWS Lambda console to view its ARN).

{
   "Statement": [{
      "Effect": "Allow",
      "Action": [
          "lambda:invokefunction"
      ],
      "Resource": [
         ”arn:aws:lambda:us-west-2:012345678901:function:yourFunctionName”
      ]
   }]
}
Click the Add Statement button, and then click the Next Step button. The wizard will show you the configuration that you generated.

Click the Apply Policy button.

To learn more about IAM policies, see IAM documentation.

Set Permissions in Your Android Manifest
In your AndroidManifest.xml, add the following permission

<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
Initialize LambdaInvokerFactory
Pass your initialized Amazon Cognito credentials provider to the LambdaInvokerFactory constructor:

LambdaInvokerFactory factory = new LambdaInvokerFactory(
  myActivity.getApplicationContext(),
  REGION,
  credentialsProvider);
Declare Data Types
Declare the Java classes to hold the data you pass to the Lambda function. The following class defines a NameInfo class that contains a person's first and last name:

package com.amazonaws.demo.lambdainvoker;

/**
 * A simple POJO
 */
 public class NameInfo {
    private String firstName;
    private String lastName;

    public NameInfo() {}

    public NameInfo(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }

    public String getFirstName() {
       return firstName;
    }

    public void setFirstName(String firstName) {
       this.firstName = firstName;
    }

    public String getLastName() {
       return lastName;
    }

    public void setLastName(String lastName) {
        this.lastName = lastName;
    }
}
Create a Lambda proxy
Declare an interface containing one method for each Lambda function call. Each method in the interface must be decorated with the "@LambdaFunction" annotation. The LambdaFunction attribute can take 3 optional parameters:

functionName allows you to specify the name of the Lambda function to call when the method is executed, by default the name of the method is used.

logType is valid only when invocationType is set to "Event". If set, AWS Lambda will return the last 4KB of log data produced by your Lambda Function in the x-amz-log-results header.

invocationType specifies how the Lambda function will be invoked. Can be one of the following values:

Event: calls the Lambda Function asynchronously

RequestResponse: calls the Lambda Function synchronously

DryRun: allows you to validate access to a Lambda Function without executing it

The following code shows how to create a Lambda proxy:

package com.amazonaws.demo.lambdainvoker;
import com.amazonaws.mobileconnectors.lambdainvoker.LambdaFunction;

/*
 * A holder for lambda functions
 */
public interface MyInterface {

   /**
    * Invoke lambda function "echo". The function name is the method name
    */
   @LambdaFunction
   String echo(NameInfo nameInfo);

   /**
    * Invoke lambda function "echo". The functionName in the annotation
    * overrides the default which is the method name
    */
   @LambdaFunction(functionName = "echo")
   void noEcho(NameInfo nameInfo);
}
Invoke the Lambda Function
Note

Do not invoke the Lambda function from the main thread as it results in a network call.

The following code shows how to initialize the Cognito Caching Credentials Provider and invoke a Lambda function. The value for IDENTITY_POOL_ID will be specific to your account. Ensure the region is the same as the Lambda function you are trying to invoke.

// Create an instance of CognitoCachingCredentialsProvider
CognitoCachingCredentialsProvider credentialsProvider = new CognitoCachingCredentialsProvider(
     myActivity.getApplicationContext(),
     IDENTITY_POOL_ID,
     Regions.YOUR_REGION);

// Create a LambdaInvokerFactory, to be used to instantiate the Lambda proxy
LambdaInvokerFactory factory = new LambdaInvokerFactory(
  myActivity.getApplicationContext(),
  REGION,
  credentialsProvider);

// Create the Lambda proxy object with default Json data binder.
// You can provide your own data binder by implementing
// LambdaDataBinder
MyInterface myInterface = factory.build(MyInterface.class);

NameInfo nameInfo = new NameInfo("John", "Doe");

// The Lambda function invocation results in a network call
// Make sure it is not called from the main thread
new AsyncTask<NameInfo, Void, String>() {
    @Override
    protected String doInBackground(NameInfo... params) {
    // invoke "echo" method. In case it fails, it will throw a
    // LambdaFunctionException.
    try {
            return myInterface.echo(params[0]);
     } catch (LambdaFunctionException lfe) {
         Log.e(TAG, "Failed to invoke echo", lfe);
         return null;
      }
 }

@Override
protected void onPostExecute(String result) {
    if (result == null) {
        return;
     }

        // Do a toast
        Toast.makeText(MainActivity.this, result, Toast.LENGTH_LONG).show();
    }
}.execute(nameInfo);
Now whenever the Lambda function is invoked, you should see an application toast with the text "Hello John using <device>".

For more information on accessing AWS Lambda, see lambda.

Android: Use Natural Language with Amazon Lex

Just Getting Started?

Use streamlined steps to install the SDK and integrate Amazon Lex.

Or, use the content on this page if your app integrates existing AWS services.

Overview
Amazon Lex is an AWS service for building voice and text conversational interfaces into applications. With Amazon Lex, the same natural language understanding engine that powers Amazon Alexa is now available to any developer, enabling you to build sophisticated, natural language chatbots into your new and existing applications.

The AWS Mobile SDK for Android provides an optimized client for interacting with Amazon Lex runtime APIs, which support both voice and text input and can return either voice or text. Amazon Lex has built-in integration with AWS Lambda to allow insertion of custom business logic into your Amazon Lex processing flow, including all of the extension to other services that Lambda makes possible.

For information on Amazon Lex concepts and service configuration, see How it Works in the Lex Developer Guide.

For information about Amazon Lex Region availability, see AWS Service Region Availability.

To get started using the Amazon Lex mobile client, integrate the SDK for Android into your app, set the appropriate permissions, and import the necessary libraries.

Setting Up
Include the SDK in Your Project
Follow the instructions at setup-legacy to include the JAR files for this service and set the appropriate permissions.

Set Permissions in Your Android Manifest
In your AndroidManifest.xml file, add the following permission:

<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
Declare Amazon Lex as a Gradle dependency
Make sure the following Gradle build dependency is declared in the app/build.gradle file.

implementation 'com.amazonaws:aws-android-sdk-lex:2.3.8@aar'
Set IAM Permissions for Amazon Lex
To use Amazon Lex in an application, create a role and attach policies as described in Step 1 of Getting Started in the Lex Developer Guide.

To learn more about IAM policies, see Using IAM.

Configure a Bot
Use the Amazon Lex console console to configure a bot that interacts with your mobile app features. To learn more, see Amazon Lex Developer Guide. For a quickstart, see Getting Started .

Amazon Lex also supports model building APIs, which allow creation of bots, intents, and slots at runtime. This SDK does not currently offer additional support for interacting with Amazon Lex model building APIs.

Implement Text and Voice Interaction with Amazon Lex
Get AWS User Credentials
Both text and voice API calls require validated AWS credentials. To establish Amazon Cognito as the credentials provider, include the following code in the function where you initialize your Amazon Lex interaction objects.

CognitoCredentialsProvider credentialsProvider = new CognitoCredentialsProvider(
            appContext.getResources().getString(R.string.identity_id_test),
            Regions.fromName(appContext.getResources().getString(R.string.aws_region)));
Integrate Lex Interaction Client
Perform the following tasks to implement interaction with Lex in your Android app.

Initialize Your Lex Interaction Client
Instantiate an InteractionClient, providing the following parameters.

The application context, credentials provider, and AWS Region

bot_name - name of the bot as it appears in the Amazon Lex console

bot_alias - the name associated with selected version of your bot

InteractionListener - your app's receiver for text responses from Amazon Lex

AudioPlaybackListener - your app's receiver for voice responses from Amazon Lex

// Create Lex interaction client.
    lexInteractionClient = new InteractionClient(getApplicationContext(),
            credentialsProvider,
            Regions.US_EAST_1,
            <your_bot_name>,
            <your_bot_alias>);
    lexInteractionClient.setAudioPlaybackListener(audioPlaybackListener);
    lexInteractionClient.setInteractionListener(interactionListener);
Begin or Continue a Conversation
To begin a new conversation with Amazon Lex, we recommend that you clear any history of previous text interactions, and that you maintain a inConversation flag to make your app aware of when a conversation is in progress.

If inConversation is false when user input is ready to be sent as Amazon Lex input, then make a call using the textInForTextOut, textInForAudioOut, audioInForTextOut, or audioInForAudioOut method of an InteractionClient instance. These calls are in the form of:

lexInteractionClient.textInForTextOut(String text, Map<String, String> sessionAttributes)
If inConversation is true, then the input should be passed to an instance of LexServiceContinuation using the continueWithTextInForTextOut, continueWithTextInForAudioOut, continueWithAudioInForTextOut, continueWithAudioInForAudioOut method. Continuation enables Amazon Lex to persist the state and metadata of an ongoing conversation across multiple interactions.

Interaction Response Events
InteractionListener captures a set of Amazon Lex response events that include:

onReadyForFulfillment(final Response response)

This response means that Lex has the information it needs to co fulfill the intent of the user and considers the transaction complete. Typically, your app would set your inConversation flag to false when this response arrives.

promptUserToRespond(final Response response, final LexServiceContinuation continuation)

This response means that Amazon Lex is providing the next piece of information needed in the conversation flow. Typically your app would pass the received continuation on to your Amazon Lex client.

onInteractionError(final Response response, final Exception e)

This response means that Amazon Lex is providing an identifier for the exception that has occured.

Microphone Events
MicrophoneListener captures events related to the microphone used for interaction with Amazon Lex that include:

startedRecording()

This event occurs when the user has started recording their voice input to Amazon Lex.

onRecordingEnd()

This event occurs when the user has finished recording their voice input to Amazon Lex.

onSoundLevelChanged(double soundLevel)

This event occurs when the volume level of audio being recorded changes.

onMicrophoneError(Exception e)

The event returns an exception when an error occurs while recording sound through the microphone.

Audio Playback Events
AudioPlaybackListener captures a set of events related to Amazon Lex voice responses that include:

onAudioPlaybackStarted()

This event occurs when playback of a Amazon Lex voice response starts.

onAudioPlayBackCompleted()

This event occurs when playback of a Amazon Lex voice response finishes.

onAudioPlaybackError(Exception e)

This event returns an exception when an error occurs duringplayback of an Amazon Lex voice response.

Add Voice Interactons
Perform the following tasks to implement voice interaction with Amazon Lex in your Android app.

InteractiveVoiceView simplifies the acts of receiving and playing voice responses from Lex by internally using the InteractionClient methods and both MicrophoneListener and AudioPlaybackListener events described in the preceding sections. You can use those interfaces directly instead of instantiating InteractiveVoiceView.

Add a voice-component Layout Element to Your Activity
In the layout for your activity class that contains the voice interface for your app, include the following element.

<include
   android:id="@+id/voiceInterface"
   layout="@layout/voice_component"
   android:layout_width="200dp"
   android:layout_height="200dp"
    />
Initialize Your Voice Activity
In your activity class that contains the voice interface for your app, have the base class implement InteractiveVoiceView.InteractiveVoiceListener.

The following code shows initialization of InteractiveVoiceView.

private void init() {
    appContext = getApplicationContext();
    voiceView = (InteractiveVoiceView) findViewById(R.id.voiceInterface);
    voiceView.setInteractiveVoiceListener(this);
    CognitoCredentialsProvider credentialsProvider = new CognitoCredentialsProvider(
        <your_conginto_identity_pool_id>,
        Regions.fromName(<your_aws_region>)));
    voiceView.getViewAdapter().setCredentialProvider(credentialsProvider);
    voiceView.getViewAdapter().setInteractionConfig(
        new InteractionConfig(<your_bot_name>),
            <your_bot_alias>));
    voiceView.getViewAdapter().setAwsRegion(<your_aws_region>));
}