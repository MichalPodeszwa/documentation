---
title: "VueX"
metaTitle: "VueX"
metaDescription: "VueX plugin for OpenReplay."
---

This plugin allows you to capture `VueX` mutations/state and inspect them later on while replaying session recordings. This is very useful for understanding and fixing issues.

## Installation
```bash
npm i @openreplay/tracker-vuex
```

## Usage
Initialize the `@openreplay/tracker` package as usual and load the plugin into it. Then put the generated plugin into your `plugins` field of your store.

### If your website is a Single Page Application (SPA)

```js
import Vuex from 'vuex'
import OpenReplay from '@openreplay/tracker';
import trackerVuex from '@openreplay/tracker-vuex';
//...
const tracker = new OpenReplay({
  projectKey: PROJECT_KEY
});
const vuexPlugin = tracker.use(trackerVuex(<options>));  // check list of available options below

tracker.start();

const store = new Vuex.Store({
  //...
  plugins: [vuexPlugin],
});
```
### If your web app is Server-Side-Rendered (SSR)

Follow the below example if your app is SSR. Ensure `tracker.start()` is called once the app is started (in `useEffect` or `componentDidMount`).

```js
import Vuex from 'vuex'
import OpenReplay from '@openreplay/tracker/cjs';
import trackerVuex from '@openreplay/tracker-vuex/cjs';
//...
const tracker = new OpenReplay({
  projectKey: PROJECT_KEY
});
const vuexPlugin = tracker.use(trackerVuex(<options>));  // check list of available options below

cosnt store = new Vuex.Store({
    //...
    plugins: [vuexPlugin],
  });
}
```


## Options

You can customize the plugin behavior with options to sanitize your data. They are similar to the ones from the standard `createLogger` plugin.

```js
trackerVuex({
  filter (mutation, state) {
    // returns `true` if a mutation should be logged
    // `mutation` is a `{ type, payload }`
    return mutation.type !== "aBlacklistedMutation";
  },
  transformer (state) {
    // transforms the state before logging it.
    // for example return only a specific sub-tree
    return state.subTree;
  },
  mutationTransformer (mutation) {
    // mutations are logged in the format of `{ type, payload }`
    // we can format it any way we want.
    return mutation.type;
  },
})
```

## Troubleshooting

Having trouble setting up this plugin? please connect to our [Slack](https://slack.openreplay.com) and get help from our community.