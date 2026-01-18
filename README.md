# Real-Time Chat Application

A real-time chat application built with Spring Boot and WebSockets, demonstrating bidirectional communication between server and clients using the STOMP protocol over WebSocket.

## Overview

This project was developed as a learning exercise to understand how WebSockets work in Spring Boot and how to implement real-time, bidirectional communication in web applications. The application allows multiple users to join a chat room, send messages, and see when other users join or leave in real-time.

## How It Works

### Message Flow

1. **User Joins:**
    - Client connects to `/ws` endpoint via SockJS
    - Client sends JOIN message to `/app/chat.addUser`
    - Server stores username in session and broadcasts JOIN to `/topic/public`
    - All subscribers receive notification

2. **Sending Messages:**
    - Client sends CHAT message to `/app/chat.sendMessage`
    - Server broadcasts message to `/topic/public`
    - All subscribers (including sender) receive the message

3. **User Leaves:**
    - WebSocket session disconnects (browser close, network issue, etc.)
    - `SessionDisconnectEvent` is triggered
    - Event listener retrieves username from session
    - Server broadcasts LEAVE message to `/topic/public`


![Demo image](src/main/resources/static/images/demo-image.png)

### Architecture Pattern

This application follows a simple message broker pattern:
- **Clients** publish messages to application destinations (`/app/*`)
- **Controller** processes and routes messages
- **Broker** distributes messages to topic subscribers (`/topic/*`)
- **All subscribed clients** receive the broadcasted messages

## Tech Stack

**Backend:**
- Java 17
- Spring Boot 4.0.1
- Spring WebSocket
- STOMP (Simple Text Oriented Messaging Protocol)
- Lombok

**Frontend:**
- Vanilla JavaScript
- SockJS (WebSocket fallback)
- STOMP.js
- HTML5/CSS3

## Features

- Real-time messaging between multiple users
- User join/leave notifications
- Username-based identification with color-coded avatars
- WebSocket connection with SockJS fallback for browser compatibility
- Session management with automatic disconnect handling

## What I Learned

### 1. **WebSocket Fundamentals**

WebSockets provide full-duplex communication channels over a single TCP connection, enabling real-time data exchange between client and server without the overhead of traditional HTTP polling.

### 2. **STOMP Protocol**

STOMP (Simple Text Oriented Messaging Protocol) is a messaging protocol that works over WebSocket. It provides:
- A frame-based protocol with commands like CONNECT, SEND, SUBSCRIBE
- Message routing through destinations (similar to topics in pub/sub systems)
- Simplified message handling compared to raw WebSocket connections

### 3. **Spring WebSocket Configuration**

Implemented in `WebSocketConfig.java:11`, the configuration involves:

**STOMP Endpoint Registration:**
```java
registry.addEndpoint("/ws").withSockJS();
```
- Creates a WebSocket endpoint at `/ws`
- SockJS provides fallback options for browsers that don't support WebSocket

**Message Broker Configuration:**
```java
registry.setApplicationDestinationPrefixes("/app");
registry.enableSimpleBroker("/topic");
```
- `/app` prefix: Routes messages to `@MessageMapping` annotated methods
- `/topic` prefix: Simple in-memory message broker for broadcasting to subscribers

### 4. **Message Handling with Controllers**

The `ChatController.java:12` demonstrates two key patterns:

**Broadcasting Messages:**
```java
@MessageMapping("/chat.sendMessage")
@SendTo("/topic/public")
```
- Messages sent to `/app/chat.sendMessage` are processed and broadcast to all subscribers of `/topic/public`
- Simple pub/sub pattern for chat messages

**Session Management:**
```java
@MessageMapping("/chat.addUser")
```
- Stores username in WebSocket session attributes using `SimpMessageHeaderAccessor`
- Session attributes persist across the WebSocket connection lifecycle

### 5. **Event-Driven Architecture**

The `WebSocketEventListener.java:18` implements Spring's event-driven model:

**Disconnect Event Handling:**
```java
@EventListener
public void handleWebSocketDisconnectListener(SessionDisconnectEvent event)
```
- Automatically triggered when a WebSocket session disconnects
- Retrieves username from session attributes
- Broadcasts leave notification to all connected clients using `SimpMessageSendingOperations`

This approach decouples disconnect logic from the controller layer and demonstrates Spring's event system.

### 6. **Client-Side WebSocket Integration**

The frontend (`main.js:1`) implements:

**Connection Flow:**
1. Creates SockJS connection: `new SockJS('/ws')`
2. Wraps with STOMP client: `Stomp.over(socket)`
3. Connects and subscribes to `/topic/public`
4. Sends username to server on connection

**Message Types:**
- `JOIN`: User joined the chat
- `LEAVE`: User left the chat
- `CHAT`: Regular chat message

The client handles all three message types differently, showing event messages versus chat messages with avatars.

### 7. **Key Concepts**

- **Bidirectional Communication:** Unlike REST APIs, WebSocket maintains an open connection for instant two-way data flow
- **Pub/Sub Pattern:** Publishers send messages to topics; subscribers receive all messages from subscribed topics
- **Session State Management:** Storing user context in WebSocket sessions for connection lifecycle management
- **Graceful Fallbacks:** SockJS provides alternative transports (long-polling, etc.) when WebSocket isn't available
- **Message Broadcasting:** Using `SimpMessageSendingOperations` to send messages to all connected clients

## Project Structure

```
src/main/java/com/sebastjan/realtimechat/
├── chat/
│   ├── ChatController.java           # Message routing and handling
│   ├── ChatMessage.java              # Message model
│   └── MessageType.java              # Enum for message types
├── config/
│   ├── WebSocketConfig.java          # WebSocket and STOMP configuration
│   └── WebSocketEventListener.java   # Event handling for connect/disconnect
└── RealTimeChatApplication.java      # Spring Boot main class

src/main/resources/static/
├── index.html                         # Chat UI
├── css/main.css                       # Styling
└── js/main.js                         # Client-side WebSocket logic
```

## Acknowledgments

This project was built as a hands-on learning exercise to understand WebSocket implementation in Spring Boot. 
The project was completed with the help of Ali Bouali [YouTube](https://www.youtube.com/@boualiali)
