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

## Data from the Server

The first thing to do is to analyze the data that comes from the server. To improve the UX, both sides server and Front-End have to concur in a common API and stablish the structure of the API. In this example, the server is sending the current state of two workstations **0 and 1**. Once the connection between the server and the client is established the first array is sent with the id's, name and state keys of each workstation. Then for each state change, the server is sending an object with the workstation id and state key.

<!-- markdownlint-disable MD033 MD010 -->

```js
// JS code by PrismJS

[{"ws_id": 0, "ws_name": "engraving machine", "states": {"0": "null", "1": "spindle", "2": "engraving"}}, {"ws_id": 1, "ws_name": "drill", "states": {"0": "null", "1": "moving", "2": "drilling", "3": "playing"}}]

{"ws_id": 1, "state_key": 2}

{"ws_id": 0, "state_key": 2}
```

## Websocket Client

Within the Websocket Client the client establish connection to a server or a local machine, in this case is done through the port _4000_, then using the hook **useContext** we import the methods from the State context.

```js
import { useContext } from "react";
import { w3cwebsocket as W3CWebSocket } from "websocket";
import ModelContext from "../../context/model/modelContext";

export const client = new W3CWebSocket("ws://localhost:4000");

const ModelWebsocket = () => {
  const modelContext = useContext(ModelContext);
  const {
    setUpModelMap,
    setDataInModelMap,
    setModelStatesColors,
    getModelWebsocketStatus,
    getModelWks,
  } = modelContext;

  client.onerror = () => {
    try {
      console.error("Connection Error with WebSocket Model");
      getModelWebsocketStatus("ERROR");
    } catch (error) {
      console.error(error);
    }
  };

  client.onopen = () => {
    try {
      console.log("WebSocket Model Client Connected");
      getModelWebsocketStatus("CONNECTING");
    } catch (error) {
      console.error(error);
    }
  };

  client.onmessage = message => {
    try {
      message = JSON.parse(message.data);
      if (Array.isArray(message) && message.length > 0) {
        // Gets the initial array of workstations
        getModelWks(message);
        setUpModelMap(message);
        setModelStatesColors(message);
      } else if (typeof message === "object") {
        // Starts the dictionary structure and the data from the websocket
        setDataInModelMap(message);
        getModelWebsocketStatus("OPEN");
      }
    } catch (error) {
      console.error(error);
    }
  };

  return null;
};

export default ModelWebsocket;
```

_websocket.js_

## React Context

Exporting Context so it can be used in other components within the app.

```js
import { createContext } from "react";

const modelContext = createContext();

export default modelContext;
```

_modelContext.js_

## State

Here we set the initial state, where the workstations and data from the server in stored. Then we use the **useReducer** hook to pass the reducer and context to manipulate the state and dispatch the types. At the end of the file we pass the whole state and the methods that can be used in other components through the Context Provider. To store the data from different workstations, we used a **Map()** data structure, so we could access data in parallel.

```js
import React, { useReducer } from "react";
import ModelContext from "./modelContext";
import ModelReducer from "./modelReducer";
import {
  GET_MODEL_WKS,
  SET_MODEL_MAP,
  SET_MODEL_COLORS,
  GET_MODEL_WEBSOCKET_STATUS,
} from "../types";
import { getRandColor } from "../../assets/libs/helperFunctions";

const ModelState = props => {
  const initialState = {
    wks: [],
    wksMap: {},
    wksStatesColors: {},
    websocketStatus: "",
  };

  const [state, dispatch] = useReducer(ModelReducer, initialState);

  // Set up Map data structure with workstations ids as keys and 50 Arrays fill with -1 as values
  const setUpModelMap = wks => {
    try {
      if (wks.length > 0) {
        const wksMap = new Map();
        wks.forEach(workstation => {
          wksMap.set(workstation.ws_id, new Array(50).fill(-1));
        });
        dispatch({
          type: SET_MODEL_MAP,
          payload: wksMap,
        });
      }
    } catch (error) {
      console.error(error);
    }
  };

  // Creates a Map Data structure to store an object with an object holding all states and its corresponding color
  const setModelStatesColors = wks => {
    try {
      if (wks.length > 0) {
        const wksStatesColors = new Map();
        wks.forEach(workstation => {
          const colorStates = {};
          Object.values(workstation.states).forEach(state => {
            colorStates[state] = getRandColor(5);
          });
          wksStatesColors.set(workstation.ws_id, colorStates);
        });
        dispatch({
          type: SET_MODEL_COLORS,
          payload: wksStatesColors,
        });
      }
    } catch (error) {
      console.error(error);
    }
  };

  // Shift the first state_key values and push the income state_key values on its corresponding array, within the labeler map structure
  const setDataInModelMap = data => {
    try {
      if (state.wksMap.size > 0) {
        for (const [wks_id, stateKeyValuesArray] of state.wksMap.entries()) {
          if (data.ws_id === parseInt(wks_id)) {
            stateKeyValuesArray.shift();
            stateKeyValuesArray.push(data.state_key);
          }
        }
      }
    } catch (error) {
      console.error(error);
    }
  };

  // Get WebsocketStatus
  const getModelWebsocketStatus = status => {
    dispatch({
      type: GET_MODEL_WEBSOCKET_STATUS,
      payload: status,
    });
  };

  // Get Workstations from the API
  const getModelWks = wks => {
    dispatch({
      type: GET_MODEL_WKS,
      payload: wks,
    });
  };

  return (
    <ModelContext.Provider
      value={{
        wks: state.wks,
        wksMap: state.wksMap,
        wksStatesColors: state.wksStatesColors,
        websocketStatus: state.websocketStatus,
        setUpModelMap,
        setModelStatesColors,
        setDataInModelMap,
        getModelWebsocketStatus,
        getModelWks,
      }}
    >
      {props.children}
    </ModelContext.Provider>
  );
};

export default ModelState;
```

_modelState.js_
