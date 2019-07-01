# Getting Started with Twitch Extensions

## What are Twitch Extensions?

[Twitch Extensions](https://twitch.tv/p/extensions) are programmable interactive overlays and panels, which help streamers interact with their viewers. The Twitch community can interact in new ways such as heat maps, real-time game data overlays, mini-games, music requests, and leaderboards. If you haven't seen an Extension, check out the [TwitchDev channel page](https://www.twitch.tv/twitchdev) and view the Twitter panel Extension below the video player.

## What Are We Building?

This guide will cover the fundamentals of building your first Extension. It will allow Twitch viewers to cycle through colors via a visual element added on top of the video player (we call this a component overlay). On the frontend, a user clicks a button in this component that changes the color of a circle. Instead of changing the CSS locally, it makes a call to a backend server for a new hex value (we call this an Extension Backend Service or EBS for short).

## What's the Twitch Developer Rig?
<img src="https://github.com/twitchdev/extension-getting-started/blob/master/Extension-Rig-Banner.png" width="45%">

The recommended path for building this sample is with the Twitch Developer Rig. This is a native application to help with your development. The following steps will help you download, install, and set up the Rig.

## Let's Get Started!

## Setup

1. Download the Developer Rig for [Mac](https://link.twitch.tv/2u8BNNm), [Windows](https://link.twitch.tv/2JcsuGQ), or [Linux](https://link.twitch.tv/2F8hFjw). Then, open the downloaded file to install the Rig.
2. Visit the Extensions section of you [Developer Dashboard](https://dev.twitch.tv/dashboard/extensions). You will need to have a Twitch account to login and two factor authentication (2FA) will need to be activated. You will see a button prompting you with instructions if 2FA has not yet been activated.
3. Click the "Create Extension" button on this page and you will be taken to a form to setup your new Extension.
4. Make sure you select "Video - Component" under the "Type Of Extension" section, check the box to indicate that you are using the Developer Rig, and fill out the remaining fields.
5. Click "Create Extension" at the bottom of the form and you will be taken to a status page for the first version of this Extension.
6. On this page, click the "Overview" link along the top tabs. You'll see an "About This Extension" section on the right of this page (if not, you may need to widen your browser window or look for a menu icon that has a "Product Info" option). Copy and paste the `Client ID` into a text editor; you'll need this in a few steps.
7. Click the "Settings" tab and then the "Secret Keys" menu option along the left. Hover over your key, then copy and paste it into a text editor as well for the next few steps.

## Running Hello World

1. Open the Developer Rig, click "Add Project" at the top left and then "Create Project."
2. Fill in the `Client ID` and `Secret` you found in the above steps.
3. Click the "Import" button to get your Extension's information.
4. Give the project a name if it was not pulled automatically (e.g. "Hello World") and select a directory for the project files.
5. Below this, make sure "Use Existing Example" is selected. For this example, also make sure to select "Hello World" on the right side to automatically start with this repository's code.
6. Click "Save."
7. The Rig will host your frontend and backend logic for the Extension. To do this, click "Host with Rig" under "Host your front-end files." Then click "Activate" under "Back-end Run Command" to run the backend.
8. Now the app should be ready for testing! Click on "Extension Views" in the left menu. This simulates a broadcaster's channel. Let's add a component overlay view to see our Extension by clicking "Create New View."
9. Ensure the view type is "Component" and give it a label such as "main". Click "Save."
10. You should now see the Hello World Extension running on a simulated video player. Experiment with the Extension by cycling through colors with the button within the component.

## A Closer Look

Now that you've built the app, let's take a look under the hood and explore how it works.

### Extension Architecture

1. Extension Frontend – comprised of HTML files for the different Extension views as well as corresponding JavaScript and CSS files. The frontend for the Hello World Extension you are currently running has the following functionality:
    * A button and script (`viewer.js`) that makes a POST call to the EBS to request a color change for the circle.
    * A GET call when the Extension is initialized to change the circle to the current color stored on the EBS.
2. Extension Backend – An EBS that performs the following functionality:
    * Spins up a simple HTTPS server with a POST handler for changing color.
    * Validates an Extension JWT.
    * Returns a new color using the `/cycle/color` endpoint.

### Frontend

Let's dive into the frontend components. The HTML files allow this Extension to be run as any [Extension type](https://dev.twitch.tv/docs/extensions/required-technical-background/#types-of-extensions): overlay, component, or panel. In this example, we are using a component so `video_component.html` will be rendered.

The frontend logic is handled by `viewer.js`. The two core functions are handling authentication and making GET/POST requests to our EBS. On first load, `twitch.onAuthorized` enables the button, sets our auth token, and dispatches the GET request to retrieve the initial color.

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

When the viewer presses the button, the onClick handler creates a POST request to the `/color/cycle/` endpoint. On a succesful response, `updateBlock()` is called passing the payload which contains a new hex value. `updateBlock()` simply renders the new hex value using CSS.

`$('#color').css('background-color', hex);`

### Backend

Our backend logic is contained in `backend.js`. Using [hapi](https://hapijs.com/), we are able to spin up a light webserver. Hapi handles hosting our GET endpoint `/color/query` and POST endpoint `/color/cycle`. These endpoints then route to either `colorCycleHandler` or `colorQueryHandler`.

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
* Read our [Extensions documentation](https://dev.twitch.tv/docs/extensions) and continue developing with the Twitch Developer Rig.
* Join our communities on [Twitter](https://twitter.com/twitchdev), [Discord](https://discordapp.com/invite/G8UQqNy) and the [Forums](https://discuss.dev.twitch.tv/) for help and to stay updated!
