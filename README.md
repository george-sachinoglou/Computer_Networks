# Computer Networks Description

>The purpose of this project was to get familiar with programming in a socket environment and implementating mechanisms of certain protocols. 

:warning: Some of the functions mentioned below have not been implemented.

The application simulates the simple functions of a Social Network with publishing, photo sharing capabilities
and accompanying texts based on client-server architecture using sockets.

The system consists of a server and multiple similar clients, which maintain an account on the server and have an identifier (clientID). Every client has
a set of followers, which constitute his social graph (SocialGraph). The graph can be changed during execution.

Contents:
* [KDClient Interface](KDClient-Interface)
* [KDServer Interface](KDServer-Interface)
* [MultiThreading](MultiThreading)
* [Notes](Notes)

  ## KDClient Interface
  
In this simple form of the network, we consider that each client performs the following functions:

* Each client communicates with the server over a network connection using sockets (sockets).
    
* Creates and maintains an account on the server and has an identifier (clientID)
(implementation of signup and log in functions respectively).

* Maintains a list of clients it "follows" (following) and is notified
for their publications.

* Maintains a set of "followers", i.e. a list of clients, whick are informed of his own posts.

* Maintains a directory locally, in which photo files are stored
as well as accompanying texts. This directory must be synchronized with the
corresponding directory in his account on the server.

* It can read another client's Profile_XclientID (implementation of function
access_profile) it follows, but not its directory (either locally or on the server).
The access_profile function returns the current profile format to the requesting client
that was requested. Therefore, to have access to a specific photo on
directory of the other client (the title of which must already be published in the
Profile_XclientID) it is required to make a request (as described in clause 11 below).
Access is not allowed for users that the client in question does not follow
in Profile_XclientID and the associated request for access_profile is rejected. The rejection
is accompanied by the sending of a relevant diagnostic message (implementation of
deny_profile mode).

* Can add a photo file and a file with a corresponding attachment
text in its local directory. Once the addition is complete:

  * The client directory on the server must also be updated (implementation of
upload and synchronization functions).

  * The client must post a diagnostic message to Profile_XclientID
which will be of the form: ”clientID posted Photo_name”, where Photo_name is the
name of the photo file that was uploaded to its directory, and it should be indicative of its content.

* It can search for a photo in its social graph (implementation of 
search function on the server) using Photo_name, where Photo_name is its name
file. In the search mode, the server returns a list of the IDs of the clients that
have photo files with the specific landscape and belong to the client's social graph
who initiated the search.

* Can ask to download a photo file, from the directory
of the clients it "follows", in its directory (both on the server and locally
as described below) (download mode implementation). More specifically:

  * the client searches (implementation of search function) for a photo in
his social graph using Photo_name. In case the search
(for which the server responds) finds multiple files with the same photo
(in multiple clients) only one of them is randomly selected.

  * Based on the aforementioned selection from the list generated by
search, the client requests the specific photo file and the corresponding
accompanying text from one of the clients he "follows" (eg
client B), sending them a corresponding request with the file name Photo_name
on the server. The request mentions the identity of client B from who's directory
the server should offer the file.

  * To start the download process from the selected directory
then handshake packets must be exchanged on the server. So,
the 3-way handshake process is implemented at the level
of the specific application, in the third step of which the client requests the file that they
wish. The client initially sends a communication request with the server. The
server confirms that it accepts this communication and then the client sends the name of the desired photo file to the server.

  * We consider that the server that has just received a request (in its 3rd and last step of the
handshake) starts sending the requesting client the photo file, fragmented into multiple (about 10) messages (= APDU = Protocol
Application Layer Data Units). The server waits for a confirmation, for each message it sends,
from the client, in order to start sending the
next message, which is basically the stop-and-wait protocol being implemented
but at the level of the specific application. With the start of the send off the server starts a timer to indicate the timeout (if
the corresponding confirmation is delayed), for which, the deadline must be selected appropriately.

  * For the 3rd message, the client (intentionally) does not send an acknowledgment on the first transmission. Then, once the timer expires, the server displays a message “Server did not
receive ACK" and resends the message (function implementation
retransmit). Once confirmation of the copy of the message is received, the rest of the messages
are sent and confirmed normally (by sending confirmation directly from
the client).

  * Also, for the 6th message, the client delays quite a bit (on purpose) before sending
confirmation of its first broadcast. Therefore, the above must be re-implemented and also the server must properly manage the confirmations being sent to it.

  * The accompanying photo text is sent in a single message, because
it is very short. In case there is no accompanying text for a photo file, a diagnostic message is sent to the client, informing them.

  * Upon completion of the process of sending the photos and the
accompanying text, the following diagnostic message appears on the client: “The
transmission is completed”. The directory of the client still needs to be updated
on the server side (synchronization function implementation), i.e. to have
the same files as the directory stored locally on the client computer.

  ## KDServer Interface

  In this form of the social network and the application, we consider that the server performs them
following functions:
  * Maintains a list of IP addresses and Ports that correspond to the clients' clientIDs
who participate in the social network and updates this list accordingly
changes occurring in social graphs.
  * Once a client has connected, the following diagnostic message should appear:
"Welcome client clientID".
* It must support multiple threads simultaneously by creating a separate thread for
every client it serves. You can assume that the maximum number of threads that
can create the server is given and is equal to 8.
  * Multiple clients can simultaneously communicate with the server and have
access to their respective files Profile_XclientID and Others_XclientID (Implementation in
2ND PHASE).
  * It must implement the update and synchronization (from the client to the server and
vice versa) of Profile_XclientID and Others_XclientID files (Implementation in PHASE 2)
as soon as a post is posted by a client to its respective followers or
as soon as a client replies to a post of another client
  * It must implement the updating and synchronization of the client directories only
an upload/download is done.
  * It must implement the "search" function to search for a photo.
  * It must implement the send function to send a photo file
and accompanying text to a client.
  * Must keep statistics on which photo files are the most
popular and from which clients they are downloaded, and display them at the end
execution of the program (implementation of the statistics function). (Implementation in 2nd
PHASE.


  ## MultiThreading

  Each client will have an identifier (clientID).
* Client A accesses the file Profile_XclientА ("locks" the file), which
is on the server and the server starts a timer.
* After the timer expires, if client A has not "unlocked" the file, it is displayed
on the screen diagnostic warning message that the server will "unlock" it
file.
Client A saves the file and "unlocks" it.
* The server tries to access the Profile_XclientA file to read them
his posts and to inform the Others of his followers If client A already has
access to the file, a diagnostic message appears e.g. "The file is locked!" to
server (Implementation in PHASE 2). .
* Also, a client B tries to access the file Profile_XclientА to
read his posts. If client A already has access to the file, one is displayed
diagnostic message e.g. "The file is locked!" to client B (Implementation in PHASE 2).
* The server maintains a directory (in order of priority) with the clientIDs of all clients
who tried to access the file when it was "locked" by him
Client A.
* As soon as the file is released, the server and/or another client gains access to it
file.

  ## Notes

The clients and the server will communicate via sockets. Connecting to sockets is essentially one
connecting two programs over a network. The sockets implementation from your java.net
allows you to manage sockets as a data exchange channel. There are two classes
in the java.net package, Socket and ServerSocket, which are used by the client and the
server respectively. A typical flow with sockets is described as follows:

* The server will listen on a ServerSocket, which is associated with a specific port.
* Each client will use a socket to connect to the ServerSocket. The client must
already know the hostname of the computer running ServerSocket as well as the port
ServerSocket number.
* The ServerSocket will accept the client connection and there will now be one available
special socket it can use to communicate with the client.
* The client and server will now be able to communicate by reading and writing via
of their sockets.
