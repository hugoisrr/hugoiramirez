---
title: "Display Spinner while loading data"
date: "2020-03-20"
coverImage: "../images/loadingSpinner.png"
path: "/spinner-loading"
author: "Hugo I. Ramirez"
excerpt: "Usually these operations are perform with libraries such as fetch/ajax. For the user might be cumbersome when he clicks a bottom or performs an action, he expects a response instead not action is shown or data is display, only after few seconds"
tags: ["react", "react hooks"]
---

## Why should I use a Spinner?

If your web application uses an API to fetch data from a server, these operations are asynchronous, which means that it takes some amount of time to fetch the data, meanwhile your web application can load other parts or work in other processes. Usually these operations are perform with libraries such as fetch/ajax. For the user might be cumbersome when he clicks a bottom or performs an action, he expects a response instead not action is shown or data is display, only after few seconds, however we can solve this issue by adding a validation a verify if the data has been loaded, then show the data, if not show a Spinner or loading message indicating that it might take some time.

So what are the steps to perform a Spinner in React using React Hooks:

- Tracks any kind of async process based on promises (including async/await calls)
- Includes an HOC that can feed a loading component with the _loading_ true/false flag
- Can be used from any part of the code (component, plain javascript, redux...)
- Keeps track of parallel asynchronous calls
- Keeps track of errors
- Allows you to define multiple areas

## Spinner Component

This component can be imported by other components, it uses a Loading animation from bootstrap, the component is styled so that it will cover the with of the div parent.

```js
//Spinner that renders when undefined data comes in as a result of no connection in the websocket
import React, { Fragment } from "react";

export const Spinner = () => {
  const spinnerStyle = {
    width: "10rem",
    height: "10rem",
  };

  return (
    <Fragment>
      <div className="d-flex justify-content-center">
        <div className="spinner-border my-5" style={spinnerStyle} role="status">
          <span className="sr-only">Loading...</span>
        </div>
      </div>
    </Fragment>
  );
};
```

## Spinner imported and use in another component

```js
import { Spinner } from "../layout/Spinner";

// Verifies if there are Workstations and Data is been received, if not it renders a Spinner
if (wks.length > 0 && wksMap.size > 0) {
  return (
    <ContentAPI
      title={title}
      websocketStatus={websocketStatus}
      change={handleSwitch}
      dropdown
    >
      <div className="row">
        {currentDevice === null ? (
          <SelectDevice>Select Device</SelectDevice>
        ) : (
          <DeviceCard currentDevice={currentDevice} wks={wks} />
        )}
      </div>
    </ContentAPI>
  );
} else {
  return (
    <ContentAPI
      title={title}
      websocketStatus={websocketStatus}
      change={handleSwitch}
    >
      <Spinner />
    </ContentAPI>
  );
}
```
