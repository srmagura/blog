If you're a mid-level React developer looking to become an advanced React developer, this post is for you!

I've been reviewing React code written by junior and mid-level developers on a daily basis for a couple of years now, and this post covers the most common mistakes I see. I'll be assuming you already know the basics of React and therefore won't be covering pitfalls like "don't mutate props or state".

## Bad Habits

**Each heading in this section is a bad habit that you should avoid!**

I'll be using the classical example of a to-do list application to illustrate some of my points.

### Duplicating state

**There should be a single source of truth for each piece of state.** If the same piece of information is stored in state twice, the two pieces of state can get out of sync. You can try writing code that synchronizes the two pieces of state, but this is an error prone band-aid rather than a solution.

Here's an example of duplicate state in the context of our to-do list app. We need to track the items on the to-do list as well as which ones have been checked off. You could store two arrays in state, with one array containing all of the to-dos and the other containing only the completed ones:

```tsx
const [todos, setTodos] = useState<Todo[]>([]);
const [completedTodos, setCompletedTodos] = useState<Todo[]>([]);
```

But this code is buggy at worst and smelly at best! Completed to-dos are stored in the state twice, so if the user edits the text content of a to-do and you only call `setTodos`, `completedTodos` now contains the old text which is incorrect!

There are a few ways to deduplicate your state. In this contrived example, you can simply add a `completed` boolean to the `Todo` type so that the `completedTodos` array is no longer necessary.

### Underutilizing reducers

React has two built-in ways to store state: `useState` and `useReducer`. There are also countless libraries for managing global state, with Redux being the most popular. Since Redux handles all state updates through reducers, I'll be using the term "reducer" to refer to both `useReducer` reducers and Redux reducers.

`useState` is perfectly fine when state updates are simple. For example, you can `useState` to track whether a checkbox is checked, or to track the `value` of a text input.

That being said, **when state updates become even slightly complex, you should be using a reducer.** In particular, **you should be using a reducer any time you are storing an array in state and the user can edit each item in the array.** In the context of our to-do list app, you should definitely manage the array of to-dos using a reducer, whether that's via `useReducer` or Redux.

Reducers are beneficial because:

- They provide a centralized place to define state transition logic.
- They are extremely easy to unit test.
- They move complex logic out of your components, resulting in simpler components.
- They prevent state updates from being overwritten if two changes occur simultaneously. Passing a function to `setState` is another way to prevent this.
- They enable performance optimizations since `dispatch` has a stable identity.
- They let you write mutation-style code with [Immer](https://immerjs.github.io/immer/). You _can_ use Immer with `useState`, but I don't think many people actually do this.

### Not writing unit tests for the low-hanging fruit

Developers are busy people and writing automated tests can be time consuming. When deciding if you should write a test, ask yourself, "Will this test be impactful enough to justify the time I spent writing it?" When the answer is yes, write the test!

I find that mid-level React developers typically do not write tests, **even when the test would take 5 minutes to write and have a medium or high impact!** These situations are what I call the "low-hanging fruit" of testing. **Test the low-hanging fruit!!!**

In practice, this means writing unit tests for all "standalone" functions which contain non-trivial logic. By standalone, I mean pure functions which are defined outside of a React component.

Reducers are the perfect example of this! Any complex reducers in your codebase should have nearly 100% test coverage. I highly recommend developing complex reducers with Test-Driven Development. This means you'll write at least one test for each action handled by the reducer, and alternate between writing a test and writing the reducer logic that makes the test pass.

### Underutilizing `React.memo`, `useMemo`, and `useCallback`

User interfaces powered by React can become laggy in many cases, especially when you pair frequent state updates with components that are expensive to render (React Select and FontAwesome, I'm looking at you.) The React DevTools are great for identifying render performance problems, either with the "Highlight updates when components render" checkbox or the profiler tab.

**Your most powerful weapon in the fight against poor render performance is `React.memo`,** which only rerenders the component if its props changed. The challenge here is ensuring that the props don't change on every render, in which case `React.memo` will do nothing. You will need to employ the `useMemo` and `useCallback` hooks to prevent this.

I like to proactively use `React.memo`, `useMemo`, and `useCallback` to prevent performance problems before they occur, but a reactive approach — i.e. waiting to make optimizations until a performance issue is identified — can work too.

### Writing `useEffect`s that run too often or not often enough

My only complaint with React Hooks is that `useEffect` is easy to misuse. **To become an advanced React developer, you need to fully understand the behavior of `useEffect` and dependency arrays.**

If you aren't using the [React Hooks ESLint plugin](https://reactjs.org/docs/hooks-rules.html#eslint-plugin), you can easily miss a dependency of your effect, resulting in an effect that does not run as often as it should. This one is easy to fix — just use the ESLint plugin and fix the warnings.

Once you do have every dependency listed in the dependency array, you may find that your effect runs too often. For example, the effect may run on every render and cause an infinite update loop. There's no "one size fits all" solution to this problem, so you'll need to analyze your specific situation to figure out what's wrong. I will say that, if your effect depends on a function, storing that function in a ref is a useful pattern. Like this:

```tsx
const funcRef = useRef(func);

useEffect(() => {
  funcRef.current = func;
});

useEffect(
  () => {
    // do some stuff and then call
    funcRef.current();
  },
  [
    /* ... */
  ]
);
```

### Not considering usability

As a frontend developer, you should strive to be more than just a programmer. **The best frontend developers are also experts on usability and web design, even if this isn't reflected in their job titles.**

Usability simply refers to how easy it is to use an application. For example, how easy is it to add a new to-do to the list?

If you have the opportunity to perform usability testing with real users, that is awesome. Most of us don't have that luxury, so we have to design interfaces based on our intuition about what is user-friendly. **A lot of this comes down to common sense and observing what works or doesn't work in the applications you use everyday.**

Here's a few simple usability best practices that you can implement today:

- Make sure clickable elements appear clickable. Moving your cursor over a clickable element should change the element's color slightly and cause the cursor to become a "pointing hand" i.e. `cursor: pointer` in CSS. [Hover over a Bootstrap button to see these best practices in action.](https://getbootstrap.com/docs/5.1/components/buttons/)
- Don't hide important UI elements. Imagine a to-do list app when there "X" button that deletes a to-do is invisible until you hover over that specific to-do. Some designers like how "clean" this is, but it requires the user to hunt around to figure out how to perform a basic action.
- Use color to convey meaning. When displaying a form, use a bold color to draw attention to the submit button! If there's a button that permanently deletes something, it better be red! Check out [Bootstrap's buttons](https://getbootstrap.com/docs/5.1/components/buttons/) and [alerts](https://getbootstrap.com/docs/5.1/components/alerts/) to get a sense of this.

### Not working towards mastery of CSS & web design

**If you want to create beautiful UIs efficiently, you must master CSS and web design.** I don't expect mid-level developers to immediately be able to create clean and user-friendly interfaces while still keeping their efficiency high. It takes time to learn the intricacies of CSS and build an intuition for what looks good. But you need to be working towards this and getting better over time!

It's hard to give specific tips on improving your styling skills, but here's one: **master flexbox**. While flexbox can be intimidating at first, it is a versatile and powerful tool that you can use to create virtually all of the layouts you'll need in everyday development.

That covers the bad habits! See if you are guilty of any of these and work on improving. Now I'll zoom out and discuss some big picture best practices that can improve your React codebases.

## General Best Practices

### Use TypeScript exclusively

Normal JavaScript is an okay language, but the lack type checking makes it a poor choice for anything but small hobby projects. Writing all of your code in TypeScript will massively increase the stability and maintainability of your application.

If TypeScript feels too complex to you, keep working at. Once you gain fluency, you'll be able to write TypeScript just as fast as you can write JavaScript now.

### Use a data-fetching library

As I said in the "Bad Habits" section of this post, writing `useEffect`s correctly is hard. This is especially true when you are using `useEffect` directly to load data from your backend's API. You will save yourself countless headaches by using a library which abstracts away the details of data fetching. My personal preference is [React Query](https://react-query.tanstack.com/), though [RTK Query](https://redux-toolkit.js.org/rtk-query/overview), [SWR](https://swr.vercel.app/), and [Apollo](https://www.apollographql.com/docs/react/api/react/hooks) are also great options.

### Only use server rendering if you really need it

Server-side rendering (SSR) is one of the coolest features of React. It also adds a massive amount of complexity to your application. While frameworks like Next.js make SSR much easier, there is still unavoidable complexity that must be dealt with. If you need SSR for SEO or fast load times on mobile devices, by all means use it. But if you're writing a business application that does not have these requirements, please just use client-side rendering. You'll thank me later.

### Colocate styles with components

An application's CSS can quickly become a sprawling mess that no one understands. Sass and other CSS preprocessors add a few nice-to-haves but still largely suffer from the same problems as vanilla CSS.

I believe styles should be scoped to individual React components, with the CSS colocated with the React code. I highly recommend reading [Kent C. Dodds' excellent blog post on the benefits of colocation.](https://kentcdodds.com/blog/colocation) Scoping CSS to individual components leads to component reuse as the primary method of sharing styles and prevents issues where styles are accidentally applied to the wrong elements.

You can implement component-scoped, colocated styles with the help of [Emotion](https://emotion.sh/), [styled-components](https://styled-components.com/), or [CSS Modules](https://css-tricks.com/css-modules-part-1-need/), among other similar libraries. My personal preference is Emotion with the `css` prop.

**Update 2022-04-15:** Clarified my statement that you should "always" use a reducer when the state is an array.
