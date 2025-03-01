---

title: "MoQ Chat"
abbrev: "moq-chat"
category: info

docname: draft-frindell-moq-chat-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Applications and Real-Time"
workgroup: "Media Over QUIC"
keyword:
 - media over quic
venue:
  group: "Media Over QUIC"
  type: "Working Group"
  mail: "moq@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/moq/"
  github: "afrind/draft-frindell-moq-chat"
  latest: "https://afrind.github.io/draft-frindell-moq-chat/draft-frindell-moq-chat.html"

author:
 -
    fullname: Alan Frindell
    organization: Meta
    email: afrind@meta.com

normative:

informative:


--- abstract

MoQ Chat (moq-chat) is a simple text based protocol for exercising MoQ
Transport.


--- middle

# Introduction

MoQ Chat (moq-chat) is a simple text based protocol for exercising MoQ
Transport {{!MOQT=I-D.ietf-moq-transport}}.  The protocol allows many
participants to join a virtual chat room, publish messages to the room and
receive messages published by others.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

Commonly used terms in this document are described below.


# Chat Operation

## MoQ Relay

The protocol requires a MoQ relay to act as a chat server.  The relay maintains
the set of connected clients that have announced a chat track.

## Chat ID

Every chat has a unique ID.  The ID is a string of arbitrary length and uniquely
identifies the chat.  The creation of chat IDs and discovery of the relay is
out of the scope of this document. A ID of "_testing" is reserved for
experimental/testing purposes and MUST NOT be used in any realtime deployments.

## Track Names

Each chat participant has a chat track.  The namespace of the track is
("moq-chat", \<id\>, \<user-id\>, \<device-id\>, \<timestamp\>) and the name
track is "chat".

* id - the ID of the chatroom
* user-id - the user ID
* device-id - a unique identifier for each device for the user.  This allows
              the same user to join the chat from multiple devices
* timestamp - the timestamp in seconds when the track started, encoded as a
              string.  This allows a stateless client to start publishing
              without accidentally overwriting a previously sent group and
              object.
              Note: the protocol will still function so long as each chat
              client selects a monotonically increasing number for this field.
              Using the common format described here could support future
              functionality like pulling chat history.

## Joining the Chat

To join the chat a participant sends a SUBSCRIBE_NAMESPACE message to the relay
with a namespace prefix ("moq-chat", \<id\>).  MoQ Relays track the current
state of all announced namespaces and namespace subscriptions, and forward any
matching ANNOUNCE or UNANNOUNCE messages to interested endpoints.

The participant also sends an ANNOUNCE message for their chat track namespace.

## Subscribing to Chat Messages

When receiving an ANNOUNCE that matches the chat prefix, the client extracts the
client's user-id, device-id and timestamp from the third, fourth and fifth tuple
elements, respectively.  The client SHOULD subscribe to the latest timestamp
track for each (user-id, device-id) pair.

Upon receiving an UNANNOUNCE, a client SHOULD UNSUBSCRIBE from that
matching track if it had previously subscribed.

## Chat Messages

Each chat message is sent in a new Group and new Object.  The format of a chat
chat message this draft is UTF-8 Encoded text.  There is no limit to
the length of a chat message beyond those imposed on QUIC streams.  Chat clients
MUST send an END_OF_GROUP message for each Group.

The starting Group ID for each track starts at 0 and increments by 1.  The
Object ID for each chat message starts at 0 and increments by 1.

## Leaving the Chat

When a user leaves the chat, they SHOULD send an UNANNOUNCE message for their
namespace. They also SHOULD publish an object with status
END_OF_TRACK_AND_GROUP on their chat track, since they will start a new track if
they rejoin.  Finally, they SHOULD send an UNSUBSCRIBE message for any
tracks they subscribed to before closing their Transport Session.

If all publishers of a given namespace disconnect from the relay abruptly, the
relay will send UNANNOUNCE messages matching SUBSCRIBE_NAMESPACE to interested
endpoints.

## Stream Mapping

The RECOMMENDED forwarding preference for the chat track is Subgroup, with all
subgroup IDs set to 0, though clients MAY use other forwarding preferences at
their discretion.

## Session Closure by the Server

If a client detects a MOQT session has been closed by the relay, it assumes
the relay has exited or crashed, and does not attempt to reconnect.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
