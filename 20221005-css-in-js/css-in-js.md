# Why We're Breaking Up with CSS-in-JS

Hi, I'm Sam — software engineer at [Spot](https://www.spotvirtual.com/) and the 2nd most active maintainer of [Emotion](https://emotion.sh/), a widely-popular CSS-in-JS library for React. This post will delve into what originally attracted me to CSS-in-JS, and why I (along with the rest of the Spot team) have decided to shift away from it.

We'll start with an overview of CSS-in-JS and give an overview of its pros & cons. Then, we'll do a deep dive into the performance issues that CSS-in-JS caused at Spot and how you can avoid them.

## What is CSS-in-JS?

As the name suggests, CSS-in-JS allows you to style your React components by writing CSS directly in your JavaScript or TypeScript code:

```tsx
// @emotion/react (css prop), with object styles
function ErrorMessage({ children }) {
  return (
    <div
      css={{
        color: "red",
        fontWeight: "bold",
      }}
    >
      {children}
    </div>
  );
}

// styled-components or @emotion/styled, with string styles
const ErrorMessage = styled.div`
  color: red;
  font-weight: bold;
`;
```

[styled-components](https://styled-components.com/) and [Emotion](https://emotion.sh/) are the most popular CSS-in-JS libraries in the React community. While I have only used Emotion, I believe virtually all points in this article apply to styled-components as well.

This article focuses on **runtime CSS-in-JS**, a category which includes both styled-components and Emotion. Runtime CSS-in-JS simply means that the library interprets and applies your styles when the application runs. We'll briefly discuss compile-time CSS-in-JS towards the end.

## The Good, The Bad, and the Ugly of CSS-in-JS

Before we get into the nitty-gritty of specific CSS-in-JS coding patterns and their implications for performance, let's start with a high-level overview of why you might choose to adopt the technology, and why you might not.

### The Good

1. **Locally-scoped styles.** When writing plain CSS, it's very easy to accidentally apply styles more widely than you intended. For example, imagine you're making a list view where each row should have some padding and a border. You'd likely write CSS like this:

```css
.row {
  padding: 0.5rem;
  border: 1px solid #ddd;
}
```

Several months later when you've completely forgotten about the list view, you create another component that has rows. Naturally, you set `className="row"` on these elements. Now the new component's rows have an unsightly border and you have no idea why! While this type of problem can be solved by using longer class names or more specific selectors, it's still on you as the developer to ensure there are no class name conflicts.

CSS-in-JS completely solves this problem by making styles locally-scoped by default. If you were to write your list view row as

```tsx
<div css={{ padding: "0.5rem", border: "1px solid #ddd" }}>...</div>
```

there's no way the padding and border can accidentally get applied to unrelated elements.

2. **Colocation.** If using plain CSS, you might put all of your `.css` files in a `src/styles` directory, while all of your React components live in `src/components`. As the size of the application grows, it quickly becomes difficult to tell which styles are used by each component. Often times, you will end up with dead code in your CSS because there's no easy way to tell that the styles aren't being used.

A better approach for organizing your code is to **include everything related to a single component in same place.** This practice, called colocation, has been covered in an [excellent blog post](https://kentcdodds.com/blog/colocation) by Kent C. Dodds.

The problem is that it's hard to implement colocation when using plain CSS, since CSS and JavaScript have to go in separate files, and your styles will apply globally regardless of where the `.css` file is located. On the other hand, if you're using CSS-in-JS, you can write your styles directly inside the React component that uses them! (And these styles will be locally-scoped, as discussed above.) If done correctly, this greatly improves the maintainability of your application.

3. **You can use JavaScript variables in styles.** CSS-in-JS enables you to reference JavaScript variables in your style rules, e.g.:

```tsx
// colors.ts
export const colors = {
  primary: "#0d6efd",
  border: "#ddd",
  /* ... */
};

// MyComponent.tsx
function MyComponent({ fontSize }) {
  return (
    <p
      css={{
        color: colors.primary,
        fontSize,
        border: `1px solid ${colors.border}`,
      }}
    >
      ...
    </p>
  );
}
```

As this example shows, you can use both JavaScript constants (e.g. `colors`) and React props / state (e.g. `fontSize`) in CSS-in-JS styles. The ability to use JavaScript constants in styles reduces duplication in some cases, since the same constant does not have to be defined as both a CSS variable and a JavaScript constant. The ability to use props & state allows you to create components with highly-customizable styles, without using inline styles. (Inline styles are not ideal for performance, when the same styles are applied to many elements.)

### The Neutral

1. **It's the hot new technology.** Many web developers, myself included, are quick to adopt the hottest new trends in the JavaScript community. Part of this is totally rationale, since in many cases, the new libraries and frameworks have proven to be massive improvements over their predecessors (just think about how much React enhances productivity over earlier libraries like jQuery). On the other hand, the other part of our obsession with shiny new tools is just that — an obsession. We're afraid of missing out on the next big thing, and we might overlook real drawbacks when deciding to adopt a new library or framework. I think this has certainly been a factor in the widespread adoption of CSS-in-JS — at least it was for me.

### The Bad

1. **CSS-in-JS adds runtime overhead.** When your components render, the CSS-in-JS library must "serialize" your styles into plain CSS that can be inserted into the document. It's clear that this takes up extra CPU cycles, but is it enough to have a noticeable impact on the performance of your application? We'll investigate this question in depth in the next section.

2. **CSS-in-JS increases your bundle size.** This is an obvious one — each user who visits your site now has to download the JavaScript for the CSS-in-JS library. Emotion is [7.9 kB](https://bundlephobia.com/package/@emotion/react@11.10.4) minzipped and styled-components is [12.7 kB](https://bundlephobia.com/package/styled-components@5.3.6). So neither library is huge, but it all adds up. (`react` + `react-dom` is 44.5 kB for comparison.)

### The Ugly

1. **Frequently inserting CSS rules forces the browser to do a lot of extra work.** [  
   Sebastian Markbåge](https://github.com/sebmarkbage), member of the React core team and the original designer of React Hooks, wrote an [extremely informative discussion](https://github.com/reactwg/react-18/discussions/110) in the React 18 working group about how CSS-in-JS libraries would need to change to work with React 18, and about the future of runtime CSS-in-JS in general. In particular, he says this:

> In concurrent rendering, React can yield to the browser between renders. If you insert a new rule in a component, then React yields, the browser then have to see if those rules would apply to the existing tree. So it recalculates the style rules. Then React renders the next component, and then that component discovers a new rule and it happens again.
>
> **This effectively causes a recalculation of all CSS rules against all DOM nodes every frame while React is rendering.** This is VERY slow.

The worst thing about this problem is that it's not a fixable issue (within the context of runtime CSS-in-JS). Runtime CSS-in-JS libraries work by inserting new style rules when components render, and this is bad for performance on a fundamental level.

2. **With CSS-in-JS, there's a lot more that can go wrong, especially when using SSR and/or component libraries.** In the Emotion GitHub repository, we receive _tons_ of issues that go like this:

> I'm using Emotion with server-side rendering and MUI/Mantine/(another Emotion-powered component library) and it's not working because...

While the root cause varies from issue to issue, there are some common themes:

- Multiple instances of Emotion get loaded at once. This can cause problems even if the multiple instances are all the same version of Emotion. [(Example issue)](https://github.com/emotion-js/emotion/issues/2639)
- Component libraries often do not give you full control over the order in which styles are inserted. [(Example issue)](https://github.com/emotion-js/emotion/issues/2803)
- Emotion's SSR support works differently between React 17 and React 18. This was necessary for compatibility with React 18's addition of streaming SSR. [(Example issue)](https://github.com/emotion-js/emotion/issues/2725)

And believe me, these sources of complexity are just the tip of the iceberg. (If you're feeling brave, take a look at the [TypeScript definitions for `@emotion/styled`](https://github.com/emotion-js/emotion/blob/8a163746f0de5c6a43052db37f14c36d703be7b9/packages/styled/types/base.d.ts).)

## About Spot

At [Spot](https://www.spotvirtual.com/), we're building the future of remote work. When companies go remote, they often lose the sense of connection and culture that was present in the office. Spot is a next-gen communication platform that brings your team together by combining traditional messaging and video conferencing features with the ability to create & customize your own 3D virtual office. Please check us out if that sounds interesting!

P.S. We're looking for talented software engineers to join the team! [See here for details.](https://www.spotvirtual.com/careers/)

![A picture of Spot](TODO:spot-hero.png)

_This post was also published [on the Spot blog](TODO)._
