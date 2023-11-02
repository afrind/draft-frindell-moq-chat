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


# Catalog Format

The moq-chat catalog is used to determine the list of participants in the chat.
The format is a text file with text delta encodings.

Object 0 in any Group is a complete list of the usernames of the participants in
the chat at the time the Object was published, one per-line.  The first line of
Object 0 is the version of the catalog format specificaiton. The format
described here is moq-chat catalog format version 1.

Object 0 Example:

~~~
version=1
alice
bob
charlie
~~~

Any Object in a Group with sequence number higher than 0 is a delta encoding,
with new participants prefaced with a '+' character and participants that have
left the chat prefaced with a '-' character.  Every delta encoding object MUST
be applied for an endpoint to have a current list of participants.

Delta Encoding Exmaple:

~~~
+daphne
-bob
~~~

Every username in the catalog MUST be unique.  If an endpoint receives a catalog
or delta encoding that would result in the same username more than once, it MUST
close the session with an error.

# Chat Operation

## MoQ Chat Server

The protocol requires that one entity operate as a chat server.  The function
of the server is to maintain the list of participants in the chat and publish
catalog updates.  The server can be a standalone server, or can function as a
client of a generic relay.

## Chat ID

Every chat has a unique ID.  The ID is a string of arbitrary length and uniquely
identifies the chat.  The creation of chat IDs and discovery of the server
endpoint is out of the scope of this document.

## Track Names

The catalog for a given chat is available by subscribing to `Track Namespace
= moq-chat/<id>, Track Name = /catalog`.

The chat stream for a participant with username `user` is `Track Namespace =
moq-chat/<id>/participants/<user>, Track Name = /messages`.

The server also has virtual "control" tracks, with names under `Track
Namespace = moq-chat/<id>/control`.

## Control Channel

To send commands to the chat server, a participant sends a SUBSCRIBE to `Track
Namespace = moq-chat/<id>/control/, Track Name = /<uuid>`, where uuid is a
random string sufficiently large to be unique within the MoQ Scope.  The
presencence of the uuid is to bypass any caching relays and ensure the SUBSCRIBE
reaches the chat server.

The start and end group and object locations MUST all be set to Type=Absolute,
Value=0.  There are two additional Track Request Parameters defined:

Command (Key=0x12345): An ASCII string either "join" or "leave".

Username (Key=0x54321): The name of the user joining or leaving

The server processes the command, adding or removing the user from the catalog,
optionally verifying any AUTHORIZATION information requried by the server.  If
the join or leave is successful, the server sends a SUBSCRIBE_OK immediately
followed by SUBSCRIBE_FIN, since the request was for 0 objects.  If the command
failed, the server responds with a SUBSCRIBE_ERROR with an appropriate error
code and reason phrase.

When a moq-chat server processes a command, it MUST update the catalog track.
It can either publish a new Group with the updated particiant list, a delta
encoding against the current Group, or both.

## Joining a Chat

In addition to sending a SUBSCRIBE to the control track to join the chat, a
participant SHOULD SUBSCRIBE to the chat's catalog to get the participant list
and subsequent participant updates.  It is RECOMMENDED the participant request
the catalog starting from the latest Group using Start Group =
RelativePrevious/0, Start Object = Absolute/0. If the participant wishes to
operate in "All-Broadcast, No Receive" Mode, it can omit subscribing to the chat
catalog.

## Subscribing to Chat Messages

After receiving the most recent catalog information, a client SHOULD subscribe
to the track for each active participant.

Upon receiving a delta update removing a participant, a client SHOULD
unsubscribe from that track if it had previously subscribed.

Upone receiving a delta update adding a participant, a client SHOULD subscribe
to the new track.

## Chat Messages

Each chat message is sent in a new Group and new Object.  The format of a chat
chat message for version 1 catalogs is UTF-8 Encoded text.  There is no limit to
the length of a chat message beyond those imposed on QUIC streams.

## Leaving the Chat

When a participant leaves the chat, they SHOULD send a SUBSCRIBE_RST on their
messages track to any subscribers indicating the final group and object.  The
participant SHOULD also send leave command.  A particpant might be disconnected
from the chat server in an ungraceful manner and be unable to send these
messages.

## Stream Mapping

There is no prescription for how to map catalog or chat messages onto QUIC
streams.  Endpoints can choose to use one stream per track, one stream per group
or one stream per object.

## Session Closure by the Server

If a client detects a MOQT session has been closed by the server, it assumes
the server has exited or crashed, and does not attempt to reconnect.  If the
server transmits a Goaway, the client MAY reconnect if a non-empty New Connect
URI is provided

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
