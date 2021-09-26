Hi! I'm Sam, a senior software developer at [Interface Technologies](http://www.iticentral.com/).

Today I'm announcing the public release of [`real-cancellable-promise`](https://github.com/srmagura/real-cancellable-promise), a simple but robust cancellable promise library for JavaScript and TypeScript.

`real-cancellable-promise` solves two key problems that I've encountered in every React app I've ever written:

## Problem 1: setState after unmount

If you try to update your component's state after it has unmounted, you'll get

> Warning: Can't perform a React state update on an unmounted component. This is a no-op, but it indicates a memory leak in your application. To fix, cancel all subscriptions and asynchronous tasks in a useEffect cleanup function.

This can happen, for example, if your component starts an API call but the user navigates away before the API call completes. React tells you to "cancel all asynchronous tasks" but doesn't tell you _how_ to do it. That's where `real-cancellable-promise` comes in.

**The `CancellablePromise` class from `real-cancellable-promise` is just like a normal promise, except it has a `cancel` method.** You can use the `cancel` method as the cleanup function in a `useEffect` to cancel your API call and prevent the setState after unmount warning.

```ts
useEffect(() => {
  const cancellablePromise = listBlogPosts()
    .then(setPosts)
    .catch(console.error);

  return cancellablePromise.cancel;
}, []);
```

## Problem 2: Queries with variable parameters

API calls often have parameters that can change. A `searchUsers` API method might take in a search string and return users whose name matches that string. You can implement a React UI for this like:

```tsx
function searchUsers(searchTerm: string): Promise<User[]> {
  // call the API
}

export function UserList() {
  const [searchTerm, setSearchTerm] = useState("");
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    searchUsers(searchTerm).then(setUsers).catch(console.error);
  }, [searchTerm]);

  return <div>...</div>;
}
```

But there are two issues here:

1. If the API calls complete in a different order than they were initiated in, your UI shows the wrong data.
2. If the search term changes while an API call is in progress, the in-progress API call is allowed to complete even though its result is now irrelevant. This wastes bandwidth and server resources.

_(Also in a real app you would definitely want to debounce `searchTerm`, but that's another topic.)_

`real-cancellable-promise` resolves both issues by allowing you to cancel the in-progress API call when the search term changes:

```ts
useEffect(() => {
  const cancellablePromise = searchUsers(searchTerm)
    .then(setUsers)
    .catch(console.error);

  return cancellablePromise.cancel;
}, [searchTerm]);
```

### But I'm using [React Query](https://react-query.tanstack.com/)!

The `useQuery` hook from React Query has many advantages over making API calls in a `useEffect` like I showed in the previous example. React Query already handles API calls returning in the wrong order, but isn't able to abort the HTTP request without your help. `real-cancellable-promise` has you covered here — React Query will automatically call the `cancel` method of `CancellablePromise` when the query key changes. [(Reference)](https://react-query.tanstack.com/guides/query-cancellation)

## How do I get started?

[Head on over to the README on GitHub](https://github.com/srmagura/real-cancellable-promise) for instructions on integrating your HTTP library with `real-cancellable-promise` and for more detailed examples.

## Not just for React

I built `CancellablePromise` to solve problems I encountered in React development, but the library is not tied to React in any way. `real-cancellable-promise` is also tested in Node.js and React Native and should provide value in frontend applications built with other frameworks like Vue and Angular.

## The story behind the code

While this is the initial public release of the library, older versions of `CancellablePromise` have been used in production at Interface Technologies for over 3 years! It's one of the foundational components in our family of packages that enable us to deliver stable and user-friendly React apps quickly.

Previous implementations of `CancellablePromise` were designed specifically to work with `async-await` and didn't have good support for traditional Promise callbacks via `then`, `catch`, and `finally`. The new `CancellablePromise` supports everything that normal Promises do, and the nice thing is that your promise stays cancellable no matter what you throw at it:

```ts
const cancellablePromise = asyncOperation1()
  .then(asyncOperation2)
  .then(asyncOperation3)
  .catch(asyncErrorHandler)
  .finally(cleanup);

cancellablePromise.cancel(); // Cancels ALL the async operations
```

## Prior art

There are other libraries that enable Promise cancellation in JavaScript, namely [p-cancelable](https://www.npmjs.com/package/p-cancelable) and [make-cancellable-promise](https://www.npmjs.com/package/make-cancellable-promise).

`make-cancellable-promise` is limited in that it doesn't provide the facility to cancel the underlying asynchronous operation (often an HTTP call) when `cancel` is called. It simply prevents your callbacks from running after cancellation occurs.

`p-cancelable` does let you cancel the underlying operation via the `onCancel` callback, but the library's API is limited compared to `real-cancellable-promise` in that

- `then`, `catch`, or `finally` return a normal, non-cancellable Promise and,
- There is no support for returning a cancellable Promise from `Promise.all`, `Promise.race`, and `Promise.allSettled`. `real-cancellable-promise` provides these via `CancellablePromise.all`, `CancellablePromise.race`, and `CancellablePromise.allSettled`.

## Stability

**`real-cancellable-promise` has been tested extensively and is ready for production!** The new `CancellablePromise` will be rolling out to one of our production apps next week, and our other apps will be updated soon after.

## Issues

Please post any issues you encounter in the GitHub repository.
