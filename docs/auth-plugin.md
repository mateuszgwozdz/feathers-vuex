---
title: Auth Plugin
---

The Auth module assists setting up user login and logout.

## Setup

See the [Auth Setup](/getting-started.html#auth-plugin) section for an example of how to setup the Auth Plugin.

## Breaking Changes in 2.0

The following breaking changes were made between 1.x and 2.0:

- The `auth` method is now called `makeAuthPlugin`.

## Configuration

You can provide a `userService` in the auth plugin's options to automatically populate the user upon successful login.

## State

It includes the following state by default:

```js
{
  accessToken: undefined, // The JWT
  payload: undefined, // The JWT payload

  userService: null, // Specify the userService to automatically populate the user upon login.
  entityIdField: 'userId', // The property in the payload storing the user id
  responseEntityField: 'user', // The property in the payload storing the user
  user: null, // Deprecated: This is no longer reactive, so use the `user` getter. See below.

  isAuthenticatePending: false,
  isLogoutPending: false,

  errorOnAuthenticate: undefined,
  errorOnLogout: undefined
}
```

## Getters

Two getters are available when a `userService` is provided to the `makeAuthPlugin` options.

- `user`: returns the reactive, logged-in user from the `userService` specified in the options. Returns `null` if not logged in.
- `isAuthenticated`: a easy to remember boolean attribute for if the user is logged in.

## Actions

The following actions are included in the `auth` module.  Login is accomplished through the `authenticate` action.  For logout, use the `logout` action.  It's important to note that the records that were loaded for a user are NOT automatically cleared upon logout.  Because the business logic requirements for that feature would vary from app to app, it can't be baked into Feathers-Vuex.  It must be manually implemented.  The recommended solution is to simply refresh the browser, which clears the data from memory.

- `authenticate`: use instead of `feathersClient.authenticate()`
- `logout`: use instead of `feathersClient.logout()`

If you provided a `userService` and have correctly configured your `entityIdField` and `responseEntityField` (the defaults work with Feathers V4 out of the box), the `user` state will be updated with the logged-in user.  The record will also be reactive, which means when the user record updates (in the users service) the auth user will automatically update, as well.

> Note: The Vuex auth store will not update if you use the feathers client version of the above methods.

## Example

Here's a short example that implements the `authenticate` and `logout` actions.

```js
export default {
  // ...
  methods: {

    login() {
      this.$store.dispatch('auth/authenticate', {
        email: '...',
        password: '...'
      })
    }

    // ...

    logout() {
      this.$store.dispatch('auth/logout')
    }

  }
  // ...
}
```

Note that if you customize the auth plugin's `namespace` then the `auth/` prefix in the above example would change to the provided namespace.

## Automatic login with stored token

After a user logs in, their auth token will be saved in either localStorage or as a cookie, depending on your settings. But when the page refreshes or reloads, application memory is cleared and the user session is not available anymore. In this section we will grab the localStorage or cookie and automatically log the user back in, before the page renders.

Create a new file in `~/plugins/authInit.js`. This plugin will authenticate the user automatically for us. 

```
// ~/plugins/authInit.js
// a) LOCAL STORAGE OPTION
// see if a token is available in localstorage
const storedToken = typeof localStorage['feathers-jwt']

// b) COOKIE OPTION
// see if a token is available in localstorage or cookie
/*
import { CookieStorage } from 'cookie-storage'

const cookieStorage = new CookieStorage()
const storedToken = cookieStorage.getItem('feathers-jwt')
*/

// see if token is available in URL - this is used when redirecting
// from oAuth strategies from the api
const hashTokenAvailable = window.location.hash.indexOf('access_token' > -1)

export default async (context) => {
  // if token is available authenticate the user
  if (
    (!context.app.store.state.auth.user && storedToken) ||
    hashTokenAvailable
  ) {
    await context.app.store
      .dispatch('auth/authenticate')
      .then(() => {})
      .catch((e) => {
        console.error(e)
      })
  }
}
```

Activate the plugin in `nuxt.config.js`

```
  plugins: [
    { src: '~/plugins/authInit.js', ssr: false }
  ],
```
Now when you refresh the page, you will be authenticated before page is rendered. 
