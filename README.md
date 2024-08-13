# Project Walkthrough

## The Problem 

The **Dispatch Hub** is an app used for exchanging **audio messages** created by Zello. 

This document describes the project of adding **text message** exchange to the Dispatch Hub.

### Mobile App

Zello makes a mobile app that lets the phone act like a walkie-talking. You can send a voice message to another user by pushing the big button and talking.

<img src="/assets/1-phone-display.png" alt="drawing" width="200"/>

### Users 

The app is used by teams and people that are "out in the field" and need to communicate in real time, like truck drivers, taxi services, event staff, warehousing teams, utility companies, etc. 

### Dispatch Hub

The **Dispatch Hub** is a desktop app used by people back at the office, the dispatchers, to coordinate and help the mobile users out in the field. 

For example, you can imagine a dispatcher working for the power company coordinating trucks after a power outage. They're there to answer questions, direct trucks to certain areas, and receive updates when work is done.

### Why Solve the Problem? 
There are contexts in which it is more helpful to send text messages. Here are a few: 

  1. You can visually search through text messages. 
  2. You can read off important information instead of having to relisten to an audio message.
  3. You can copy and paste the information.
  4. You can share information in noisy areas.
  5. You can share information in contexts where loud noises are not permitted. 

### How the Dispatch Hub Currently Works 

On the mobile app, messages are displayed like this:

<img src="/assets/2-phone-msgs.png" alt="drawing" width="200"/>

On the dispatch app, messages are displayed like this: 

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
  5. Recents List 
  6. Conversation History Panel 

Clicking on a particular tab reveals the list that corresponds to the tab's name:

![phone-display!](/assets/5-tab-selections.png)

There are also a number of additional features that help with messaging. 

  1. A microphone button that allows you to record and send a message.
  2. A red notification badge that indicates how many unread messages are available.
  3. A red dot next to a message that indicates an unread message.

### Requirements 

The **main requirement**: add the ability to exchange text messages in a way that feels natural with respect to the existing messaging architecture. 

To be a bit more specific, we need to:

  1. Show text messages where we already show audio messages
  2. Add the ability to create text messages where we can typically create audio messages.
  3. Notify that there are new and unread text messages similar to audio messages.

## Solution, Challenges and Design Decisions

### Existing Tech Stack 

The tech stack of the Dispatch App is as follows:

  1. Built with Electron JS
  2. Written in TypeScript 
  3. React for front-end components
  4. Redux for client state management 
  5. Sass for styling
  6. Webpack for bundling front-end
  7. A SQL database management system for local persistence 
  8. Unit testing with Jest 
  9. Logging with `electron-log`

### High Level Architecture

At the most general level, the architecture consists of:

  1. The Electron Client
  2. The Database Layer
  3. Message Transfer Service

There are a few things worth pointing out in the interaction between these three:
  
**The Message Transfer Service handles the entire process of sending and receiving messages**. It's accessed via a library imported into the Electron Client that exposes methods for sending and receiving messages. It’s interface is fully read to exchange text messages.

**Audio Messages are streamed in and begin to play immediately after there is enough data to play**. This means that the user receives the message as it is being created. The Electron Client uses the Message Transfer Service library to track the beginning of the message, the reception of message data, and the end of the message. These states are tracked in the Redux Store and represented in the UI. 

**The message streams come with information about the sender, the channel, date, and other basics.**. This data is decoded by the Message Transfer Service and given to the Electron Client via the library mentioned.

**Messages are automatically stored in the database**. There is a fixed number of allowed messages in the database. Older one's are deleted in favor of newer ones to maintain the number. 

**When a conversation history is pulled up, the Electron Client loads a fixed number of older messages from the database**. If the user scrolls up (i.e in the past), additional messages are lazy loaded from the database into the Redux Store and then the history panel.

**The Electron Client uses multiple redux stores that are separated by theme to track client state**. For example, there are stores for the history, contacts, channels, and an active message. This stands in contrast to one large redux store. Different components connect to the store they need to.  

**Logging is set up at points of critical data transfer and display**, so database queries, messages creation and reception, primary renders like conversations and lists. We did not use logging to evaluate user behavior. It’s used to monitor critical connections in the app.  

### The Main Challenge 

The **Main Challenge**: There's already logic for messages. In introducing a new type of message, where do we need to add logic and where do we need to generalize or change existing logic?

On the one hand, you want parity between both kinds of message so that they fit into how the user is already thinking about messages. On the other, they're very different kinds of messages, and the design has to account for that. 

Some similarities: 

  1. You expect to see messages in the same place
  2. You expect to send messages from the same place.
  3. You expect to be notified of new messages in a similar way.

Some differences unique to Audio Messages: 

  1. Audio messages automatically play in channels
  2. Audio messages are considered "read" when you hear them
  3. Audio messages are created at the push of a button

### Instances of the Challenge

1. The creation of an input for creating and sending text messages.

2. The creation of a component in the history panel for displaying text messages.

3. The modification of the "Unread Message Dot" to account for a “read” text message.

4. The modification of the "Unread Message Notification Circle" to include text messages in the count.

5. The modification of the "Last Sent Arrow" in the Recents List to indicate that the dispatcher recently sent a text message.

6. The creation of a menu with a "Send Text Message" button that corresponders to the microphone button found on the list of channels and contacts. 

7. The modification of lazy loading messages in the history panel so that text messages are pulled in from the database on load.

8. The creation of a `TextMessage` table in the database. 

9. The creation of a `TextMessage` type in the client. Eventually, we have three types: `Message`, `AudioMessage`, and `TextMessage` where an instance could be passed around as, e.g., `TextMessage` or `Message` type.

10. The modification and creation of reducers and actions for the stores. In the end, some reducers used the general `Message` type and others used the specific `AudioMessage` type. 

11. The modification of container components like the history component to receive a list of `Message`s instead of `AudioMessage`s. 

12. The creation of callbacks that receive the text message from the Message Transfer Service. For audio messages, we expect streams; for text messages, we expect completed data. 

13. The modification of style classes to make different React components look the same like the bubble style of the message components in the history panel. 

14. Modify logging around messages so that logging distinguishes text and audio messages where it makes sense to do so. 

### Design Process

The design process looked like this: 

  1. Product and design would discuss the feature and create wireframes
  2. There would be initial discussions with development 
  3. Full mocks were created
  4. Another round of discussions with developers
  6. Requirements specified
  7. Division into tasks
  8. Implementation 

This was the general approach, but there was constant contact throughout the process. 

### Success Metrics and User Insights

The two main questions for the feature were: 

  1. Will text messages be used? 
  2. Will text messages be enjoyable to use? 

From a data perspective: 

  1. Measure use via logging
  2. Measure use via database 

The primary method, however, was Zello's relationship to clients. They were in constant contact with them, getting feedback about what they needed and how things worked. 