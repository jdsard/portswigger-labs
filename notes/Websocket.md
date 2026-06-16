
# Lab 1

## Description

This online shop has a live chat feature implemented using WebSockets.

Chat messages that you submit are viewed by a support agent in real time.

To solve the lab, use a WebSocket message to trigger an `alert()` popup in the support agent's browser.

## Solution

Head to live chat and turn interception on. Write a message and change it's content to:

```
<img src="x" onerror="alert(1)">
```

This solves the lab.

# Lab 2

## Description

