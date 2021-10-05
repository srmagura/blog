# The Easiest Way to Make a Mobile App? CodeSandbox!

_Now, before you come at me with a pitchfork — I'm saying this is the **easiest** way to make a mobile app, not the **best** way to develop an app for your use case._

I started weightlifting about a year ago and I needed a way to time the break I took between sets. A mobile app was the obvious solution but I needed one that met my requirements:

- I wanted a huge, easy to press button to start the timer.
- I didn't want to have mess with text inputs or dropdowns to change the length of the timer.
- I didn't want an app with ads or complex features that I would never use.
- I wanted something free.

While there may have been an app on Google Play that met my requirements, I didn't bother looking, **because I knew I could create my own mobile app in 30 minutes.** But how?

## Introducing CodeSandbox

If you haven't heard of [CodeSandbox](https://codesandbox.io/), it's a web-based development environment that makes it super easy to create JavaScript applications.

**IMAGE HERE**

Unlike earlier tools like CodePen [1], CodeSandbox is a much closer approximation of a local development environment. You can create files, folders, and easily install packages from npm. While it's not a replacement for local development, CodeSandbox is perfect for demos, experiments, and minimal reproducible examples.

## Creating the Weightlifting Timer

The first step to creating an app on CodeSandbox is to select a template. I chose the React + TypeScript template, but you can use normal JavaScript, Vue, or whatever else floats your boat. I won't go into the details of the app's code since there are already many great resources for learning React and JavaScript timers.

All things considered, it took around 30 minutes to finish the sandbox, [which you can view here.](https://codesandbox.io/s/weightlifting-timer-tkcsu?file=/src/App.tsx)

**APP IMAGE HERE**

_Clicking one of the buttons starts a timer for that many seconds. A sound is played when the timer finishes._

## Using the App

Getting the app on my phone was as simple as typing the sandbox URL, `tkcsu.csb.app`, into Chrome's address bar on my phone. The CodeSandbox React templates are set up as [Progressive Web Apps](https://web.dev/progressive-web-apps/) (PWAs), so Chrome immediately prompted me to add the app to my home screen! [2] Here's how it looks:

**ICON IMAGE HERE**

I wasn't bothered at all by the CodeSandbox branding, but you might want your own icon and app name to show up instead. While this is normally easily accomplished by adding a [web app manifest](https://web.dev/add-manifest/), I still got the CodeSandbox logo after adding my own `manifest.json`. It turns out CodeSandbox overwrites your manifest with its own. 😑 There's a [closed GitHub issue about this](https://github.com/codesandbox/codesandbox-client/issues/1532) with a hacky workaround if you're persistent.

Manifest issues aside, the app works perfectly and has been used in "production" (by me) for over a year!

## SERIOUS Mobile Development 😠

CodeSandbox is a great way to make your first app or create a mobile utility for personal use. But if you want to get serious and create a professional-grade mobile app, there are better options.

At one end of the spectrum, you have truly native iOS & Android development. At the other end, you have PWAs and hybrid app frameworks like [Ionic](https://ionicframework.com/). For me, [React Native](https://reactnative.dev/) hits the sweet spot. There's also Google's [Flutter](https://flutter.dev/) and the upcoming [.NET MAUI](https://docs.microsoft.com/en-us/dotnet/maui/what-is-maui). I won't do a detailed comparison of these options here, but it may be the topic of a future post.

### Endnotes

**[1]** To be fair, CodePen has improved since I last used it and now allows you to install npm packages.
**[2]** I've heard Apple isn't the biggest fan of PWAs, so you may not get prompted to add the app to your home screen if you use iOS. Of course, you can still bookmark the app and access it through your browser.
