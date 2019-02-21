# Twitch Getting Started with Extensions


## Extensions Overview

Twitch Extensions are programmable interactive overlays and panels, which help streamers interact with their viewers. The Twitch community can interact in new ways such as heat maps, real-time game data overlays, mini-games, music requests, and leaderboards. If you haven't seen an Extension, check out the [TwitchDev channel](https://www.twitch.tv/twitchdev) and view the Twitter Panel Extension beneath the video display.

## Getting Started

This guide will cover the fundamentals of building your first Extension. It will allow viewers to cycle through display colors via a component overlay. On the frontend, a user clicks a button that changes the color of a circle. Instead of changing the CSS locally, it calls its Extension Backend Service (EBS) for a new hex value.  

<img src="https://github.com/twitchdev/extension-getting-started/blob/master/Extension-Rig-Banner.png" width="45%">

## Developer Rig 

The recommended path for building this sample is with the [Developer Rig](https://github.com/twitchdev/developer-rig). The Developer Rig allows Extensions developers to develop and test Extensions quickly, easily, and locally. It is a lightweight web app that runs in a browser and lets you test with production APIs and hosted assets on Twitch. 


## Setup

1. Download [Mac OSX](https://s3-us-west-2.amazonaws.com/developer-rig-install-update-development/Twitch+Developer+Rig-0.9.8.dmg) || [Windows](https://s3-us-west-2.amazonaws.com/developer-rig-install-update-development/Twitch+Developer+Rig+Setup+0.9.8.exe) and install the Rig.  
2. Create an Extension in your [Developer Dashabord](https://dev.twitch.tv/dashboard/extensions)
3. After clicking, "Create Extension" give it a name and click the Video - Component checkbox under Type of Extension.
4. Select you are using the Developer Rig and fill in the remaining details. 
5. Create the Extension.
6. Record the `clientID` and `secretID` (can be found within settings). 


## Running Hello World

1. Open the Rig and create a new Extension project.
2. Fill in the `clientID` and `secretID`.
3. Give the project a name and select a new directory.
(insert based on our chat with Chris)
4. For this example, select "Getting Started" to automatically pull down this repo. 
5. The rig will host your frontend and backend logic. To do this, press "Host with Rig" under Host your front-end files. Then press "Activate" to run the backend. 
6. Now the app should be ready for testing! Click on the "Extension Views" tab. This simulates a broadcaster's channel. Let's add an overlay view to see our Extension. 
7. Ensure the view type is component and give it a label such as "main". Click Save.
8. Play with the Extension by cycling through colors.


## Building the App

### Extension Architecture

1. Extension Frontend -- comprised of HTML files for the different Extension views and corresponding JavaScript files and CSS. The frontend has the following functionality:
    * A button and script (`viewer.js`) that makes a POST call to the EBS to request a color change for the circle.
    * A GET call when the Extension is initialized to change the circle to the current color stored on the EBS.
2. Extension Backend -- EBS that performs the following functionality:
    * Spins up a simple HTTPS server with a POST handler for changing color
    * Validates an Extension JWT
    * Returns a new color using the `/cycle/color` endpoint


## Frontend

Let's dive into the frontend components. The HTML files allow this Extension to be run as any [Extension type](https://dev.twitch.tv/docs/extensions/required-technical-background/#types-of-extensions): overlay, component or panel. In this example, we are using a component so `video_component.html` will be rendered. 

The frontend logic is handled by `viewer.js`. The core functions here are 1) handing authentication 2) making GET/POST requests to our EBS. On first load, `twitch.onAuthorized` enables the button, sets our auth token and dispatches the GET request to retrieve the inital color.

```
twitch.onAuthorized(function (auth) {
  // save our credentials
  token = auth.token;
  tuid = auth.userId;
  // enable the button
  $('#cycle').removeAttr('disabled');
  setAuth(token);
  $.ajax(requests.get);
});

``` 

When the viewer presses the button, the onClick hanlder creates a POST request to the `/color/cycle/` endpoint. On succesful response, `updateBlock()` is called passing the payload which contains a new hex value. `updateBlock()` simply renders the new hex value using CSS.

`$('#color').css('background-color', hex);`

## Backend

Our backend logic is contained in `backend.js`. Using `[hapi`](https://hapijs.com/), we are able to spin up a light webserver. Hapi handles hosting our GET endpoint `/color/query` and POST endpoint `/color/cycle`. These endpoints then route to either `colorCycleHandler` or `colorQueryHandler`. 

Hapi makes this mapping easy: 

```
(async () => {
  // Handle a viewer request to cycle the color.
  server.route({
    method: 'POST',
    path: '/color/cycle',
    handler: colorCycleHandler
  });

  // Handle a new viewer requesting the color.
  server.route({
    method: 'GET',
    path: '/color/query',
    handler: colorQueryHandler
  });

```

`colorCycleHandler` and `colorQueryHandler` have similar logic, but the `colorCycleHandler` additonally produces a new hex value. First, these functions authenticate the request with `verifyAndDecode`. Then we use the `channelColors` array (indexed by `channelID`) to store hex values. After producing a new color, we save it back to the array and return the value to the frontend.

```
function colorCycleHandler (req) {
  // Verify all requests.
  const payload = verifyAndDecode(req.headers.authorization);
  const { channel_id: channelId, opaque_user_id: opaqueUserId } = payload;

  // Store the color for the channel.
  let currentColor = channelColors[channelId] || initialColor;

  // Rotate the color as if on a color wheel.
  verboseLog(STRINGS.cyclingColor, channelId, opaqueUserId);
  currentColor = color(currentColor).rotate(colorWheelRotation).hex();

  // Save the new color for the channel.
  channelColors[channelId] = currentColor;

  return currentColor;
}
```


## Next Steps
* Read our Extension documentation and contiune developing with the Rig
* View our next chapters: adding Extensions to your channel, PubSub, etc...


