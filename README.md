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

The **main requirement**:add the ability to exchange text messages in a way that feels natural with respect to the existing messaging architecture. 

To be a bit more specific, we need 

  1. show text messages where we already show audio messages
  2. the ability to create a text messages where we can typically create audio messages.
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
  
The Message Transfer Service handles the entire process of sending and receiving messages. It's accessed via a library imported into the Electron Client that exposes methods for sending and receiving messages. 

When the client receives the messages, its automatically stores them in the database to save the history. The number of messages in the database is monitored, only allowing a specific number of saved messages. When new messages come in and the number is exceeded, the oldest set of messages is deleted. 

One very notable issue with the message service is that it does not send compelted messages. Instead, it streams messages so that once a user begins to send the message, the stream is received and the client begings to play what it has. So there are 3 important events that the client needs to trac: message beginning, receiving data, message ending. 

Similary with sending an audio message. A stream is openened up and continues to send. 

The streams also carries all the required information about the sender, the channel, and date. 

The message service also provides lists of channels and contacts. 

As far as updating message UI, when a dispatcher opens a coversation, limited portion of the messages from the database history are pulled up. Then, new messages are added to the client ui.

There's also existing logging that logged exceptions, failed messages, etc. 

### The Main Challenge 

When starting work on the problem of adding text messages, the dispatch app was fully fledged out for sending audio messages. It was already working with messages. There's existing logic and existing UI. Now, were adding a new kind of message. So how are you going to fit in a new kind of message to an existing archteciture that's meant to deal with a different kind of message? 

Am I going to try to generalize or augment existing logic and components or create new log and new components to handle the new case, and risk code that isn't DRY. 

So a lot of the questions for this project were of the form: am I going to make this handle two kinds of messages or am I going to create a different handler. 

And there were two categories of this question: UX and Logic. Here are a few areas where this question came up. 


augment component
create unique component

### Instances of the Challenge

1. Representing Messages. On the database level, audio messages and text messages were distinguished and stored as different tables. Retreiving all messages included retreiving from both tables. On the client, they were also given different types. However, the audio message and text message had an overlap type, Message, and could be referred to as such. So you could have a list of objects of the `Message` type but then refer to as `TextMessage` or `AudioMessage`. On the reception of the message, the service told us which was which. 

2. Message Handler 
For audio messages, you track the state of receiving when you receive it. For text message you don't

#### UX

There were a few main compeonts that needed to be created or augmented. 

1. The creation of a text message bar at the bottom of the conversation history panel. 

2. The creation of a text message component corresponding to the audio message component. It received and displayed the text message. 

3. The conversation history panel needed to be augmented so that it allowed for a new kind of message display; namely, the component made above. 

4. A way to send text messages that corresponded to the microphone icon found in the channel and contact options. When you click the microphone, you can send a message. We wanted a corresponding way to send text messages. So, we added menu button that displayed upon hovering over the bar and, when clicked, gave the option to send a Text Message. 

5. Augmenting the New Message Notification Circle to include TextMessages into the count. Shows up on Contect option

6. Show message as unread. On the audio message, there is a red dot that shows and goes away after the message has been palyed. When will we rmove that for Text Message

7. Certain channels automically play a message when it comes in so there's no reason to have an unheard message notification. However, they haven't read it, so it still has to be conisdfeed unread for rtext messages

8. show sent message for text message on recents

9. number of unread messages in box
unread message red dot on conversation history

10. lazy loading in history panel

11. style classes to share style but differetn components

# monitoring
  - what are the potential problems, bugs to watch out for?
  - how are we going to measure how much the feature gets used? 

the design
approach
technical details
success metrics 
user insights
how you discussed tradeoffs with your product and design partners.