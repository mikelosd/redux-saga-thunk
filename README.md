# redux-saga-async-action

[![Generated with nod](https://img.shields.io/badge/generator-nod-2196F3.svg?style=flat-square)](https://github.com/diegohaz/nod)
[![NPM version](https://img.shields.io/npm/v/redux-saga-async-action.svg?style=flat-square)](https://npmjs.org/package/redux-saga-async-action)
[![Build Status](https://img.shields.io/travis/diegohaz/redux-saga-async-action/master.svg?style=flat-square)](https://travis-ci.org/diegohaz/redux-saga-async-action) [![Coverage Status](https://img.shields.io/codecov/c/github/diegohaz/redux-saga-async-action/master.svg?style=flat-square)](https://codecov.io/gh/diegohaz/redux-saga-async-action/branch/master)

Dispatching an action handled by redux-saga returns promise

## Install

    $ npm install --save redux-saga-async-action

## Basic setup

Add `middleware` and `saga` to your redux configuration:

```js
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'
import { middleware as asyncMiddleware, saga as asyncSaga } from 'redux-saga-async-action'

// add middleware
const sagaMiddleware = createSagaMiddleware()
const store = createStore({}, applyMiddleware(sagaMiddleware, asyncMiddleware))

// add saga
sagaMiddleware.run(asyncSaga)
```

## Usage

Add `meta.async` to your actions and receive `key` on response actions:
```js
const resourceCreateRequest = data => ({
  type: 'RESOURCE_CREATE_REQUEST',
  payload: data,
  meta: {
    async: 'RESOURCE_CREATE'
    ^
  }
})

const resourceCreateSuccess = (detail, key) => ({
                                       ^
  type: 'RESOURCE_CREATE_SUCCESS',
  payload: detail,
  meta: {
    async: { name: 'RESOURCE_CREATE', key }
                                      ^
  }
})

const resourceCreateFailure = (error, key) => ({
                                      ^
  type: 'RESOURCE_CREATE_FAILURE',
  error: true, // flux standard action default
  payload: error,
  meta: {
    async: { name: 'RESOURCE_CREATE', key }
                                      ^
  }
})
```

`redux-saga-async-action` will automatically transform your request action and inject `async.key` into it.

Handle actions with `redux-saga` like you normally do, but you'll need to grab `key` from the request action and pass it to the response actions:

```js
// worker saga
function* createResource(data, { async }) {
                                 ^
  try {
    const detail = yield call(api.post, '/resources', data)
    yield put(resourceCreateSuccess(detail, async.key))
                                            ^
  } catch (e) {
    yield put(resourceCreateFailure(e, async.key))
                                       ^
  }
}

// watcher saga
function* watchResourceCreateRequest() {
  while (true) {
    const { payload, meta } = yield take('RESOURCE_CREATE_REQUEST')
                     ^
    yield call(createResource, payload, meta)
                                        ^
  }
}
```

Dispatch the action from somewhere. Since that's being intercepted by `asyncMiddleware` cause you set `meta.async` on the action, dispatch will return a promise.

```js
store.dispatch(resourceCreateRequest({ title: 'foo' })).then((detail) => {
  // detail is the action payload property
  console.log('Yay!', detail)
}).catch((error) => {
  // error is the action payload property
  console.log('Oops!', error)
})
```

## Usage with selectors

To use `isPending` and `hasFailed` selectors, you'll need to add the `asyncReducer` to your store:

```js
import { combineReducers } from 'redux'
import { reducer as asyncReducer } from 'redux-saga-async-action'

const reducer = combineReducers({
  async: asyncReducer,
  // your reducers...
})
```

Now you can use selectors on your containers:

```js
import { isPending, hasFailed } from 'redux-saga-async-action'

const mapStateToProps = state => ({
  loading: isPending(state, 'RESOURCE_CREATE'),
  error: hasFailed(state, 'RESOURCE_CREATE')
})
```

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

### isPending

Tells if an action is pending

**Parameters**

-   `state` **State** 
-   `name` **([string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)>)** 

**Examples**

```javascript
const mapStateToProps = state => ({
  fooIsPending: isPending(state, 'FOO'),
  fooOrBarIsPending: isPending(state, ['FOO', 'BAR']),
  anythingIsPending: isPending(state)
})
```

Returns **[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 

### hasFailed

Tells if an action has failed

**Parameters**

-   `state` **State** 
-   `name` **([string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Array](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)>)** 

**Examples**

```javascript
const mapStateToProps = state => ({
  fooHasFailed: hasFailed(state, 'FOO'),
  fooOrBarHasFailed: hasFailed(state, ['FOO', 'BAR']),
  anythingHasFailed: hasFailed(state)
})
```

Returns **[boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean)** 

## License

MIT © [Diego Haz](https://github.com/diegohaz)
