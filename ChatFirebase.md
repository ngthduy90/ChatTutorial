## Week 3 Challenge - Chat client with Firebase

In this exercise you will build a chat client using [Firebase](https://console.firebase.google.com/). We'll explore how to work with schema, creating and querying objects, and user authentication.

  - In this lab, we will provide you the Firebase config file to use. If later (not today) you want to create your own project, go to the [Firebase](https://www.firebase.com/signup/), create an account, and then create a new Firebase project and follow the instruction there.

At the end of the exercise your app should look something like this:

![Imgur](http://i.imgur.com/Gkekn5th.png)

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
  - Firebase provides multiple sign-in method such as email/password, Google, Facebook,... We are going to use email/password method. We already enabled `Anonymous` mode.
  - Add the following views to the login screen:
    - An email/password text field 
    - A sign up/log in button
  - On sign up action, attemp to `createUser(withEmail:password:)`
  
```swift
   FIRAuth.auth()!.createUser(withEmail: email, password: pass, completion: { (user: FIRUser?, error: Error?) in
       if error == nil {
           // signup successfully, auto login and move to chatVC
       } else {
           // login failed, display an alert
       }
   })
```
  - On log in action, attempt to `signIn(withEmail:password)`
  
```swift
   FIRAuth.auth()!.signIn(withEmail: email, password: pass) { (user: FIRUser?, error: Error?) in
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
    - The `ChatViewController` should have the logged in email as the title and should be inside a [navigation controller](http://guides.codepath.com/ios/Navigation-Controller-Quickstart).
  - At the top of the layout, add a text field and a button to compose a new message. Make sure to set up your [auto layout](http://guides.codepath.com/ios/Auto-Layout-Basics) constraints.
  
  ![ChatVC](http://i.imgur.com/yb4yPy3.png)
  
  - When the user taps the button, create a new message in Firebase:
     - On `ChatViewController`: Declare a `messageRef` to store a reference to the list of messages in the database. Use the class name: `messages0217` (this is case sensitive). Read more about [lazy](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html).
     
     ```swift
     lazy var messageRef: FIRDatabaseReference = FIRDatabase.database().reference().child("messages0217")
     ```
     
     - On action for the send button: Store the message data with 3 fields (sender, content, createdAt).
     
     ```swift
     @IBAction func onSend(_ sender: UIButton) {
        if let msg = textField?.text {
            let newMessageRef = messageRef.childByAutoId()
            let messageData: [String: Any] = [
                "sender": senderName,
                "content": msg,
                "createdAt": [".sv": "timestamp"]
            ]
            newMessageRef.setValue(messageData)
            textField.text = ""
        }
    }
    ```
    
     - The `[".sv": "timestamp"]` will save the time since the [Unix epoch](https://en.wikipedia.org/wiki/Unix_time) in milliseconds. To retrieve `Date` from timestamp, use `NSDate(timeIntervalSince1970: (messageData["createdAt"] as! Double)/1000)`

### Milestone 4: Load the messages
  - Pull down all the messages from [Firebase](https://firebase.google.com/docs/database/ios/read-and-write): Firebase allows you attach a listener (observer) to the database using the `observeEventType:withBlock` methods of `FIRDatabaseReference` to observe `FIRDataEventTypeValue` events.

    - We have the `messageRef` in step 3, now create a `messageRefHandle` hold a handle to the reference.
    - Declare the messages array of `[String: Any]`
    
    ```swift
    lazy var messageRef: FIRDatabaseReference = FIRDatabase.database().reference().child("messages0217")
    var messageRefHandle: FIRDatabaseHandle?
    
    var messages: [[String: Any]]()
    ```
    
    - Add the observer to monitor if any new message added.
    
    ```swift
    private func observeNewMessages() {
        messageRefHandle = messageRef.observe(.childAdded, with: { (snapshot) in
            let messageData = snapshot.value as! Dictionary<String, Any>
            
            if let name = messageData["sender"] as! String!, name.characters.count > 0 {
                // add this message to the messages
            } else {
                print("Error! Could not decode message data")
            }
            
        })
    }
    ```
    
    - Call the observer in `viewDidLoad`.
  
  - It's a good practice (and is your responsibility) to detach the observer when you leave a ViewController. 
    
    ```swift
    deinit {
        if let refHandle = messageRefHandle {
            messageRef.removeObserver(withHandle: refHandle)
        }
    }
    ```

### Milestone 5: Display message on TableView and logout
  - Display the messages in a TableView:
    - Add a tableView to the `ChatViewController` and a custom cell that will contain each message.
    - The cell will only contain a `senderLabel`, `messageLabel` (multi-line), and `timeLabel`.
    
    ![messageCell](http://i.imgur.com/nnXCHQJ.png)
    
    - You'll want to use [auto layout](http://guides.codepath.com/ios/Auto-Layout-Basics) and [automatically sizing rows](http://guides.codepath.com/ios/Table-View-Quickstart#automatically-resize-row-heights) in the tableView.
  - To calulate the `timeAgo`, install the pod `pod 'NSDate+TimeAgo'`
  - If it's your message and change the display name to `you`, and change the text color.
  
  

### Milestone 6: Go to Chat View Controller directly if not yet logged out
  - Go to ChatViewController directly if already logged in: in LoginViewController's `viewDidLoad`
  
  ```swift
     FIRAuth.auth()!.addStateDidChangeListener { (auth, user) in ...
  ```
  
  - Log out: `try! FIRAuth.auth()!.signOut()`
  
### Bonus: Additional features:
  - Auto scroll tableView to show the latest message. Hint: use `tableView.scrollToRow`
  - Only load 20 most recent messages, pull to fetch next 20.
 
