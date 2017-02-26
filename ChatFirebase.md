## Week 3 Challenge - Chat client with Firebase

- [Lab Overview and Hints](https://) (will capture a short video introduce Firebase here)

In this exercise you will build a chat client using [Firebase](https://console.firebase.google.com/). We'll explore how to work with schema, creating and querying objects, and user authentication.

  - In this lab, we will provide you the Firebase config file to use. If later (not today) you want to create your own project, go to the [Firebase](https://www.firebase.com/signup/), create an account, and then create a new Firebase project and follow the instruction there.

At the end of the exercise your app should look something like this:

![Chat|250](http://i.imgur.com/xhiCRdml.png)

### Getting Started

The checkpoints below should be implemented as pairs. In pair programming, there are two roles: supervisor and driver.

The supervisor makes the decision on what step to do next. Their job is to describe the step using high level language ("Let's print out something when the user is scrolling"). They also have a browser open in case they need to do any research. The driver is typing and their role is to translate the high level task into code ("Set the scroll view delegate, implement the didScroll method").

After you finish each checkpoint, switch the supervisor and driver roles. The person on the right will be the first supervisor.

### Milestone 1: Setup
  - Create a new project. Run `pod init` to create a new Podfile.
  - Add the Firebase pods to your project with: 
  
  ```
  pod 'Firebase/Storage'
  pod 'Firebase/Auth'
  pod 'Firebase/Database'
  ```
  
  - Make sure to [enable Swift support](http://guides.codepath.com/ios/CocoaPods#swift-support) by adding the `use_frameworks!` directive to your `Podfile`.
  - Add [config file](https://github.com/avo1/ChatTutorial/blob/master/GoogleService-Info.plist) to your project
  
  ![Import google plist](http://i.imgur.com/9m3sBNp.png)
  
  - Configure Firebase in AppDelegate
  
```swift
import UIKit
import Firebase

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

  var window: UIWindow?

  func application(application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?)
    -> Bool {
    FIRApp.configure()
    return true
  }
}
```
   - In any Swift file that you're using Firebase, add `import Firebase` to the top of the file.

### Milestone 2: Create a Login Screen
  - Create a new View Controller (or rename the default one) called `LoginViewController`.
  - `Import Firebase` to `the LoginViewController`
  - Firebase provides multiple sign-in method such as email/password, Google, Facebook,... but we are going to use the easiest one: `Anonymous`. We already enabled `Anonymous` mode. (To set up anonymous authentication, open the **Firebase App Dashboard** -> **Authentication** -> **Sign-In Method**, then enable **Anonymous** and Save.)
  - Add the following views to the login screen:
    - An username text field 
    - A log in button
  - On log in action, attempt to `signInAnnonymously`
  
```swift
        FIRAuth.auth()!.signInAnonymously(completion: { (user, error) in
            if error == nil {
                // login successfully, move to chatVC
            } else {
                // login failed, display an alert
            }
        })
```
  - Display an [alert](https://guides.codepath.com/ios/Using-UIAlertController) on error.

### Milestone 3: Send a Chat Message
  - Create a new View Controller (`ChatViewController`) for the chat room.
  - After a successful log in from the `LoginViewController`, modally present the `ChatViewController`.
    - The `ChatViewController` should have a title `Chat` and should be inside a [navigation controller](http://guides.codepath.com/ios/Navigation-Controller-Quickstart).
  - At the top of the layout, add a text field and a button to compose a new message. Make sure to set up your [auto layout](http://guides.codepath.com/ios/Auto-Layout-Basics) constraints.
  
  ![ChatVC](http://i.imgur.com/GrX8Yjb.png)
  
  - When the user taps the button, create a new message in Firebase:
     - On `ChatViewController`: Declare a `messageRef` to store a reference to the list of messages in the database. Use the class name: `messages0217` (this is case sensitive). Read more about [lazy](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html).
     
     ```swift
     lazy var messageRef: FIRDatabaseReference = FIRDatabase.database().reference().child("messages0217")
     ```
     
     - On action for the send button: Store the message item with 3 fields (sender, content, createdAt).
     
     ```swift
     @IBAction func onSend(_ sender: UIButton) {
        if let msg = textField?.text {
            let newMessageRef = messageRef.childByAutoId()
            let messageItem: [String: Any] = [
                "sender": senderName,
                "content": msg,
                "createdAt": [".sv": "timestamp"]
            ]
            newMessageRef.setValue(messageItem)
            textField.text = ""
        }
    }
    ```
    
     - The `[".sv": "timestamp"]` will save the time since the [Unix epoch](https://en.wikipedia.org/wiki/Unix_time) in milliseconds.

### Milestone 4: View the Chat Room
  - Display the messages in a TableView:
    - Add a tableView to the `ChatViewController` and a custom cell that will contain each message.
    - For now, the cell will only contain a UILabel (multi-line) for the message.
    - You'll want to use [auto layout](http://guides.codepath.com/ios/Auto-Layout-Basics) and [automatically sizing rows](http://guides.codepath.com/ios/Table-View-Quickstart#automatically-resize-row-heights) in the tableView.
  - Pull down all the messages from Firebase: Firebase automatically trigger 

    - [Query Parse](https://parse.com/docs/ios/guide#queries) for all messages using the `Message_Swift_102016` class.
    - You can [sort the results](https://parse.com/docs/ios/guide#query-constraints) in descending order with the `createdAt` field.
    - Once you have a successful response, save the result in an NSArray and [reload](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UITableView_Class/#//apple_ref/occ/instm/UITableView/reloadData) the tableView data.

### Milestone 5: Associating Users with Messages
  - When creating a new message, add a key called `user` and set it to `PFUser.current()`
  - Modify the custom message cell to display the [username](https://parse.com/docs/ios/guide#users) (if it exists).
    - You might want to check out [UIStackView](https://developer.apple.com/library/prerelease/ios/documentation/UIKit/Reference/UIStackView_Class_Reference) (iOS9+) for an easy way to hide views in your layout (for the case when there is no username).
    - Toggling a view's [hidden](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/#//apple_ref/occ/instp/UIView/hidden) property will add or remove it from its contained UIStackView.
  - When querying for messages, use the method `includeKey("user")` to instruct Parse to [fetch the related user](https://parse.com/docs/ios/guide#relational-queries).

### Milestone 6: Add "Log out" button. Go to Chat View Controller directly if not yet logged out.

Parse stores the current user in the session. Take advantage of this to skip the login/signup page if the user is already logged in.
Add Logout bar button item to the navigation bar of ChatViewController so that you can log in again as a different user.

### Bonus 1: Link to a Facebook Account

Check out Parse instructions to include Facebook SDK with Parse [here](https://parse.com/docs/ios/guide#facebook-users).
Add a SettingsViewController to let a user link to his or her own Facebook account and retrieve that person's profile picture. Display profile pictures on chat.


<% if @cohort['language'] == 'objc' %>  
### Bonus 1: Link a Facebook Account
  - [Add the pods](http://guides.codepath.com/ios/CocoaPods#adding-a-pod) `'FBSDKCoreKit'` and `'ParseFacebookUtilsV4'` to authenticate with Facebook.
  - Configure the Facebook SDK:  
    - Modify your `info.plist` to include the keys outlined [in step 5](https://developers.facebook.com/docs/ios/getting-started#xcode):
      - `Facebook App Id` = 1381230028782707
      - `Display Name` = com.codepath.codepath
    - Make sure to also follow the steps to whitelist Facebook domains for [App Transport Security](https://developers.facebook.com/docs/ios/getting-started#xcode).
    - Also add the `fbauth2` url scheme to your `info.plist` as outlined in this [stackoverflow post](http://stackoverflow.com/a/32006111).
  - Add the following code to your `AppDelegate`:
      ```
      // AppDelegate.m

      #import <FBSDKCoreKit/FBSDKCoreKit.h>
      #import <ParseFacebookUtilsV4/ParseFacebookUtilsV4.h>

      // ...

      - (BOOL)application:(UIApplication *)application
      didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

          // ...          

          [PFFacebookUtils initializeFacebookWithApplicationLaunchOptions:launchOptions];
          [[FBSDKApplicationDelegate sharedInstance] application:application
                                   didFinishLaunchingWithOptions:launchOptions];

		  // ...
      }

      - (BOOL)application:(UIApplication *)application
                  openURL:(NSURL *)url
        sourceApplication:(NSString *)sourceApplication
               annotation:(id)annotation {
          return [[FBSDKApplicationDelegate sharedInstance] application:application
                                                                openURL:url
                                                      sourceApplication:sourceApplication
                                                             annotation:annotation];
      }

      // ...
      ```
  - Create a new `SettingsViewController` to allow the user to [connect his/her Facebook account](https://parse.com/docs/ios/guide#users-linking):
    - You'll need to add: `#import <ParseFacebookUtilsV4/ParseFacebookUtilsV4.h>`
    - Add a left bar button item on the ChatViewController to open the settings screen.
    - If the user is not yet linked, show a "Link Facebook Account" button.
    - If the account is already linked, display an unlink button.
    - Add a "Done" button to close the `SettingsViewController`.
<% end %>
