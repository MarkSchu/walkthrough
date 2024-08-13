# Project Walkthrough

## The Problem 

The **Dispatch Hub** is an an app used for exchanging **audio messages**. 

This docuemnt describes the project of **text message** exchange to the Dispatch Hub.

### Mobile App

Zello makes a mobile app that lets the phone act like a walkie-talking. You can send a voice message to another user by pushing the big button and talking.

<img src="/assets/1-phone-display.png" alt="drawing" width="200"/>

### Users 

The app is used by teams and people that are "out in the field" and need to communicate in real time like truck drivers, taxi services, event staff, warehousing teams, utility companies, etc.  

### Dispatch Hub

The **Dispatch Hub** is a desktop app used by people back at the office, the dispatchers, to coordinate and help the mobiles users out in the field. 

For example, you can imagine a dispatcher working for the power company coordinatng trucks after a power outage. They're there to answer questions, direct trucks to certain areas, and revieve updates when work is done.

### Why Solve the Problem? 
There are contexts in which it is more helpful to send text messages. Here are a few: 

  1. You can visually search through text messages. 
  2. You can read off important information instead of having to relisen to an audio message.
  3. You can copy and paste to your phone.
  4. You can share information in noisy areas.
  5. You can share information in contexts where loud noises are not permitted. 

### How the Dispatch Hub Currently Works 

On the mobile app, messages are displayed like this:

<img src="/assets/2-phone-msgs.png" alt="drawing" width="200"/>

On the dispatch app, messages are displayedlike this: 

<img src="/assets/3-dispatch-msgs.png" alt="drawing" width="200"/>

When you click the triangle button, the audio message is played.

There are **3 ways** in which messages are exchanged in the dispatch app:

  1. Between Contacts. 
  2. In Channels. 
  3. In Calls

**Contacts** are individual users, like drivers using the mobile app. 

**Channels** include multiple users. Anyone in the channel can send and receive messages. There are two types channel: 

  1. Dispatch
  2. Non-Dispatch. 

Non-Dispatch channels act as explained. Dispatch channels are unique. In addition to exchanging messages, non-dispatchers in the channel can also initiate **Calls**. 

**Calls** are private conversations between a dispatcher and a user. When a call is created, it's added to a queue. A dispatcher can accept the call, have a private conversation with the user, and then end the call. In short, a call has the following lifecycle: created, pending, accepted, ended. 

Calls are used by mobile users to raise particular issues or requests that they need resolved.

### The Dispatch Hub UI 

All of this is tracked on the following Dashboard:

<img src="/assets/4-dash.png" alt="drawing" width="500"/>

There are three main components to point out here. 

  1. The Tabs
  2. Calls List 
  3. Contacts List
  4. Channels List
  3. Converation History Panel 

Clicking on a particular tab reveals the list that corresponds to the tab's name:

![phone-display!](/assets/5-tab-selections.png)

There are also a number of additional features that help with messaging. 

  1. A microphone button that allows you to record and send a message.
  2. A red notification badge that indicates how many unread messages are available.
  3. A red dot next to a message that indicates an unread message.

### Requirements 

The **main requirement**: add the ability to exchange text messages in a way that feels natural with respect to the existing messaging architecture. 

To be a bit more specific, we need to:

  1. show text messages where we already show audio messages
  2. add the ability to create a text messages where we can typically create audio messages.
  3. notify that there are new and unread text messages similar to how we do for audio messages.

## Solution, Challenges and Design Decisions

### Existing Tech Stack 

The tech stack of the Dispatch App is as follows:

  1. Built with Electron JS
  2. Written in TypeScript 
  3. React for front-end components
  4. Redux for client state management 
  5. Sass for styling
  6. Webpack for build
  7. A SQL database management system for local persistence 

### High Level Archicture

At the most general level, the architecture consists of:

  1. The Electron Client
  2. The Database Layer
  3. Message Transfer Service

There are a few things worth pointing out in the interaction between these three:
  
**The Message Transfer Service handles the entire process of sending and receiving messages**. It's accessed via a library imported into the Electron Client that exposes methods for sending and receiving messages. 

**Messages are streamed in and begin to play immediately after there is enough data to play**. This means that the user receives the message as it is being created. The Electron Client uses the Message Transfer Service library to track the beginning of the message, the reception of message data, and the end of the message. These states are tracked in the Redux Store and subseqently represented in the UI. 

**The message streams come with information about the sender, the channel, and date**. This data is decoded by the Message Transfer Service and given to the Electron Client via the library mentioned.

**Messages are automatically stored in the database**. There is a fixed number of allowed messages in the database. Older one's are deleted in favor of newer ones to maintain the number. 

**When a conversation history is pulled up, the Electron Client loads a fixed number of older messages from the database**. If the user scrolls up (i.e in the past), additional messages are lazy loaded from the database into the Redux Store and thereafter the history panel.

**The Electron Client uses multiple redux stores to that are separated by theme to track client state**. For example, there are stores for the conversation history, the lists of contacts and channels, and user information. This stands in contrast to one large redux store. Different components connect to the store they need to.  

**Logging is set up to monitor the app**. 

### The Main Challenge 

The **Main Challenge**: There's already logic for messages. In introducing a new type of message, where do we need to add logic and where do we need to generalize or change existing logic?

On the one hand, you want parity between both kinds of message so that they fit into how the user is already thinking about messages. On the other, they're very different kinds of messages, and the design has to account for that. 

Some similarities: 

  1. You expect messages in the same place 
  2. You expect to send messages from the same place 
  3. You expect to be user to be notified in the same way

Some differences unique to Audio Messages: 

  1. Audio messages automatically play in channels
  2. Audio messages are considered "read" when you hear then
  3. Audio messages are created at the put of a button

### Instances of the Challenge

added logic

1. The creation of an input for creating and sending text messages at the bottom of the conversation history panel. 

2. The creation of a component in the history panel for displaying text messages in the conversation.

3. The modification of the "Unread Nessage Dot" since a text message is "read" under different circumstances from an audio message.

4. The modification of the "Unread Message Notification Circle" to include text messages in the count.

5. The modification of the "Last Sent Arrow" in the Recents List to indicate that the dispatcher recently sent a text message.

6. The creation of a menu with a "Send Text Message" that correspondeds to the microphone button found on the list of channels and contacts. Clicking the button would navigate you to the conversation, unlike the pressing the microphone.

7. The modification of lazy loading messages in the history panel so that text messages from the database are pulled in.

8. The creation of a `TextMessage` table in the database. 

9. The creation of a `TextMessage` type in the client. Eventually, we have three types: `Message`, `AudioMessage`, and `TextMessage` where an instance could be passed around as, e.g., `TextMessage` or `Message` type.

10. The modification and creation of reducers and properties in the store. In the end, some reducers used the general `Message` type and others used the specific `AudioMessage` or `ImageMessage` types. This was made on a case-by-case basis. 

11. The modification of container components like the history component to receive a list of `Message`s instead of `AudioMessage`s. Also done on a case-by-base basis. 

12. The creation of callbacks that receive the text message from the Message Transfer Servie. For audio messages, we expect streams; for text messages, we expect completed data. 

13. The modification of style classes to make different components look the same. For example, the message components in the history panel. 

### Monitoring, Metrics, and Insights



the design
approach
technical details
success metrics 
user insights
how you discussed tradeoffs with your product and design partners.