# react-native-qiscus-sdk

## example of usage

### 1) Install from npm

`npm install --save react-native-qiscus-sdk && npm install`

Beware please since this SDK use native-base component which require `"react-native": "0.45.1"` higher react-native version not supported yet, lower version are welcome.

You can specified react-native version in your new project by run this command `react-native init AppSDK --version 0.45.1`

### 2) Run React Native Link

```

react-native link react-native-vector-icons

```

### 3) Your code inside index.ios.js / index.android.js will look like

```javascript

/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */

import React, { Component } from 'react';
import {
  AppRegistry,
  StyleSheet,
  Text,
  View,
  Image,
  TouchableOpacity,
} from 'react-native';

import {InitApp, ChatRenderer} from 'react-native-qiscus-sdk';

export default class AppSDK extends Component {
  constructor() {
    super();
    this.state = {
      qiscus: null,
      newMessage: null,
      rooms: null,
      selectedRoom: null,
    };
  }
  componentWillMount() {
    // required object to initialize app, this config just for demo, change it with your qiscus credential account.
    const userAuth = {
      email: 'fikri@qiscus.com',
      password: 'password',
      displayName: 'fikri',
      avatar: null,
      appID: 'sdksample',
    }
    // required call back method to set global state of rooms list
    const setRooms = (data) => this.setState({rooms: data});

    // required call back method to set global state of qiscus object
    const initApp = (data) => this.setState({qiscus: data});

    // required call back method to catch new received message
    const receiveNewMessage = (data) => this.setState({newMessage: data});
    InitApp({initApp, receiveNewMessage, setRooms, userAuth});
  }
  _openChat(room: {name: string, id: number}) {
    this._chatTarget(room);
  }
  _chatTarget(room: Object) {
    this.setState({
      selectedRoom: room,
    });
  }
  render() {
    const {state: {rooms, selectedRoom, qiscus, newMessage}} = this;
    const initApp = (data) => this.setState({qiscus: data});
    if (!rooms) {
      return <View style={{marginTop: 40}}><Text>Initialize App...</Text></View>;
    }
    if (!selectedRoom) {
      return (
        <View style={styles.container}>
          {rooms.map((item, i) => {
              const name = item.room_name;
              const avatar_url = item.avatar_url ? item.avatar_url : 'https://qiscuss3.s3.amazonaws.com/uploads/55c0c6ee486be6b686d52e5b9bbedbbf/2.png';
              return (
                  <TouchableOpacity
                    key={i}
                    style={styles.row}
                    onPress={() =>
                      this._openChat({name: name, id: item.id})
                    }
                  >
                    <View style={styles.containerRow}>
                      <Image source={{ uri: avatar_url }} style={styles.photo} />
                      <Text style={styles.text}>
                        {name}
                      </Text>
                    </View>
                  </TouchableOpacity>
              );
            })}
        </View>
      );
    } else {
      return (
        <View style={styles.container}>
          <TouchableOpacity style={{marginLeft: 30, marginBottom: 10, marginTop: 0, justifyContent: 'center', alignItems: 'center', height: 40, width: 80, borderWidth: 1, borderColor: '#333131'}} onPress={() => this.setState({selectedRoom: null})}>
            <Text>Back</Text>
          </TouchableOpacity>
          <ChatRenderer qiscus={qiscus} message={newMessage} room={selectedRoom} initApp={initApp} />
        </View>
      );
    }

  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#F5FCFF',
    marginTop: 40,
  },
  welcome: {
    fontSize: 20,
    textAlign: 'center',
    margin: 10,
  },
  instructions: {
    textAlign: 'center',
    color: '#333333',
    marginBottom: 5,
  },
  photo: {
    height: 40,
    width: 40,
    borderRadius: 20,
  },
  containerRow: {
    padding: 12,
    flexDirection: 'row',
    alignItems: 'center',
    borderBottomWidth: 1,
    borderBottomColor: '#dce2e9',
  },
  text: {
    marginLeft: 12,
    fontSize: 16,
  },
});

AppRegistry.registerComponent('AppSDK', () => AppSDK);


```

[Working example are here:](https://github.com/qiscus/qiscus-react-native-js-example)


#### Run `react-native run-android / react-native run-ios`

#### If you got `ENOENT: no such file or directory, uv_chdir`, make sure you have create index.ios.js / index.android.js file on your root project directory




# API DOC

Once you initiated this SDK the SDK will give you Object. Let say you have assign this object into `qiscus` varibale. So here is the list of available API:

**Send message**

* Text comment:

qiscus.submitComment(roomId, message, commentUniqueId, type, payload)

* File comment / upload media

qiscus.uploadFile(roomId, file)
// file can be acquired from event 'e.target.files || e.dataTransfer.files'

* Custom payload comment:

qiscus.submitComment(roomId, message, uniqueId, type, payload)

**Load comments**

calling qiscus.chatTarget or qiscus.chatGroup will load 20 comments from respected room

**Load more comments**

qiscus.loadComments(roomId, lastCommentId)


**Publish typing**

// value 1 for typing, 0 for finish typing
qiscus.realtimeAdapter.publishTyping(value)


**Update message state (read)**

// this method tells others that this particular comment has been read
// as for the state, it's automatically updated by the core
// comment is a <Comment> object, get it from qiscus.selected.comments
qiscus.updateCommentStatus(room_id, comment)


**Create chat room (group)**

qiscus.createGroupRoom(room_name, unique_id)


**Get chat room by id**

qiscus.chatGroup(room_id)


**Get chat room by channel**
*create if not exist

qiscus.getOrCreateRoomByChannel(channel_name)


**Get chat room by opponent user_id**
*create if not exist

qiscus.chatTarget(userID)


**Get rooms list**

* Load room with specific room id and channel id:

**Get room members**

qiscus.selected.participants

**callbacks/Events handler**

**Listen new comment**


newMessagesCallback(comment)

**Listen  comment state**
pass

commentReadCallback(data)

or

commentDeliveredCallback(data)

while initiating QiscusSDK

**Listen user online presence**

 presenceCallback(presence)

**Listen user typing**

pass

typingCallback(data)

while initiating QiscusSDK

**Object

User :**

qiscus.userData // properties :
properties:
- app: { code, id }
- avatar_url
- email
- id
- last_comment_id
- token
- username

**Room : **

qiscus.rooms // all rooms loaded
qiscus.selected // currently selected room

Room Properties:
- avatar
- comments
- participants
- id
- name
- count_notif
- last_comment_id
- last_comment_message
- last_comment_message_created_at
- last_comment_topic_id
- last_comment_topic_title


**Comment :**

qiscus.selected.comments // properties :
properties:
- id
- message
- avatar
- date
- beforeId
- afterId
- isRead
- isDelivered
- isFailed
- isSent
- room_id
- type
- subtype
- unique_id
- username_as
- username_real
- payload: {}
