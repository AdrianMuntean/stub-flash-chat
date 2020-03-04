# Flash Chat ⚡️

## Goal

We are going to learn how to incorporate Firebase with our Flutter app. We'll be using Firebase Cloud Firestore as well as the Firebase authentication package to equip our app with a cloud-based NoSQL database and secure authentication methods

## What we will create

We’re going to build a modern messaging app where users can sign up and log in to chat.

![image](https://user-images.githubusercontent.com/12469787/75607598-8274a980-5b01-11ea-9fc9-e8d9c033223e.png)

## Steps
### 1. Update the routing options
```dart
    Widget build(BuildContext context) {
        return MaterialApp(
          initialRoute: WelcomeScreen.routeName,
          routes: {
            WelcomeScreen.routeName: (context) => WelcomeScreen(),
            ChatScreen.routeName: (context) => ChatScreen(),
            LoginScreen.routeName: (context) => LoginScreen(),
            RegistrationScreen.routeName: (context) => RegistrationScreen()
          },
        );
      }
```

With defined routes:
```dart
static const String routeName = 'welcomeScreen';
```

### 2. Change screens on button press
We have multiple options: 

Named routing:
```dart
Navigator.pushNamed(context, RegistrationScreen.routeName)
```

Navigate to a specifc screen:
```dart
Navigator.push(
   context,
   MaterialPageRoute(builder: (context) => LoginScreen()),
)
```

### 3. Adding some animations
  1. Hero animations (or _shared element transition_)
  Flutter makes it easy to work with [hero animations](https://flutter.dev/docs/development/ui/animations/hero-animations). 
  
  How to implement hero widgets? 
   - 2 Hero widgets
   - a shared tag property
   - navigator based screen transitions
   
  `welcome_screen.dart`
   ```dart
    Hero(
      tag: 'logo', // the tag must be the same
       child: Container(
        child: Image.asset('images/logo.png'),
        height:  60,
      ),
    ),
   ```
   
   `registration_screen.dart`
   ```dart
    Hero(
       tag: 'logo',
        child: Container(
         height: 200.0,
         child: Image.asset('images/logo.png'),
        ),
      ),
   ```
   
   2. Custom animations with an animation controller
    e.g.  - Change the background
          - Change the position of an element
         
        What we need:
         1. A ticker
         2. An animation controller
         3. An animation value
         
`welcome_screen.dart` 
     
```dart
    class _WelcomeScreenState extends State<WelcomeScreen>
        with SingleTickerProviderStateMixin {
    AnimationController controller;
    Animation colorAnimation;
    
    @override
      void initState() {
        super.initState();
    
        controller = AnimationController(
          duration: Duration(seconds: 2),
          vsync: this,
        );
    
        controller.forward();
    
        controller.addListener(() {
          setState(() {});
        });
    
        animation = CurvedAnimation(parent: controller, curve: Curves.bounceOut);
        colorAnimation = ColorTween(begin: Colors.white, end: Colors.lightBlue[100])
            .animate(controller);
      }
```
    
  Don't forget to dispose the animation controller
  ```dart
@override
  void dispose() {
    super.dispose();
    controller.dispose();
  }
```
    
  Then update the background color in the build method
  
  ```dart
    backgroundColor: colorAnimation.value
  ``` 
  and the height of the logo container widget
  
  ```dart
   height: animation.value * 60,
  ``` 
  
  You can read more about animations on the [this Flutter official doc](https://flutter.dev/docs/development/ui/animations/tutorial).
  
  More about [curves animations](https://api.flutter.dev/flutter/animation/Curves-class.html). 
   
   3. Prepackaged animations
  There are a lot of ready to use packages for animations. [A simple search](https://pub.dev/packages?q=animate) yields over 1200 packages.
  
  We are going to use [animated text kit](https://pub.dev/packages/animated_text_kit)
  
  After installation:
  
  In `welcome_screen.dart` replace the `Text` widget:
  ```dart
TypewriterAnimatedTextKit(
                  speed: Duration(milliseconds: 100),
                  text: ['Flash Chat'],
                  totalRepeatCount: 2,
                  textStyle: TextStyle(
                    fontSize: 45.0,
                    fontWeight: FontWeight.w900,
                  ),
                ),
```

### 4. Refactor

The padding widget is repeated code. We need to extract it in a separate file.

Add a new file in the `components` directory and extract the button. 

```dart
import 'package:flutter/material.dart';

class LongButton extends StatelessWidget {
  @required
  final String text;
  @required
  final Function onTap;
  final Color color;

  LongButton({this.text, this.onTap, this.color});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.symmetric(vertical: 16.0),
      child: Material(
        elevation: 5.0,
        color: this.color,
        borderRadius: BorderRadius.circular(30.0),
        child: MaterialButton(
          onPressed: () {
            this.onTap();
          },
          minWidth: 200.0,
          height: 42.0,
          child: Text(
            this.text,
            style: TextStyle(color: Colors.white),
          ),
        ),
      ),
    );
  }
}

```
  
And replace the occurrences of the button in `welcome_screen`, `login_screen` and `registration_screen`;


### 5. Create the Firebase project

Head over to [Firebase](https://firebase.google.com/).

Login with your Google account and create a new project (or add Firebase to an existing one).
![image](https://user-images.githubusercontent.com/12469787/75851642-e396c880-5df2-11ea-9ef7-84cb00edbf65.png)

### 6. Android Firebase setup

![image](https://user-images.githubusercontent.com/12469787/75851697-0fb24980-5df3-11ea-8b57-b5ddc9089c52.png)

**Android package name**: copy from `/android/app/build.gradle` `applicationId`

__You can leave the rest unfilled__

:warning: It's really important the package name you copy to Firebase to match the application id from `build.gradle`

Download the `google-services.json` file and copy it in the `android/app` folder

Copy all the dependencies in `build.gradle`.

:warning: Beware of the difference between project-level `build.gradle` and app-level `build.gradle`

Run an android simulator and if no error is displayed that this means it's all good!

### 7. iOS Firebase setup

You need to run on MacOS with XCode already installed. 

![image](https://user-images.githubusercontent.com/12469787/75852382-856ae500-5df4-11ea-9f4f-a1c75332a96e.png)

**iOS bundle ID**: open `Runner.xcodeproj` in XCode and you'll find the bundle ID in the top level `Runner` menu:

![image](https://user-images.githubusercontent.com/12469787/75852618-1477fd00-5df5-11ea-8444-8cc50ca99830.png)

__You can leave the rest unfilled__

Download the `GoogleService-Info.plist` file and copy it in the `Runner` project directly in the XCode.

Run an iOS simulator and if no error is displayed that this means it's all good!

### 8. Firebase Flutter packages setup

There are a lot of Firebase services we can use and for each of them there is a [Flutter plugin](https://github.com/FirebaseExtended/flutterfire).

We are only using: 
    - [firebase_core](https://pub.dev/packages/firebase_core)
    - [cloud_firestore](https://pub.dev/packages/cloud_firestore)
    - [firebase_auth](https://pub.dev/packages/firebase_auth)
    
After installing test on the emulator from cold start. 

 ## iOS Specific only
 
 You might want to update the cocoa pods so be on the safe side:
 ```
   pod repo update
   sudo gem install cocoapods
   pod setup
 ```
### 9. Registering users with FirebaseAuth

Open `registration_screen.dart` and add 2 new fields in the state class:
```dart
String email;
String password;
```

Update the `TextField` widget to set the email and password on changed:

```dart
TextField(
     onChanged: (value) {
       email = value;
     }
```

__Optional:__ Center align the `TextField`: `textAlign: TextAlign.center,`
__Optional:__ Change the keyboard type for the email `TextField`: `keyboardType: TextInputType.emailAddress,`

On password field hide the text `obscureText: true,`.

Import the Firebase auth package:
```dart
import 'package:firebase_auth/firebase_auth.dart';
```
And add a new private property 
```dart
final _auth = FirebaseAuth.instance;
```

In the `register button` register the user:
```dart
onPressed: () async {
                  try {
                    final newUser = await _auth.createUserWithEmailAndPassword(
                        email: email, password: password);
                    if (newUser != null) {
                      Navigator.pushNamed(context, ChatScreen.routeName);
                    }
                  } catch (e) {
                    print(e);
                  }
                }
```

If the user creation step was successful we will push the `chat_screen` route.

In the `chat_screen.dart` file import the auth package and create a new final variable:

```dart
import 'package:firebase_auth/firebase_auth.dart';
...
final _auth = FirebaseAuth.instance;
FirebaseUser loggedInUser;
```

Create a method for checking if a user has successfully signed in:
```dart
void getCurrentUser() async {
    try {
      final user = await _auth.currentUser();
      if (user != null) {
        loggedInUser = user;
        print(loggedInUser.email);
      }
    } catch (e) {
      print(e);
    }
  }
```

Before testing the functionality we should enable login in Firebase UI.
![image](https://user-images.githubusercontent.com/12469787/75854627-5440e380-5df9-11ea-87b1-3a2fee9a16bc.png)

### 10. Authenticating users with FirebaseAuth

Similar with registering new users. 

Steps:
  1. Open `login_screen.dart`
  2. Add email and password fields 
  3. Set email and password on text field change
  4. Add the optional steps for improving the `TextFields`  
  4. Use `signInWithEmailAndPassword` method from auth package
  
```dart
    onTap: () async {
        try {
            final authResut = await _auth.signInWithEmailAndPassword(
                email: email, password: password);
            if (authResut != null) {
                Navigator.pushNamed(context, ChatScreen.routeName);
            }
        } catch (e) {
            print(e);
        }
    },
```

In order to logout update the icon button `X` button from the `chat_screen.dart`:

```dart
onPressed: () {
                _auth.signOut();
                Navigator.pop(context);
              }),
```


### 11. Showing a spinner

For showing the user some feedback, like a loading indicator we can use a pre-build package called [modal_progress_hud](https://pub.dev/packages/modal_progress_hud).

Inside `registration_screen.dart` and `login_screen.dart` create a new boolean field:
```dart
bool showSpinner = false;
```

Set `showSpinner = true` before creating/logging in the user and `false` after. 

And then wrap everything inside the body of the scaffold inside a `ModalProgressHUD` and use the boolean flag to display the modal.

### 12. Saving data into Cloud Firestore

Before we can save data to first need to setup the database.

![image](https://user-images.githubusercontent.com/12469787/75855613-56a43d00-5dfb-11ea-9db0-7e7fd22fdd37.png)

Create a collection called `messages`.

![image](https://user-images.githubusercontent.com/12469787/75855890-e0540a80-5dfb-11ea-965c-e1a27f1b5190.png)

Open `chat_screen.dart` and add a new field called `messageText`.

Import the `firestore` package and create a final firestore instance:

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

...
final _fireStore = Firestore.instance;
```

On pressing the `Send` button use the `fireStore` instance to add the message to the database.

```dart
onPressed: () {
                      _fireStore.collection('messages').add({
                        'text': messageText,
                        'sender': loggedInUser.email,
                        'timestamp': DateTime.now().millisecondsSinceEpoch
                      });
                    }
```

### 13. Listening for Data from Firebase using streams and turn streams into Widgets

We can test the retrieving of the messages with a simple method. Add the following code in the `chat_screen.dart`:

```dart
  void getMessages() async {
    final messages = await _fireStore.collection('messages').getDocuments();
    messages.documents.forEach((message) => print(message.data));
  }
```

Call this method from the log out button and we should see the printed messages in the console. 

Add a new class (in a separate file or in the same file) called `MessageStream`:

```dart
class MessageStream extends StatelessWidget {
  const MessageStream({
    @required Firestore fireStore,
  }) : _fireStore = fireStore;

  final Firestore _fireStore;

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<QuerySnapshot>(
      stream: _fireStore
          .collection('messages')
          .orderBy('timestamp', descending: true)
          .snapshots(),
      builder: (context, snapshot) {
        if (snapshot.hasData) {
          final messages = snapshot.data.documents;
          List<Text> messageWidgets = [];
          for (var message in messages) {
            final messageText = message.data['text'];
            final messageSender = message.data['sender'];

            final currentUser = loggedInUser.email;
            final messageWidget = Text(
              '$messageText from $messageSender',
              style: TextStyle(
                fontSize: 50.0,
              ),
            );
            messageWidgets.add(messageWidget);
          }
          return Expanded( // so the ListView takes only the space available
            child: ListView(
              reverse: true,
              padding: EdgeInsets.symmetric(horizontal: 10, vertical: 20),
              children: <Widget>[...messageWidgets],
            ),
          );
        } else {
          return Center(
            child: CircularProgressIndicator(),
          );
        }
      },
    );
  }
}
```
You can read more about [ListView](https://api.flutter.dev/flutter/widgets/ListView-class.html).


Use this `MessageStream` right above the `Container` as the first children of the `Column` widget:
```dart
children: <Widget>[
            MessageStream(fireStore: _fireStore),
```

### Use a custom widget for message bubble

Add a new class (in the same or a new file) called `MessageBubble`:
```dart
class MessageBubble extends StatelessWidget {
  final String sender;
  final String text;

  const MessageBubble({this.sender, this.text});

  @override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.all(10),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.end,
        children: <Widget>[
          Text(
            sender,
            style: TextStyle(fontSize: 12, color: Colors.black54),
          ),
          Material(
            elevation: 5.0,
            color: Colors.lightBlue,
            textStyle: TextStyle(),
            child: Padding(
              padding: EdgeInsets.symmetric(vertical: 10, horizontal: 20),
              child: Text(
                this.text,
                style: TextStyle(fontSize: 20),
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

And instead of creating simple `Text` widgets inside the `MessageStream` create a `MessageBubble`:

```dart
List<MessageBubble> messageWidgets = [];
          for (var message in messages) {
            final messageText = message.data['text'];
            final messageSender = message.data['sender'];

            final currentUser = loggedInUser.email;
            final messageWidget = MessageBubble(
              text: messageText,
              sender: messageSender,
            );
            messageWidgets.add(messageWidget);
```

Finally, we want to clear the text when we push the `Send` button. For this create a `messageTextController` insde the `chat_screen.dart` file.

```dart
final messageTextController = TextEditingController();
```

And use it inside the `TextField`:

```dart
TextField(
        controller: messageTextController,
```

And clear the text on pressing the button:

```dart
onPressed: () {
        _fireStore.collection('messages').add({
        'text': messageText,
                        'sender': loggedInUser.email,
                        'timestamp': DateTime.now().millisecondsSinceEpoch
                      });
                      messageTextController.clear();
                    },
```

### 14. Different UI for different senders

Wrap the `Hero` widget from the `registration_screen.dart` and `login_screen.dart` inside a `Flexible` widget. 

Move the `FirebaseUser` constant outside of the `ChatScreen` class so we can access it from the `MessageStream` class. 

Inside the `MessageStream` we can determine if the logged in user is the one sending the message and have a boolean flag sent over to the `MessageBuble`:

```dart
final currentUser = loggedInUser.email;
final messageWidget = MessageBubble(
              text: messageText,
              sender: messageSender,
              isMe: currentUser == messageSender,
            );
```

And we need to update the `MessageBubble` constructor with the new flag.

```dart
final bool isMe;

const MessageBubble({this.sender, this.text, this.isMe});
```

Use this flag to have different UIs:

```dart
@override
  Widget build(BuildContext context) {
    return Padding(
      padding: EdgeInsets.all(10),
      child: Column(
        crossAxisAlignment:
            isMe ? CrossAxisAlignment.end : CrossAxisAlignment.start,
        children: <Widget>[
          Text(
            this.sender,
            style: TextStyle(fontSize: 12, color: Colors.black54),
          ),
          Material(
            elevation: 5.0,
            borderRadius: isMe
                ? BorderRadius.only(
                    topLeft: Radius.circular(30),
                    bottomLeft: Radius.circular(30),
                    bottomRight: Radius.circular(30))
                : BorderRadius.only(
                    topRight: Radius.circular(30),
                    bottomLeft: Radius.circular(30),
                    bottomRight: Radius.circular(30)),
            color: isMe ? Colors.lightBlue : Color(0xFFF7B914),
            textStyle: TextStyle(),
            child: Padding(
              padding: EdgeInsets.symmetric(vertical: 10, horizontal: 20),
              child: Text(
                this.text,
                style: TextStyle(fontSize: 20),
              ),
            ),
          ),
        ],
      ),
    );
  }
```

### 15. Cloud Firestore security rules
    
In the last step we need to allow only authorized users to write to the database.

Go to the Firestore rules and change the rules as following:

```
match /{document=**} {
      allow read, write: if request.auth.uid != null;
    }
``` 

You can test the rules using the simulator.
    
    
## Finished project
In case you are stuck you can find the final app [here](https://github.com/AdrianMuntean/flash-chat)


Special thanks to [www.appbrewery.co](https://www.appbrewery.co/) from where the course was inspired. 
