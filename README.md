# redux-router-lite
---
[![Build Status](https://travis-ci.org/alessioscalici/redux-router-lite.svg?branch=master)](https://travis-ci.org/alessioscalici/redux-router-lite)
[![npm version](https://badge.fury.io/js/redux-router-lite.svg)](https://badge.fury.io/js/redux-router-lite)
---

A pure Redux-based, lightweight  router.

## Install

```npm i redux-router-lite```

(Make sure your project has ```redux``` installed).


## Features
- Purely Redux based. No React dependencies (but it's really easy to integrate with React).
- Easy to configure, just a single JSON object to define routes, in a common syntax.
- Listens to URL changes and updates the redux state accordingly.
- It uses route names to change the URL, allowing to change a route URL with minimal effort.
- It's fully compatible with Redux devtools time travel (It changes the URL too!)

## To integrate the router:

### 1. Create the route configuration JSON object

```js
{
  "mode": "browser", // history mode, can be 'hash' or 'browser'
  "routes": [
    { "name": "homePage", "path": "/" },
    { "name": "about", "path": "/about" },
    { "name": "user", "path": "/user/:id" },
  ],
  "notFoundRoute": { "name": "notFound", "path": "/404" }
}
```
- **mode** the history type. Can be 'hash' or 'browser' (default: 'browser')
- **routes** an array of objects representing the routes.
  - **name** the route unique name
  - **path** the route path pattern
- **notFoundRoute** the route to redirect to, if no route is matched

The **path** can contain parameters, following the [path-to-regexp](https://github.com/pillarjs/path-to-regexp) syntax. Some example:

###### Mandatory parameters

`/user/:id`
  - `/user/616` ⇒ `{ route: 'user', params: { id: '616'} }`
  - `/user` ⇒ no match

###### Optional parameters (?)

`/pictures/:viewMode?`
  - `/pictures/list` ⇒ `{ route: 'pictures', params: { viewMode: 'list'} }`
  - `/pictures` ⇒ `{ route: 'pictures', params: { viewMode: ''} }`

###### Custom regexp parameters
Use them to restrict a parameter to match a regular expression. E.g. to have the _id_ parameter to be a sequence of numbers:

`/user/:id(\\d+)` _(Note the double-escaped backslashes)_
  - `/user/12` ⇒ `{ route: 'user', params: { id: '12'} }`
  - `/user/abc` ⇒ no match

###### Unnamed parameters
Parameters with no name, the params key will be their position index. E.g.:

`/coords/(\\d+)/(\\d+)`
  - `/coords/12/34` ⇒ `{ route: 'coords', params: { '0': '12', '1': '34' } }`

###### Repeated parameters

Use * for 0 or more repetitions

`/users/:ids*`
  - `/users/john/jack/jim` ⇒ `{ route: 'users', params: { 'ids': 'john/jack/jim' } }`
  - `/users` ⇒ `{ route: 'users', params: { } }`


Use + for 1 or more repetitions

`/groupcall/:users+`
  - `/groupcall/john/jack/jim` ⇒ `{ route: 'groupcall', params: { 'users': 'john/jack/jim' } }`
  - `/groupcall` ⇒ no match


### 2. Add the router reducer to the app reducer

```js
import { combineReducers } from 'redux';
import { routerReducer as router } from 'redux-router';

export default combineReducers({
  router, // the default key is 'router'
});
```

The resulting state will be:

```js
{
  router: {
    location: {...},
    route: '...',
    params: {...},
    searchParams: {...},
  }
}
```

Note that the state key **must** be `router`. If for any reason you need to mount the router
state to a different path, you must use the `setRouterSelector` function, passing a selector
to the router state. E.g.

```js
import { combineReducers } from 'redux';
import { routerReducer, setRouterSelector } from 'redux-router';

// needed to set up the custom router key(s)
setRouterSelector(state => state.one.two);

export default combineReducers({
  one: combineReducers({
    two: routerReducer,
  }),
});
```

The resulting state will be:

```js
{
  one: {
    two: {
      location: {...},
      route: '...',
      params: {...},
      searchParams: {...},
    }
  }
}
```

### 3. Call the routerConfig passing the Redux store and the config objects

The `routerConfig` function needs the Redux store and the JSON routes config object. E.g.:

```js
import { createStore } from 'redux';
import rootReducer from 'app/reducer';
import { routerConfig } from 'redux-router';

// this can be defined elsewhere and imported
const routerConfig = {
  routes: [
    { name: 'home', path: '/' },
  ],
  notFoundRoute: { name: 'notFound', path: '/not-found' },
};


const store = createStore(rootReducer);

routerConfig(store, routerConfig);

```

### 4. React: Create a routing Component

Just create a React Component taking the route via props, and connect it to the state:

```js
import React from 'react';
import PropTypes from 'prop-types';

import { connect } from 'react-redux';
import { getRoute } from 'redux-router';

import showcase from 'react-showcase';
import LoginPage from 'ui/login/LoginPage';
import LoginForm from 'ui/login/LoginForm';

const MyRouter = ({ route }) => {
  switch (id) {
    case 'home':  return <MyHomePage />;
    case 'about': return <MyAboutPage />;
    default:      return <MyNotFoundPage />;
  }
};

MyRouter.propTypes = {
  route: PropTypes.string.isRequired,
};

const mapStateToProps = state => ({
  route: getRoute(state),
});

export default connect(mapStateToProps, null)(MyRouter);
```

### 5. Links (gotoRoute)


To redirect to a route programmatically (e.g. in response to an authentication error), the
`gotoRoute` function must be used. E.g.:


```js
import { gotoRoute } from 'redux-router';

// ... other code ...

  // resulting URL: /user/72
  gotoRoute('user', { id: '72' });

  // resulting URL: /user/72?view=profile
  gotoRoute('user', { id: '72' }, { view: 'profile' });
```
