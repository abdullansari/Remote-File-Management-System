# Remote File Management Protocol

A remote file management system is crucial for client-server communications. Why is it crucial? Well that is because it: centralizes data access and storage, oversees access control, maintains security, and oganizes files to make a structured storage and facilitate retrieval.

The Remote File Management Protocol (RFMP) provides the same services but in a more simplified manner. It has three phases: _setup_, _operation_, and _closing phase_.

![filetransfer](https://github.com/user-attachments/assets/b1c5b520-e6e4-43b4-b606-85c9ec80cdc2)

_Source: Pinterest._

##  Setup Phase

The setup phase is just about laying down the foundations for the client-server communication. Once the client connects to the server, the application sends the following packets in this order:
1) **Start packet (client to server):** sends the starting packet containing information about the packet type, protocol name, protocol version, secured-communication respectively and is depicted as `(SS,RFMP,v1.0,1)`. The packet type, SS, is _start_, _RFMP_ is the protocol name, _v1.0_ is the protocol version, and the _1_ means that the client wants secured communication. If the client wishes for secured communication, the application sends an _encryption-packet_ immediately after the SS packet. However, if the client does not wish for secure communication then a _0_ will be placed in place of the _1_ and no additional packet will be sent immediately after the SS packet.
2) **Confirm-Connection packet (server to client):** sends the confirmation packet telling the client to start sending information packets. Note that the structure for this packet is `(CC)` if security is not required, otherwise it is `(CC,<Server_public_key>)`. The server's public key is generated by RSA algorithm and will be used to decrypt the _Session Key_ which will be explained in a moment.
3) **Encryption packet (client to server):** sends only if the _start_ packet specifies for secured communication. It contains information about the chosen encryption algorithm and credentials used to secure communication. Its structure is `(EC, Algorithm, <Session_key>, <Client_public_key>)`.

_Before we proceed further, let's clarify a few things about how the keys specified in the packets are generated as well as the type of encryption algorithm_ 🔐.

### Key Generation & Encryption Specification

The RFMP secured communication consists of 4 keys and 2 ciphers.

**KEYS** 🔑
- Server public/private keys.
- Client public/private keys.

**CIPHERS** 📋
- AES cipher.
- Caesar cipher.

Now, onto how these keys get generated. Evidently, the server and client(s) produce their respective keys from their own ends. If the client chooses AES encryption then 16 random bytes are generated by the `get_random_bytes()` function within the `Crypto.Random` library-16 bytes because that is the standard key length for AES encryption and then that becomes the _Session Key_. On the other hand, if the client chose Caesar encryption then a random number between 1 and 26 is generated which then serves as the _Session Key_. See Fig 1 below for a visual explanation.

![image](https://github.com/user-attachments/assets/749c9394-275b-4c0d-83cc-4caca3c131fa)

_Fig 1: How Session Keys are encrypted for different encryption algorithm choices._


##  Operation Phase

Now that the foundation of the communication has been established as well as the manner in which it will be held, we move onto the operation phase. In the operation phase, the client picks what _command_ to give to the server to perform. The structure of the packet is `(CM, <Type of command>)`. The RFMP has the following file commands options:
1. `mkdir` - Create directory
2. `cd` - Change directory
3. `rmdir`/`rd` - Remove directory
4. `del` - Delete file
5. `ren` - Rename file/directory
6. `openRead` - Read file contents
7. `openWrite` - Write to file
8. `cp` - Copy file/directory
9. `chmod` - Change file permissions
10. `chown` - Change file ownership
11. `find` - Search for files
12. `ls` - show all files/ folders
13. `Exit`

To prevent errors and to provide a better user experience, the client simply has to enter the number of the service it wishes for. For example, if the client wants to create a directory using the `mkdir` command, it simply needs to enter _1_ into the terminal and the RFMP will understand.

In cases for commands that require text input such as the `openWrite` command, the client(s) can simply type the content it wishes to write into a file, which it will define, and then the RFMP will encrypt it (if secured communication is chosen) using the algorithms the client(s) had previously picked. This operation would be sent via a **Data Packet** which has the structure `(DP,<content>)`.

Furthermore, after each successful transmission of a packet, the server returns a **Successful packet** with the structure `(SC)` otherwise an **Error packet** with the structure `(EC)`. The RFMP is a robust protocol and if an error is encountered mid-operation, it will efficiently handle it due its extensive error handling and reporting capabilities. Additionally, if an error occurs, the RFMP informs the client exactly about what type of error was encountered.

##  Closing Phase

Now that the client(s) has performed all file operations that it wanted to, it wishes to terminate the connection with the server and exit the communication. For this case, the client simply needs to enter _13_ into the terminal and a `Exit` **Close packet** will be sent to the server with the structure `(End)` to confirm that the client has finished using the application and the server will expect no more messages from the client. 

That is the functionality and scope of the RFMP in complete. Try it out for yourself and have fun!
