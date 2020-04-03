---
title: "Store data from WebSocket in useContext"
path: "/websocket-usecontext"
date: "2020-04-02"
coverImage: "../images/listening.jpg"
author: "Hugo I. Ramirez"
excerpt: "With the increment of devices an application in IoT, we have found an increment of applications that are using WebSockets, but how do we define when to use this technology."
tags: ["react", "react hooks"]
---

## When should we use WebSocket?

With the increment of devices an application in IoT, we have found an increment of applications that are using WebSockets, but how do we define when to use this technology. Thus we have to define specific scenarios:

- **Constant Updates** Imagine the following scenario, within a line of production, there are several workstations with automated machines or with PLC's that are sending constant data but also are sending several alerts and updates of their status. As a manager or data analyst, we need to process this data in real-time, without any request or petitions such as with _http_. Websockets are a good solution especially if the client doesn't know the amount of data and when can be received.
- **Fast Feedback** Several PLC's models have digital or analog I/O, if a particular instruction needs to be executed in real-time, the client needs to know if that particular instruction was executed successfully or not, thus within the UI of our application feedback of the user's action needs to be reflected. Compared to REST, WebSocket allows a higher amount of efficiency, because they do not require the request/response overhead in _http_.