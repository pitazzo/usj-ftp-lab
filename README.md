# FTP Lab 👨🏻‍🔬

This lab is part of the course **Networks & Communications 2** of **Universidad San Jorge** of Zaragoza. All contents are original.

## Intro

The objective of this lab is to play at a low level with the FTP protocol to see and "believe" that the principles covered in the theoretical classes of the course are real and functional. There are certain parts of the lab that may not work depending on the system they are executed on, as the behavior of local networks is highly dependent on the specific system. In this case, the important thing is to understand why what is failing is failing, document it in the deliverable, and if there is no obvious solution, continue working with a peer.

### Delivery

The result of the lab is a PDF file with the stated questions answered briefly and concisely in English, including screenshots if requested. If parts of the lab have been done in a group, it must be indicated. If the entire lab has been done in a group due to technical limitations, a single PDF file must be submitted.

It is allowed to use generative AI tools such as ChatGPT for information seeking and research, but if done, it must be specified. The document will be subjected to a heuristic check to determine if it has been generated with generative AI. If so and not explicitly indicated, it will result in the lab being graded with "0".

### Prerequisites

To complete the lab, it is necessary to:

- Have a functional installation of Wireshark.
- Have access to some type of command line system where the `telnet` package is available. A backup option for Windows users could be [Putty](https://www.putty.org).
- Have [netcat](https://en.wikipedia.org/wiki/Netcat) installed as a command line tool.

### Note for Windows users

All the commands provided in this guide have been tested in UNIX systems, not Windows. In order to get them to work in Windows systems, modifications may apply, specifically regaring the `nc` syntax and the usage of pipes and redirections (`>`). As last resort, WSL can be used but should not be necessary. Another option could be using Putty in advance mode.

## **1. Installation and Configuration**

### **1.1 Download and Install FileZilla Server**

1. Download FileZilla Server from [its official website](https://filezilla-project.org/download.php?type=server).
2. Run the installer and follow the recommended steps.
3. Make sure the service starts automatically at system startup.
4. After the installation is complete, open FileZilla Server.

### **1.2 Download and Install FileZilla Client**

1. Download FileZilla Client from [its official website](https://filezilla-project.org/download.php?type=client).
2. Install the software and verify that it opens correctly.

### **1.3 Initial Server Configuration**

1. Open FileZilla Server and go to `Server > Configure`.
2. Under **Server Listeners**, remove `::1` to ensure the server only listens on IPv4.
3. Under **Protocol Settings**, select **Explicit FTP over TLS and insecure plain FTP**.
   > ⚠️ _This method is insecure and should not be used in production._
4. Under **Passive Mode**, enable the **Custom port range** option and enter the port range `5000-5100`.
5. Save the configuration.

### **1.4 Creating Users**

1. Go to `Rights Management > Users` and click `Add`.
2. Enter your username and a secure password.
3. Check the **User is enabled** box.
4. Enable the **Require password to log in** option.
5. Add at least one **mount point**, associating it with a local folder.
6. Grant **read and write** permissions to the assigned folder.
7. Save the changes.

### **1.5 Initial Test**

1. Open FileZilla Client.
2. Enter the server details:

- **Host:** `127.0.0.1`
- **User:** The one you created in the previous step.
- **Password:** The one you configured.
- **Port:** `21`

3. Connect and verify that you can list the files in the assigned directory.
4. Look at the server logs and answer:

- Do the messages make sense?
- Are there any errors or warnings?

## **2. Connect Manually with Telnet**

1. Open a terminal and run:

```bash
telnet <SERVER_IP> 21
```

2. Try logging in with incorrect credentials:

```bash
USER nonexistent_user
PASS bad_password
```

3. Try logging in successfully:

```bash
USER <your_username>
PASS <your_password>
```

4. Run the following commands and observe the responses:

```bash
PWD
CWD <directory>
QUIT
```

5. Answer these questions:

- What error code did you receive when authentication failed? Investigate and explain the numbering.
- What connection is being used to run these commands?
- Why do we only need one connection?
- In what cases might it be preferable to interact with the server via the command line instead of FileZilla?

## **3. Capturing Traffic with Wireshark**

### **3.1 Capturing FTP Traffic**

1. Open Wireshark and select the corresponding network interface.
2. Apply the following filter:

```plaintext
ftp
```

3. Repeat the previous steps and capture the trace.
4. Run the capture again, then connect and browse with Filezilla Client.
5. Browse through some directories and create a folder.
6. Attach a capture of the traces in both cases.
7. Answer these questions:

- What did you see in each case?
- Does it make sense?
- Can you find the encrypted frames corresponding to the commands sent from Wireshark? Investigate why.

## **4. Commands with a Data Connection**

Now let's test some commands that require a data connection. To do this, we'll use active and passive modes. For testing purposes, we'll use the data connection to transmit the results of a LIST command.

### **4.1 Active Mode**

We'll start with active mode.

1. Get your private IP address with:

```bash
ipconfig # Windows
ifconfig # Linux/macOS
```

2. Start a listening server on your machine, for example, on port `2025`:

```bash
nc -l 2025
```

3. From FTP, run the `PORT` command with your IP and port in `X1,X2,X3,X4,X5,X6` format, adjusting the values ​​as appropriate. For the first four bytes, use the first four bytes of your private IP address, and for the last two, use port `2025` in base 256.
4. Run `LIST` on FTP and check if `nc` receives the data.
5. Answer these questions:

- Were you able to receive the results of `LIST`? Attach a screenshot.
- In what cases might it be preferable to use active mode?
- What should you keep in mind when specifying the connection port to the server? Is any port valid?
- On a home network, what steps would you need to follow to make active mode work?

### 4.2 Using PASV and Capturing the Port

1. Start a new telnet session, authenticate as before, and enter passive mode:

```bash
PASV
```

2. Calculate the port the server is telling you to connect to by reconstructing it from base 256.
3. Start a listening server on your machine on the specified port:

```bash
telnet localhost <rebuilt port>
```

4. Run LIST in FTP and view the data via telnet.
5. Answer these questions:
6. Did everything arrive correctly? Attach a screenshot.
7. In what case might this connection mode be more appropriate?

## **5. Uploading and Downloading Files with Netcat (`nc`)**

Now we will proceed to upload and download the same file from the server. This would make little sense in practice, but it's very interesting to run a complete flow.

### **5.1 Uploading a file in passive mode**

1. Create a text file with test content, for example, a text file with notepad.
2. Start a new telnet session and authenticate.
3. Create a test folder with:

```
MKD test
```

> ⚠️ _Note that depending on how you configured the mount point, you may not have write permissions_ 4. Navigate inside with:

```
CWD test
```

5. Verify that you are where you think you are with the command:

```
PWD
```

6. Activate passive mode using the command and rebuild the port from base 256:

```
PASV
```

7. Tell the server that you are uploading a file with the `STOR` command:

```
STOR <remote-filename>
```

8. From the client, send the file over the data connection:

```bash
nc localhost <rebuilt-port> < <local-filename>
```

9. Close the control connection with `QUIT`

### **5.2 Download the file in active mode**

1. Start a new telnet session and authenticate.
2. Navigate to the file location using the `CWD` and `PWD` commands if necessary.
3. Start a listener with netcat on the port of your choice:

```bash
nc -l <port> > <local-filename>
```

4. Activate passive mode using the `PORT` command, passing your private IP address in the first 4 bytes and the base 256 port in the last two:

```
PORT <X1,X2,X3,X4,X5,X6>
```

5. Request the file download with the `RETR` command:

```
RETR <server-filename>
```

6. Answer these questions:

- Can you arbitrarily choose any port?
- Did the file arrive in the format you expected? Why?
- What would happen with a binary file instead of a text file?

## **6. Sharing Files Between Machines**

### **6.1 Creating a User for a Colleague**

1. On FileZilla Server, create a new user for your colleague.
2. Assign them a directory on the mount point you prefer and grant them read and write permissions.
3. Take note of their private IP address and they'll take note of yours.

### **6.2 Transferring Files**

1. Ask them to repeat the steps in section **5.1** to upload a file to your server using passive mode. Then do the same and upload a file.
2. Try repeating the steps in section **5.2** to download it using active mode.
3. Answer these questions:

- Which step were you able to complete and which weren't?
- Why?
- What additional configurations would you need to make to make everything work?
