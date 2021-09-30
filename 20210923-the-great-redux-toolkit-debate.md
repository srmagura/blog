# The Great Redux Toolkit Debate

An [offhand comment](https://dev.to/srmagura/comment/1idl8) I wrote one day while eating lunch sparked an unexpected and interesting debate with [Mark Erikson](https://twitter.com/acemarke), one of the maintainers of Redux.

[Redux](https://redux.js.org/) has long been the go-to library for managing global state in React applications. [Redux Toolkit](https://redux-toolkit.js.org/), which Mark helped create, is a relatively new library that aims to be the "official, opinionated, batteries-included toolset for efficient Redux development." This post will go into my thoughts on the benefits and potential drawbacks of Redux Toolkit.

## Why Redux is Awesome

1. **It's unopinionated.** Redux requires you to put your global state in a store, and to manage that state via reducers and actions. An action is a simple JavaScript object with a `type` property, and a reducer is a pure function that transforms the old state into the new state based on an action. Beyond this, everything else is up to you.
2. **It has a minimal API surface.** Redux only has [5 top-level exports](https://redux.js.org/api/api-reference), and only one of those, `createStore`, is essential.
3. **It's extremely versatile.** Do you want your store to contain only the ID of the current user? Or do you want your store to track the state of every entity, page, widget, and input in your massive enterprise app? Whatever your use case, Redux and its large ecosystem have you covered.

## Why Redux is Hard

Redux is hard for the same reasons it is awesome.

1. **It's unopinionated.** Redux doesn't tell you how to structure your application's state, reducers, or actions, so you have to make your own decisions.
2. **It has a minimal API surface.** You'll quickly realize you need more than just `createStore` to create a useful application with Redux. A prime example of this is the need to fetch data from an API in response to an action.
3. **It's extremely versatile.** There are so many different frontend architectures that are possible with Redux that it's easy to get lost. It took me a long time to figure out how Redux fit into the React applications I was building.

## Redux Toolkit to the Rescue

Redux Toolkit aims to eliminate first two of these pain points by providing an opinionated, convenient, and beginner-friendly approach to Redux development. Its features include:

- `createAction` — lets you define action creators, similar to [typesafe-actions](https://github.com/piotrwitek/typesafe-actions). I'm a TypeScript die-hard so type safety is non-negotiable. 😆
- `createReducer` — allows you to write a reducer without a `switch` statement. Uses [Immer](https://immerjs.github.io/immer/) under the hood. Immer is amazing and you should use it in your reducers even if you don't plan to use Redux Toolkit.
- `createSlice` — a powerful helper that allows you to define both the reducer and actions for a slice of your state in one fell swoop.
- `createAsyncThunk` — allows you to start an API call in response to an action and dispatch another action when the call completes.
- `createEntityAdapter` — returns a set of prebuilt reducers and selector functions for performing CRUD on an entity.
- RTK Query — a **library** for fetching and caching server state in your Redux store. Can be compared to [React Query](https://react-query.tanstack.com/) which aims to solve the same problems, but in a different way.

## My Review of the Redux Toolkit (RTK) API

### Overall Recommendation

- If you're new to Redux, use RTK, but don't feel like you need to utilize all of its features. You can do a lot with just `createAction` and `createReducer`.
- If you're already using Redux and Immer, there's no reason you have to switch to Redux Toolkit. Only use it if you agree with its opinionated approach.

### `createAction`

Not a new idea but a useful one nonetheless. Currently, typesafe-actions seems to be more powerful than RTK in this area because the typesafe-actions `getType` function correctly types `action.payload` in `switch` reducers. The [`ActionType`](https://github.com/piotrwitek/typesafe-actions#actiontype) type helper is really nice too. I'd like to see RTK reach parity with typesafe-actions in this domain.

If you can figure out how to write a type safe `switch` reducer with RTK, let me know!

### `createReducer`

As I said previously, Immer is really great. But Immer already works perfectly with `switch` reducers so I don't see a huge benefit to `createReducer`.

### `createSlice`

I have a few concerns here. I like how in the traditional approach to Redux, you define your actions separately from your reducers. This separation of concerns allows you to lay out the operations your user can perform without getting bogged down in how those operations are implemented. `createSlice` eschews this separation and I'm not sure that's a step in the right direction.

### `createAsyncThunk`

By including `createAsyncThunk` in Redux Toolkit, the Redux team has made thunks the officially-recommended side effect model for Redux. I like how Redux itself is unopinionated regarding side effects, so I have mixed feelings about the built-in support for thunks.

Of course, you can still use other side effect models like sagas and observables alongside Redux Toolkit. I'm a big fan of [Redux Saga](https://redux-saga.js.org/), which makes it straightforward to integrate Redux with your backend API while also enabling you to write incredibly powerful asynchronous flows. Sagas are written using generator functions and `yield` which does take some getting used to.

Mark tells me that sagas can be overkill for common use cases and thunks fit better here. I can see this point of view, but I still find sagas to be more intuitive and will be sticking with them.

### `createEntityAdapter`

I'm concerned that `createEntityAdapter` could lead to designs that are overly CRUD-centric, favoring basic `add`, `update`, and `remove` actions over more meaningful, descriptive actions that are tailored to each entity. I'll admit that I don't fully understand the use case here. If `createEntityAdapter` saves you from writing tons of duplicate code, by all means use it.

### RTK Query

RTK Query really is a separate library that happens to live in the same package as Redux Toolkit. I think it would be better as a separate package, but that's just me. Fortunately, RTK Query is exported from a separate entry point, so it will never be included in your bundle if you don't use it.

RTK Query seems complex and unintuitive to me, but my opinion might change if I tried it out. If you're looking for an alternative, you might like [React Query](https://react-query.tanstack.com/). I also evaluated the similar [SWR](https://swr.vercel.app/) library but found it lacking some features that my team uses constantly.

## Mark's Response to my Claim that RTK is Overly Opinionated

[Read the full comment here!](https://dev.to/markerikson/comment/1ieni) In summary:

> As you can see, all of these APIs have a common theme:
>
> - These are things people were always doing with Redux, and have been taught in our docs
> - Because Redux didn't include anything built-in for this purpose, people were having to write this code by hand, or create their own abstractions
> - So we created standardized implementations of all these concepts that people could _choose_ to use _if they want to_, so that people don't _have_ to write any of this code by hand in the future.

## What I Use in My Applications

### My Last 4 React Web Apps

These are all medium-size single-page apps written entirely in React.

- Redux is used for about 10% of the overall application state, with local component state making up the other 90%. We deliberately only use Redux for state that needs to stay in memory when navigating between screens, for example information about the current user.
- We constructed our actions and reducers with typesafe-actions, Immer, and `switch` statements, whether using Redux or `useReducer`.
- A simple custom-made `useQuery` hook is used to fetch data from the backend. This data ends up in the local state of our `Page` components.
- There's a dash of Redux Saga to support login and persisting changes to complex order drafts that the user creates.

### My React Native App

This app has to work offline so it made sense to put the majority of the app's state in Redux.

- Redux Saga is responsible for all the interaction with the backend API. This worked out quite well. There's a pretty complex saga for sending queued operations to the backend when a user comes back from being offline.
- The entire Redux store is persisted using [redux-persist](https://www.npmjs.com/package/redux-persist). This is still magic to me 😂.
- Local component state is used for forms.

### My Next React Web App

New projects are always exciting because they give you the chance to rethink your architecture and tech stack. Going forward, we will:

- Stick with typesafe-actions and `switch` reducers. It was a close call between this and switching to Redux Toolkit's `createAction` and `createReducer`.
- Replace our homegrown `useQuery` with React Query. As a result, some state that we would have previously put in Redux will now be stored automatically in React Query's cache.
- Continue to use Redux Saga in a few places.

## Further Reading

- [Mark Erikson: Redux Toolkit 1.0](https://blog.isquaredsoftware.com/2019/10/redux-starter-kit-1.0/)
- [The Redux Toolkit documentation](https://redux-toolkit.js.org/)

## Self-Promotion

- Check out my new library, [real-cancellable-promise](https://dev.to/srmagura/announcing-real-cancellable-promise-gkd)!
- I'll be working on a new major version of [react-loading-skeleton](https://github.com/dvtng/react-loading-skeleton). [Check out the roadmap here!](https://github.com/dvtng/react-loading-skeleton/issues/106)
