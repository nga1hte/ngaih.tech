---
layout: post
title: "TCP - Port Scanner"
slug: "tcp"
date: "2023-07-29"
comments: false
categories:
  - hacking
tags:
  - tcp
  - ports 
  - scanner
  - tool
---

## What is TCP ?
Transmission Control Protocol (TCP) is one of the core communication protocols which is used to establish and maintain network connections on the internet and other computer networks. It enables the transfer of data and communication between devices. TCP is connection oriented ensuring the orderly and error free delivery of data packets.

TCP works at the transport layer of both the OSI and TCP/IP model. Two protocols exists at the transport layer.
- TCP: Establishes connection before transmitting data, checks for errors, resend data if error exists or undelivered.  Used for data transfers.
-  UDP: User Datagram Protocol is a simple request-response communication, which allows for faster communication. It is mostly used in streaming datas.

TCP connections allows for full-duplex communication where data can be sent and received and sent simultaneously on both devices, enabling an efficient two way communication.

### PORTS
In networking, a port is a logical channel through which networked devices communicate and exchange data. It acts as a virtual endpoint for communication, allowing different application and services on a device to use the network to send and receive data. Ports numbers identify the different services and applications to avoid conflicts. Common ports are 80 (HTTP), 443 (HTTPS), 21 (FTP), 22 (SSH).

TCP uses port numbers to identify specific processes or services running on devices. This way, multiple applications can use TCP to communicate simultaneously, with each application having a unique port number.

![handshake](/images/tcp/tcp-com.png)

img: How the Three Way Handshake works.

### TCP connection in practise.
- **Establish a connection (Three-way handshake)**
	1. **SYN**: The client sends a SYN (Synchronize) packet to the server, indicating its desire to initiate a connection.
	2. **SYN-ACK**: The server responds with a SYN-ACK (Synchronize-Acknowledgment) packet, acknowledging the client's request and indicating its readiness to establish a connection.
	3. **ACK**: The client sends an ACK (Acknowledgment) packet back to the server, confirming the server's response. At this point, the connection is established, and data transmission can begin.  

-  **Transmitting Data**
	Once connection has been established, data can be transferred between the client and the server reliably. TCP ensures that data is delivered in the correct order and without errors by using:
	1.  **Sequence numbers**: Each byte of data is assigned a unique sequence number and upon receiving of data, they are arranged according to the sequence number.
	2.  **Acknowledgements**: After receiving of each data, an acknowledgement is sent back to the sender indicating that it is received, if no acknowledgement is received then data is re-transmitted.  

-  **Closing the connecting**
	When data transfer is completed, then connection is closed by sending closing packets and the port is freed up. The connection is terminated in the following ways.
	1. **FIN**: The device that wants to close the connection sends a FIN (Finish) packet to the other device, indicating its intention to terminate the connection.
	2. **ACK**: The other device acknowledges the FIN packet with an ACK packet.
	3. **FIN**: The other device sends its FIN packet, indicating its willingness to close the connection.
	4. **ACK**: The device acknowledges the FIN packet from the other device. At this point, the connection is fully closed.  


### Port Scanner
A port scanner is a tool that can be used to probe and identify open ports on target devices.  In a TCP connection a server responds with a **SYN/ACK**  packet to our **SYN** packet if the port is open. On receiving a **SYN/ACK** packet, it can be assumed that the port is open and available for connection.
A close port will send a **RST** (Reset packet) to our **SYN** packet and if a port is filtered or under a firewall, we will most likely not receive any outbound response to our **SYN** packet. 
A TCP port scanner is a tool that sends **SYN** packets to a server and decides whether the port is open, close or filtered by the response it receives.

### Building a port scanner in GO.

The *net* golang package provides a portable interface for network I/O, including TCP/IP, domain name resolution and unix domain sockets. We will utilise the *Dial* function to connect and establish a TCP server.

```go
// net.Dial function to connect to a server and establish a TCP connection
conn, err := net.Dial("tcp", "address:port")
```

We will create a listener port on our system using netcat.

```bash
nc -lp 420
```

Here is the rudimentary code that will find the first open port in our localhost after we open a listener port 420.

```go
// A program to find the first open port
package main  

import (
	"fmt"
	"net"
)

func main() {
	//Searching for open ports from 1 to 500
	for i := 1; i <= 500; i++ {
		addr := fmt.Sprintf("localhost:%d", i) // localhost ports
		conn, err := net.Dial("tcp", addr)
		if err != nil {
			continue
		}
	conn.Close()
	fmt.Printf("Port %d is open\n", i)
	}
}
```

#### Output
```bash
> go run main.go
port 420 is open.
```


#### A port scanner that can run concurrently.
We will utilise go routines to implement a faster port scanner, that can scan for multiple open ports.

```go
package main

import (
	"fmt"
	"net"
	"sort"
)

// takes two channel, listens for port and sends to results
func worker(ports <-chan int, results chan<- int) {
	for p := range ports {
		addr := fmt.Sprintf("localhost:%d", p)
		conn, err := net.Dial("tcp", addr)
		if err != nil {
			results <- 0 // sends 0 if port is not open
			continue
		}
		conn.Close() // closes port
		results <- p // sends port if open
	}
}

func main() {
	ports := make(chan int, 100)
	results := make(chan int)
	var openPorts []int //stores open port

	//send jobs to the workers
	for i := 0; i < cap(ports); i++ {
		go worker(ports, results)
	}

    // sends ports
	go func() {
		for i := 1; i <= 1024; i++ {
			ports <- i
		}
	}()

	// receive results
	for i := 0; i < 1024; i++ {
		port := <-results
		if port != 0 {
			openPorts = append(openPorts, port)
		}
	}

	close(ports)
	close(results)

	//sort the open ports
	sort.Ints(openPorts)

	//printing of open ports
	for _, port := range openPorts {
		fmt.Printf("Port %d is open\n", port)
	}
}
```

```bash
> nc -lp 69
> nc -lp 420
> nc -lp 999
```

#### OUTPUT
```bash
> go run main.go
port 69 is open
port 420 is open
port 999 is open
```

> Thanks to Black Hat Go book for the port scanner.