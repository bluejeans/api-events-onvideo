# BlueJeans Meeting Events SDK's



![927](./927.png)

## Introduction

BlueJeans meetings sends realtime updates on the status of participants and the connections.  This data is sent over a Websocket, and the SDK's here provide application developers with simple modules to access this realtime data.





## About Event Data...

The EventService returns information asynchronously using JSON data objects over a websocket.  Since events can happen rather quickly, the JSON object compresses function operations and uses very short field names.

The structure of the raw event message generally follows this format:

```javascript
[ "String:event message type", { BlueJeansObject:"relevant to message type"} ]
```



BlueJeans will send one of 4 message types to identify the function of the raw message.  The message types defined include

| Message Type                       | Description                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| keepalive                          | a request by BlueJeans cloud to see if this websocket/client is functioning normally.  The client should respond with a string message "heartbeat" |
| meeting.notification.guid_assigned | Not applicable                                               |
| meeting.register.error             | This Events endpoint was unable to register into the specified BlueJeans meeting |
| meeting.notification               | The object member of the raw event message contains a JSON object that describes a change of condition in the BlueJeans meeting |



### BlueJeans Event Object

The salient information related to the source and details of an event in a BlueJeans meeting is contained in the BlueJeans Event Object.  The object itself can describe one of many different types of event stimuli.  Depending upon the event stimulus, relevant parameters are passed in the body property of the BlueJeans Event Object.

The raw BlueJeans Event Object is of this format

```javascript
{
    // various header fields
	body: "stringified JSON of event-specific condition"
}
```

Perform a JSON.parse() on the **body** parameter to create the following JSON object. 

```javascript
{
  event: "string",	// Event Message Type
  props: {
      // Data properties applicable to the specified event Message Type
  }
}

```



### Event Message Types

BlueJeans sends various types of event messages depending upon the scope of the event.  The event type is described the the **event** property.  The event type string consists of a prefix constant, as described below, followed by a "dot-meetingID" substring.  For example :"statechange.livemeeting.12345678".

The relevant type strings are any of the following values.

- **statechange.livemeeting** - The event message details parameters about the meeting entity itself.  For example, if the meeting starts, or terminates, this event is generated
- **statechange.endpoints** - The event message details parameters about a specific participant endpoint.  For example, if an endpoint mutes its audio, this event is generated.



### Event Object - statechange.livemeeting

For the livemeeting object, the following property fields are available

```javascript
  event : "statechange.livemeeting.4159...751",
  props : {
    meetingId : "4159...751",
    meetingGuid : "4159...751:397138-f24d.....-748-d54c",
    meetingState : "ActiveMeeting",
    status : "active",
    isContentSharingActive : false,
    bridged : false,
    locked : false,
    audioMuteOnEntry : false,
    videoMuteOnEntry : false,
    moderatorLess : true,
    title : "Glenn's Meeting",
    chatEnabled : true,
    pinnedEndpointGuid : "00000000-0000-0000-0000-000000000000",
    audioEndpointCount : 1,
    videoEndpointCount : 0,
    recordingEnabled : true,
    participantWebJoinURL : "https://bjn.bluejeans.com/415...",
    isLargeMeeting : false,
    features : ["array of strings"],
    delayedMeetingEndTime : null,
    smStreams : null,
    meetingMarkedForDelayedTermination : false,
    inactiveMeetingStatus : false,
    recordinginfo : {
      contentStatus : null,
      recordingStartTime : null,
      active : false,
      recorded : false
    }
```



### Event Object - statechange.endpoints 

The endpoint statechange message supports multiple operations.  The operation(s) are encoded into the internal properties of the "props" property of this JSON object.  

The generalized format of the JSON object structure is shown here:

```javascript
event : "statchange.endpoints.12345678",
meetingGuid : "string",
props : {
  f : [{ inFo_obj:1},   {inFo_obj:2},   { inFo_obj:"n"} ],
  a : [{ Add_obj:1},    {Add_obj:2},    { Add_obj:"n"} ],
  m : [{ Modify_obj:1}, {Modify_obj:2}, { Modify_obj:"n"} ],
  d : [{ Delete_obj:1}, {Delete_obj:2}, { Delete_obj:"n"} ]
}
```



Bear in mind the following conditions pertinent to this object:

- Any one or more of the "props" property ( f,a,m,d )  may be present in a single event object.  The property name is shorthand for the type of event operation:
  - "f" : inFormation event.  Typically this event is sent when an application first connects to the Event.  It is an informational snapshot of the current state of the BlueJeans meeting.  All subsequent event messages are incremental operations done to this snapshot
  - "a" : Add event.  This event is sent when a new endpoint is added to the meeting.
  - "m" : Modify event.  This event indicates that some condition(s) for the designated endpoint(s) have been modified.  For example, if a participant mutes their voice, BlueJeans sends an "m" event
  - "d" : Delete event.  This event is sent when a participant leaves the meeting

- Each operation property is itself an array of property objects.  If the "props" property is present, then there will be at least one property object in that array.

Here is an example of a multiple-operation statechange endpoints JSON event record.

```javascript
{
  event : "statechange.endpoints.4159...751",
  meetingGuid : "4159.....8ad54c",
  props : {
    m : [
      {
        A2 : "1",
        c : "9e4d59dc-c598-4f20-8aaa-15e15c551045",
        T : "0",
        E1 : "seamguid:b8fc8581-755d-4d18-9efe-273a65ec898b"
      },
      {
        V6 : "480",
        c : "81fe78ea-e9dc-40a6-9ada-fa2636df7848",
        T : "0",
        V1 : "1",
        V2 : "0",
        E1 : "seamguid:28d15bed-2c23-45a0-803e-a9286950d88d",
        V5 : "640"
      }
    ],
   d : [
      {
        E1 : "seamguid:28d15bed-2c23-45a0-803e-a9286950d88d",
        c : "81fe78ea-e9dc-40a6-9ada-fa2636df7848",
        n : "Glenn Inn",
        v : "1"
      }
    ]
  }
}
```



### Sample JSON Object Properties

Here is a statechange event message that might be sent when a guest joins meeting "111222333".  The description for each of the properties is provided below.

```javascript
{
  event : "statechange.endpoints.111222333",
  meetingGuid : "",
  props : {
    f : [
      {
        A1 : "1",
        A2 : "0",
        A3 : "0",
        A4 : "Opus 16Khz",
        A5 : "Opus 16Khz",
        C1 : "5",
        C5 : [
          {
            callGuid : "34c4342a-5ff5-48e6-a38f-7388f443a573",
            capabilities : [
              "AUDIO",
              "VIDEO",
              "CONTENT"
            ],
            connectionGuid : "connguid:4e0cffc5-9ab1-4f03-8dad-dc0df9723463",
            endpoint : "WebRTC"
          }
        ],
        E1 : "seamguid:586e4215-a65c-42d8-9b35-d3711a4b235a",
        L1 : "1",
        L2 : "1",
        S1 : "1",
        S2 : "0",
        T : "0",
        V1 : "1",
        V2 : "0",
        V3 : "0",
        V4 : "VP8",
        V5 : "720",
        V6 : "1280",
        V7 : "VP8",
        V8 : "180",
        V9 : "320",
        c : "34c4342a-5ff5-48e6-a38f-7388f443a573",
        ch : "2e9f2979f40e991e93c18137e6ca04dd",
        e : "WebRTC",
        lc : true,
        m : "111222333:276818-1c55e939-6b20-427e-88cf-91c0c37e2ef8,0",
        n : "John Smith",
        sm : false,
        v : "1"
      }
    ]
  }
}
```

------



### Data Field Definitions

The following list gives the functional meaning of the short JSON field names.

* "A1": Boolean, if audio is being sent by client at all
* "A2": Boolean, if client is audio local muted
* "A3": Boolean, if client is audio remote muted
* "A4": String, audio codec used by client to send
* "A5": String, audio codec used by server to send
* "C1": Integer from 1-5, current call quality of client
* "C2": String, the callguid of the client who's currently presenting. Null if no one is presenting.
* "C3": Integer, the height of content being sent by this client
* "C4": Integer, the width of content being sent by this client
* "C5":  Array, a list of physical connections that this participant represents. More details later.
* "E1": String, the participant guid of this participant
* "L1": Boolean, if this participant is a moderator
* "L2": Integer, 4 if the client's main stream is just video, 2 if the client is receiving content and video in the same stream (non-dual-stream content)
* "P1": String, callguid of the participant whose video is pinned for this participant
* "R1": String, remote desktop version
* "R2": Boolean, can this client be a RDC controller
* "R3": Boolean, can this client be a RDC controllee
* "S1": Boolean, is this call secure, i.e. encrypted
* "S2": Integer, which video layout the client is using. See reference below.
* "T": Boolean, are we detecting talking from this client
* "V1": Boolean, if video is being sent by client at all
* "V2": Boolean, if client is video local muted
* "V3": Boolean, if client is video remote muted
* "V4": String, video codec used by client to send
* "V5": Integer, the height of video being sent by this client
* "V6": Integer, the width of video being sent by this client
* "V7": String, video codec used by server to send
* "V8": Integer, the height of video being received by this client
* "V9":Integer, the width of video being received by this client
* "a": Array, list of alerts for this client. More on this later.
* "c": String, the callguid of this participant
* "ch": String, the chatguid of this participant
* "crp": Dictionary, arbitrary metadata attached to this participant by any client
* "e": String, the type of this endpoint, e.g Carmel, Blue, WebRTC, Room System, etc.
* "m": String, the meeting id of this meeting
* "n": String, the name of this participant
* "r": Boolean, is this endpoint being remote desktop controlled
* "t": String, always "endpoint" for some reason
* "v": Boolean, should this participant be visible in the roster. Internal entries like the meeting recorder and cascade legs are invisible.

