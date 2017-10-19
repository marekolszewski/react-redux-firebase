# Firestore

To correctly begin using firestore, make sure you have the following:
* `v2.0.0-beta.11` or higher (most like to use `next` tag in package.json)
* `firestore` imported with `import 'firebase/firestore'`
* `firestore` initialize with `firebase.firestore()`
* `firestoreReducer` added to your reducers (will be combinable with main before v2.0.0 release)

Should look something similar to:

```js
import { createStore, combineReducers, compose } from 'redux'
import firebase from 'firebase'
import 'firebase/firestore' // add this to use Firestore
import {
  reactReduxFirebase,
  firebaseStateReducer,
  firestoreReducer
} from 'react-redux-firebase'

const firebaseConfig = {
  apiKey: '<your-api-key>',
  authDomain: '<your-auth-domain>',
  databaseURL: '<your-database-url>',
  storageBucket: '<your-storage-bucket>'
}

// react-redux-firebase config
const rrfConfig = {
  userProfile: 'users',
  // useFirestoreForProfile: true // Firestore for Profile instead of Realtime DB
}

// initialize firebase instance
firebase.initializeApp(config) // <- new to v2.*.*

// initialize Firestore
firebase.firestore()

// Add reduxReduxFirebase enhancer when making store creator
const createStoreWithFirebase = compose(
  reactReduxFirebase(firebase, rrfConfig),
)(createStore)

// Add Firebase to reducers
const rootReducer = combineReducers({
  firebase: firebaseStateReducer,
  firestore: firestoreReducer
})

// Create store with reducers and initial state
const initialState = {}
const store = createStoreWithFirebase(rootReducer, initialState)
```

## Profile

If you would like to have your users profiles go to Firestore instead of Real Time Database, you can enable the
`useFirestoreForProfile` option when making store creator like so:

```js
// react-redux-firebase config
const rrfConfig = {
  userProfile: 'users',
  useFirestoreForProfile: true // Firestore for Profile instead of Realtime DB
}
```

## Queries

Firestore queries can be created in two ways:

* [Automatically](#firestoreConnect) - Using `firestoreConnect` HOC (manages mounting/unmounting)
* [Manually](#manual) - Using `setListeners` or `setListener` (requires managing of listeners)

### Automatic {#firestoreConnect}

`firestoreConnect` is a React Higher Order component that manages attaching and detaching listeners for you as the component mounts and unmounts. It is possible to roll a similar solution yourself, but can get complex when dealing with advanced situations (queries based on props, props changing, etc.)

#### Examples
1. Basic query that will attach/detach as the component passed mounts/unmounts. In this case we are setting a listener for the `'todos'` collection:

  ```js
  import { compose } from 'redux'
  import { connect } from 'react-redux'
  import { firestoreConnect } from 'react-redux-firebase'

  export default compose(
    firestoreConnect(['todos']), // or { collection: 'todos' }
    connect((state, props) => ({
      todos: state.firestore.ordered.todos
    }))
  )(SomeComponent)
  ```

2. Create a query based on props by passing a function. In this case we will get a specific todo:

  ```js
  import { compose } from 'redux'
  import { connect } from 'react-redux'
  import { firestoreConnect } from 'react-redux-firebase'

  export default compose(
    firestoreConnect((props) => [
      { collection: 'todos', doc: props.todoId } // or `todos/${props.todoId}`
    ]),
    connect(({ firestore: { ordered } }, props) => ({
      todos: ordered.todos && ordered.todos[todoId]
    }))
  )(SomeComponent)
  ```

## Manual {#manual}

If you want to trigger a query based on a click or mange listeners yourself, you can use `setListener` or `setListeners`. When doing this, make sure you call `unsetLister` for each listener you set.

##### Component Class

```js
import React, { Component } from 'react'
import PropTypes from 'prop-types'
import { connect } from 'react-redux'

class Todos extends Component {
  static contextTypes = {
    store: PropTypes.object.isRequired
  }

  componentWillMount () {
    const { firebase } = this.context.store
    firebase.setListener('todos')
    // firebase.setListener({ collection: 'todos' }) // or object notation
  }

  componentWillUnmount() {
    const { firebase } = this.context.store
    firebase.unsetListener('todos')
    // firebase.unsetListener({ collection: 'todos' }) // or object notation
  }

  render () {
    return (
      <div>
        {
          todos.map(todo => (
            <div key={todo.id}>
              {JSON.stringify(todo)}
            </div>
          ))
        }
      </div>
    )
  }
}

export default connect((state) => ({
  todos: state.firestore.ordered.todos
}))(Todos)
```

##### Functional Components

It is common to make react components "stateless" meaning that the component is just a function. This can be useful, but then can limit usage of lifecycle hooks and other features of Component Classes. [`recompose` helps solve this](https://github.com/acdlite/recompose/blob/master/docs/API.md) by providing Higher Order Component functions such as `withContext`, `lifecycle`, and `withHandlers`.

```js
import { compose, withHandlers, lifecycle } from 'recompose'
import { connect } from 'react-redux'

const withStore = compose(
  withContext({ store: PropTypes.object }, () => {}),
  getContext({ store: PropTypes.object }),
)

const enhance = compose(
  withStore,
  withHandlers({
    onDoneClick: props => (key, done = false) =>
      props.store.firestore.update('todos', key, { done }),
    onNewSubmit: props => newTodo =>
      props.store.firestore.add('todos', { ...newTodo, owner: 'Anonymous' }),
  }),
  lifecycle({
    componentWillMount(props) {
      props.store.firestore.get('todos')
    }
  }),
  connect(({ firebase }) => ({ // state.firebase
    todos: firebase.ordered.todos,
  }))
)

export default enhance(SomeComponent)
```

For more information [on using recompose visit the docs](https://github.com/acdlite/recompose/blob/master/docs/API.md)

## storeAs {#populate}

`storeAs` is not yet supported for the Firestore integration, but will be coming soon.

## Populate {#populate}

Populate is not yet supported for the Firestore integration, but will be coming soon.

## More Info {#more}

The Firestore integration is build on [`redux-firestore`](https://github.com/prescottprue/redux-firestore).