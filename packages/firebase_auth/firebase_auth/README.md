# firebase_auth plugin
A Flutter plugin to use the [Firebase Authentication API](https://firebase.google.com/products/auth/).

[![pub package](https://img.shields.io/pub/v/firebase_auth.svg)](https://pub.dartlang.org/packages/firebase_auth)

For Flutter plugins for other Firebase products, see [README.md](https://github.com/FirebaseExtended/flutterfire/blob/master/README.md).

## Usage

### Configure the Google sign-in plugin
The Google Sign-in plugin is required to use the firebase_auth plugin for Google authentication. Follow the [Google sign-in plugin installation instructions](https://pub.dartlang.org/packages/google_sign_in#pub-pkg-tab-installing).

If you're using Google Sign-in with Firebase auth, be sure to include all required fields in the [OAuth consent screen](https://console.developers.google.com/apis/credentials/consent). If you don't, you may encounter an `ApiException`.

### Import the firebase_auth plugin
To use the firebase_auth plugin, follow the [plugin installation instructions](https://pub.dartlang.org/packages/firebase_auth#pub-pkg-tab-installing).

### Android integration

Enable the Google services by configuring the Gradle scripts as such.

1. Add the classpath to the `[project]/android/build.gradle` file.
```gradle
dependencies {
  // Example existing classpath
  classpath 'com.android.tools.build:gradle:3.2.1'
  // Add the google services classpath
  classpath 'com.google.gms:google-services:4.3.0'
}
```

2. Add the apply plugin to the `[project]/android/app/build.gradle` file.
```gradle
// ADD THIS AT THE BOTTOM
apply plugin: 'com.google.gms.google-services'
```

*Note:* If this section is not completed you will get an error like this:
```
java.lang.IllegalStateException:
Default FirebaseApp is not initialized in this process [package name].
Make sure to call FirebaseApp.initializeApp(Context) first.
```

*Note:* When you are debugging on android, use a device or AVD with Google Play services.
Otherwise you will not be able to authenticate.

### Use the plugin

Add the following imports to your Dart code:
```dart
import 'package:firebase_auth/firebase_auth.dart';
```

Initialize `GoogleSignIn` and `FirebaseAuth`:
```dart
final GoogleSignIn _googleSignIn = GoogleSignIn();
final FirebaseAuth _auth = FirebaseAuth.instance;
```

You can now use the Firebase `_auth` to authenticate in your Dart code, e.g.
```dart
Future<FirebaseUser> _handleSignIn() async {
  final GoogleSignInAccount googleUser = await _googleSignIn.signIn();
  final GoogleSignInAuthentication googleAuth = await googleUser.authentication;

  final AuthCredential credential = GoogleAuthProvider.getCredential(
    accessToken: googleAuth.accessToken,
    idToken: googleAuth.idToken,
  );

  final FirebaseUser user = (await _auth.signInWithCredential(credential)).user;
  print("signed in " + user.displayName);
  return user;
}
```

Then from the sign in button onPress, call the `_handleSignIn` method using a future
callback for both the `FirebaseUser` and possible exception.
```dart
_handleSignIn()
    .then((FirebaseUser user) => print(user))
    .catchError((e) => print(e));
```

### Register a user

```dart
final FirebaseUser user = (await _auth.createUserWithEmailAndPassword(
      email: 'an email',
      password: 'a password',
    ))
        .user;
```

### Error handling

If a method fails it will throw a `PlatformException` with the error code as a string.
So make sure to wrap your calls with a `try..catch` or add a `.catchError` functions.
Check the source comments to see which error codes each method could throw.

### Supported Firebase authentication methods

* Google
* Email and Password
* Phone
* Anonymously
* GitHub
* Facebook
* Twitter

### Phone Auth

You can use Firebase Authentication to sign in a user by sending an SMS message to
the user's phone. The user signs in using a one-time code contained in the SMS message.

### After authentication

After a successful authentication, you will receive a `FirebaseUser` object. You can use this object to check if the email is verified, to update email, to send verification email and so on. See the [FirebaseUser](https://pub.dartlang.org/documentation/firebase_auth/latest/firebase_auth/FirebaseUser-class.html) API documentation for more details on the `FirebaseUser` object.


#### iOS setup

1. Enable Phone as a Sign-In method in the [Firebase console](https://console.firebase.google.com/u/0/project/_/authentication/providers)

  - When testing you can add test phone numbers and verification codes to the Firebase console.

1. [Enable App verification](https://firebase.google.com/docs/auth/ios/phone-auth#enable-app-verification)  

**Note:** App verification may use APNs, if using a simulator (where APNs does not work) or APNs is not setup on the
device you are using you must set the `URL Schemes` to the `REVERSE_CLIENT_ID` from the GoogleServices-Info.plist file.

#### Android setup

1. Enable Phone as a Sign-In method in the [Firebase console](https://console.firebase.google.com/u/0/project/_/authentication/providers)

  - When testing you can add test phone numbers and verification codes to the Firebase console.

### Web integration

In addition to the `firebase_auth` dependency, you'll need to modify the `web/index.html` of your app following the Firebase setup instructions:

* [Add Firebase to your JavaScript project](https://firebase.google.com/docs/web/setup#from-the-cdn).

Read more in the [`firebase_auth_web` README](https://github.com/FirebaseExtended/flutterfire/blob/master/packages/firebase_auth/firebase_auth_web/README.md).

## Example

See the [example application](https://github.com/FirebaseExtended/flutterfire/tree/master/packages/firebase_auth/firebase_auth/example) source
for a complete sample app using the Firebase authentication.

## Issues and feedback

Please file Flutterfire specific issues, bugs, or feature requests in our [issue tracker](https://github.com/FirebaseExtended/flutterfire/issues/new).

Plugin issues that are not specific to Flutterfire can be filed in the [Flutter issue tracker](https://github.com/flutter/flutter/issues/new).

To contribute a change to this plugin,
please review our [contribution guide](https://github.com/FirebaseExtended/flutterfire/blob/master/CONTRIBUTING.md),
and send a [pull request](https://github.com/FirebaseExtended/flutterfire/pulls).

## 0.4.x to 0.5.x Firebase Dynamic Links Android migration tip

When upgrading from an older version of the Dynamic Links library, passwordless (email link) sign-in will break, and the link will return null if you simply replace `FirebaseDynamicLinks.instance.retrieveDynamicLink()` with `FirebaseDynamicLinks.instance.getInitialLink()`. Instead you need to update your code to something similar to 

```
@override
  void didChangeAppLifecycleState(AppLifecycleState state) async {
    if (state == AppLifecycleState.resumed) {
      final PendingDynamicLinkData data =
      await FirebaseDynamicLinks.instance.getInitialLink();
      if( data?.link != null ) {
        handleLink(data?.link);
      }

      FirebaseDynamicLinks.instance.onLink(
          onSuccess: (PendingDynamicLinkData dynamicLink) async {
            final Uri deepLink = dynamicLink?.link;

            handleLink(deepLink);
          }, onError: (OnLinkErrorException e) async {
        print('onLinkError');
        print(e.message);
      });
    }
  }

  void handleLink(Uri link) async {
    if (link != null) {
      final FirebaseUser user = (await _auth.signInWithEmailAndLink(
        email: _userEmail,
        link: link.toString(),
      ))
          .user;

      if (user != null) {
        _userID = user.uid;
        _success = true;
      } else {
        _success = false;
      }
    } else {
      _success = false;
    }

    setState(() {});
  }
```

More context can be found in this [issue discussion](https://github.com/FirebaseExtended/flutterfire/issues/1537)
