# CTSYNC Server

The server is implemented in C and uses the `select` system call to manage
multiple TLS/TCP connections efficiently in a non-blocking manner.

After establishing a secure TLS session over a TCP socket, the server enters
a continuous loop where it monitors for incoming connections and data. As
data arrives, it is incrementally read and stored in a buffer until a specific
sequence is detected. Once this sequence is recognized, a callback function is
invoked with the accumulated data, after which the application processes the
received content.

## Application data in SSL instance
Each TLS connection instance stores some application-specific data to track
information such as authentication status, authorization status, and the
device-to-TLS connection map.


## Clipboard Text Transfer Process

![Flow Diagram](https://fileserver.ismailbiswas.com/ctsync/clipboard-transfer.svg)
1.**Monitoring the Clipboard:**  
Each device runs a background task that checks the system clipboard every few seconds for any changes.

2.**Notifying the Server:**  
When a change is detected, the device contacts the server. The server then determines which devices are allowed to receive the new clipboard content.

3.**Preparing Recipient Information:**  
The server creates a JSON object listing the eligible devices. For each device, it includes:

- The device's unique ID.
- The device's public key (used for encryption).

4.**Encrypting and Signing the Clipboard text:**  
The device that detected the clipboard change:

   - Generates a random AES key and uses it to encrypt the clipboard content.
   - Uses the recipient device's public key from the JSON to encrypt the randomly generated AES key and initialization vector.
   - Signs the encrypted AES key and initialization vector with its own private key to ensure authenticity.

5.**Encoding and Sending Data:**  
Both the encrypted content and the signature are converted into base64-encoded strings. These strings, along with the recipient's device ID, are sent back to the server.

6.**Transferring the Clipboard text:**  
Finally, the server forwards the encrypted clipboard content to the intended recipient device.


# Backend overview
![Flow Diagram](https://fileserver.ismailbiswas.com/ctsync/back-end-data-process.svg)
