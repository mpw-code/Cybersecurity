## OSI model

Open System Interconnection

<img width="1226" height="1283" alt="image" src="https://github.com/user-attachments/assets/80b1bdd5-7f97-4ab8-bc09-4a62ede7eb50" />

## The 7 layers

- Layer 7 – Application: Provides network services directly to applications, such as HTTP, DNS, and SMTP.
- Layer 6 – Presentation: Handles data formatting, encryption, decryption, and compression.
- Layer 5 – Session: Establishes, manages, and terminates communication sessions.
- Layer 4 – Transport: Provides end-to-end communication using protocols such as TCP and UDP.
- Layer 3 – Network: Uses IP addresses and routing to move packets between networks.
- Layer 2 – Data Link: Uses MAC addresses and frames for communication within a local network.
- Layer 1 – Physical: Transmits raw bits through cables, fiber, or wireless signals.

Remember ;-) Please Do Not Throw Sausage Pizza Away
---
- Encapsulation when data goes DOWN the model from 7 (Application) to 1 (Physical).
- De-encapsulation when data goes UP the model from 1 (Physical) to 7 (Application).

## Application (Layer 7) - software on your machine
Are not the GUI applications. Application protocols. Applications sits on top of this Layer 7.  
Application protocols: SMTP, SNMP, HTTP, FTP, LDP, IMAP, Telnet, POP3, NNTP, EDI

## Presentation (Layer 6) - software on your machine
Format the data. Data conversion. Compression. Encryption and decryption.  
Formats: ASCII, EBCDIC, TIFF, JPEG, MPEG, MIDI  

## Session (Layer 5) - software on your machine
Provides logical connection between machines. Creates, monitors, shutdown the session.  
Full duplex: send and receive data at the same time  
Half duplex: send and receive data NOT at the same time. Wait between messages.  
Simplex: Can only send (like a satelite)  
Network File Systemen (NFS), NetBIOS  

## Transport (Layer 4)
When data has arrived it falls into the Transport layer.
Busy layer. Has to check for error. UDP en TCP.
UDP= connection less. Best effort. TCP - oriented and reliable.
Also SSL, TSL, TLS
TCP and UDP ports 0-1023 / 1024-49151 / ..

## Network (Layer 3) - movement of data
Move data between 2 hosts not physically connected
Use logical address IP
IP is at this layer
IPSec Internet Protocol Security

## Data Link (Layer 2) - movement of data


## Physical (Layer 1) - movement of data





