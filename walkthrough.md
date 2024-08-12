# Project Walkthrough

This document describes an app called the **Dispatch Hub**, an app used for exchanging messages, and the project of adding the ability to exchange **Text Messages**. 

## The Problem We're Trying to Solve

The what and the why.

### Mobile App

Where are we starting with? Zello makes a app for iPhone and Android that turns your phone into a walkie-talkie; it lets you send push-to-talk messages between phones. 

![phone-display!](/assets/1-phone-display.png)

For instance, if you push that big circle button and talk, I'll receive your message in real time on my device. 

### Users 

The app is primarily purchased by businesses and used by teams that are "out in the field" and need to communicate in real time. For example, a power company with multiple trucks driving around fixing downed wires would be an example. They could use the app to ask questions, give updates, and coordinate whoe goes where. 

There are loads of other possible use cases: truck drivers, taxi service, event staff, warehousing & distribution, utility companies, etc. 

### Dispatch Hub (The Dispatching App)

In addition to the mobile app, Zello makes an application called the **Dispatch Hub**. It's a desktop app used by people back at the office - the dispatchers - to coordinate and help out the mobiles users out in the field. 

For example, you can imagine a dispatcher working for the power company coordinatng trucks after a power outage. They're there to answer questions, direct trucks to certain areas, and revieve updates when work is done. 

Before **Text Messages** are added, the Dispatch Hub can exchange audio messages. They can play the messages that are received and send messages to mobile users out in the field. The specifics of how that works is addressed below. 

### The Problem
Add the ability to send **Text Messages** to the Dispatch App. Specifically, add the ability to send and receive Text Messages with mobile users.

### Why Solve the Problem 
There are contexts in which it is more helpful to send a text message. Here are a few: 

1. You can visually search text. For example, you can scroll through a list of text messages looking for a word or phrase. You can't do that with audio messages. 

2. It makes accessing important information faster and easier. Instead of re-listening to an audio message to get an address, you can read it. 

3. It allows you to copy & paste information, say to notes. 

4. It allows you to share information when the area you're in is too noisy to listen to audio messagse. 

5. It allows you to share information in places that do not permit loud audio: library, speaking event, etc. 

### Domain Knowledge 

The language of "exchanging messagess" is fairly general. It's now time to get more specific about how message exhange works on the app to give a more specific idea of what will need to change. 

From the perspective of the mobile user, messages are displayed with on a user interface that's similar to the messaging apps we're used to: 

![phone-display!](/assets/2-phone-msgs.png)

From the perspective of the dispatch app, messages are displayed similarly. This is a sub-component of the dispatch app that shows messages: 

![phone-display!](/assets/3-dispatch-msgs.png)

In either case, when you click the triangle play button, the audio message is replayed: 

There are three contexts in which messages are exchanged in the dispatch app:

  1. Exchanging messages with Contacts. 
  2. Exchanging messages in Channels. 
  3. Exchanging messages in Calls

**Direct Contacts** are individual users, like drivers using the mobile app and other dispatchers on their own instance of the dispath hub. These act like direct messages in a Slack Channel. It's just a conversation.

**Channels** are a bit like Slack Channels in which multiple users can particpate. Anyone who is in the channel can send and receive messages. Creating channels and assing users to the channel is configured on a different app. You cannot do that from Dispatch Hub. 

There are two types channel: 

1. Dispatch
2. Non-Dispatch. 

Non-dispatch channels act as explained; if you're in the channel you can send and receive messages. Dispatch channels are unique. No only can you send and receive messages, non-dispatchers who are part of the channel can also initiate **Calls**. 

**Calls** are private conversations between a dispatcher and a user. A call has the following lifecycle: created -> pending -> accepted -> ended. 

When a call is created, it goes into a queue. A dispatcher can accept a call that is in the queue. Then, the dispatcher can have a private conversation with the user who initied the call. Then the dispatcher can end the call. 

The use case for a Call is when a field workd has a particular question or issue that they need focused attention on until resolved. They "call in" to the dispatcher for help.  

#### The Dispatch Hub UI 

All of this is tracked on the following Dashboard:

![phone-display!](/assets/4-dash.png)

There are three main components to point out here. 

First, the tabs on the top left permit the user to select what kind of message exchange they're interested in. The first three options correspond to the three contexts of message exchange mentioned above: Calls, Contacts, and Channels. The option "Recent" shows a list of your most recent message exchanges. 

Second, beneath the tab component on the left is a list that shows your messaging options. It displays the existing Calls, Contacts, and Channels that you are available to message in when the corresponding tab is selected. Each has a slightly different UI: 

![phone-display!](/assets/5-tab-selections.png)

When you click on any of the options that present in the list, it will bring up your converation history with that user. That is shown in the right component, which also displays basic information about the channel, contact, or call at the top. 

#### The Problem in Context

So that's the set up. We want to add Text Messages in way that makes use of and complements the existing audio message set up. Here are a few additional challenges: 

1. Components for writing and sending and message 
2. Notifications showing that there's a new message
3. Notifications showing that a message is unread


#### General Requirements 

> can recieves text message
> persists text message in local storage
> shows actual text message in dispatch conversation history 
> shows new text message notifications to dispatch user
> shows unread text message notifications to dispatch user 
> can send text messages to other clients 

## Solution, Challenges and Design Decisions

The how. 

### Existing Tech Stack and Architecture 

The basic dependencies and tech stack is as follows:

1. Built with Electron js. 
2. Written in TypeScript 
3. React for front-end components
4. Redux for client state management 
5. Sass for styling
6. Webpack to build
7. SQL database management system

The basic archicture: 

At the most general level, the architecture consists of the Electron Client, the Database Layer, and a Messaging API. The messaging api is a service that already exists that handles the entire process of encoding and transferring messages. It this context, it is already built to handle Text Messages. The Electron Client subscribes to this service to receive messages. When the client receives the messages, its automatically stores them in the database to save the history. The number of messages in the database is monitored, only allowing a specific number of saved messages. When new messages come in and the number is exceeded, the oldest set of messages is deleted. 

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