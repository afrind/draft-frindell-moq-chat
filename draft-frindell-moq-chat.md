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
catalog updates.

## Chat ID

Every chat has a unique ID.  The ID is a string of arbitrary length and uniquely
identifies the chat.  The creation of chat IDs and discovery of the server
endpoint is out of the scope of this document.

## Track Names

The catalog for a given chat is available by subscribing to moq-chat/id.

## Joining the Chat

To join the chat a participant sends a SUBSCRIBE message to moq-chat/id in order to
get the participant list and subsequent participant updates.  This SUBSCRIBE requires
the AUTHORIZATION_INFO to contain the participant's ASCII string username.
It is RECOMMENDED the participant request the catalog starting from the beginning 
of the current Group. 

The participant also sends an ANNOUNCE message to the server with a track namespace 
of moq-chat/id/participant/username, which is also the Full Track Name.  The moq-chat
server will reply with an ANNOUNCE OK if joining was successful or ANNOUNCE ERROR if
the join failed.

When a moq-chat server receives a SUBSCRIBE and sends an SUBSCRIBE OK, it MUST
update the catalog.  It can either publish a new Group with the updated
particiant list, a delta encoding against the current Group, or both.

## Subscribing to Chat Messages

After receiving the most recent catalog information, a client SHOULD subscribe
to the track for each active participant.

Upon receiving a delta update removing a participant, a client SHOULD
unsubscribe from that track if it had previously subscribed.

Upone receiving a delta update adding a participant, a client should subscribe
to the new track.

## Chat Messages

Each chat message is sent in a new Group and new Object.  The format of a chat
chat message for version 1 catalogs is UTF-8 Encoded text.  There is no limit to
the length of a chat message beyond those imposed on QUIC streams.

## Leaving the Chat

When a user leaves the chat, it would be nice if they could send a message
indicating that their track is complete or no longer available, but the protocol
has no such message (yet).

When a server or relay detects a MOQT session has terminated, it MUST update the
catalog and remove any participants that had sent ANNOUNCE messages on that
session.

## Stream Mapping

There is no prescription for how to map catalog or chat messages onto QUIC
streams.  Endpoints can choose to use one stream per track, one stream per group
or one stream per object.

## Session Closure by the Server

If a client detects a MOQT session has been closed by the server, it assumes
the server has exited or crashed, and does not attempt to reconnect.

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
