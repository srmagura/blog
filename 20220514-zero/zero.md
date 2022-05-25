# Zero: One API Key to Rule Them All

_This post is sponsored by [Zero](https://tryzero.com/)._

[Zero](https://tryzero.com/) is an upcoming service that gives you a single API key that provides access to AWS, Stripe, Twilio, and tons of other APIs. This post is a sneak peek into what Zero will offer when the beta goes live. The team is aiming to launch in early June, so not too far away!

## What Problem Does It Solve?

Just about any modern cloud application needs to access external APIs for things like provisioning infrastructure, sending notifications, and processing payments, and each of these APIs will require an API token for authentication. It's not hard to manually manage the API keys under the following assumptions:

1. You only use a few APIs.
2. Your company is only a few people.
3. You're OK sharing the API keys via email or a Google doc, even though this is potentially less secure.

If any of these assumptions do not apply to your situation, managing API keys can become a real hassle. Imagine how much effort it would take to manage all your API tokens if:

1. You use 20 different APIs across your company.
2. Your company has 100 people who need API access, but only certain teams should have access to certain APIs.
3. Security is paramount, so sending keys in emails is a no-go.

This is where Zero comes in. In the next section, I'll cover how Zero helps with each of these difficulties.

## How Does Zero Address These Pain Points?

### Problem: You use 20 different APIs across your company.

**Once you sign up for a Zero account, you gain instant access to many popular APIs.** You no longer have to create separate accounts for Microsoft Azure, SendGrid, Shopify, and so on. This is a big time saver in the early stages of developing a product when you want to get moving as quickly as possible.

Zero will provide you with a "Zero token" that grants access to one or more APIs. You can use it in your Node.js code like this:

```js
import zero from "@zero/zero";
import { S3Client } from "@aws-sdk/client-s3";
import Stripe from "stripe";

// The most popular APIs, like AWS, Stripe or Sendgrid are precreated for each
// account. One can start using them right after a sign up by simply requesting
// them in a Zero API call
const creds = await zero({
  apis: ["aws", "stripe"],
  token: process.env.ZERO_TOKEN,
}).fetch();

const client = new S3Client({ credentials: creds.aws });
const stripeApi = Stripe(creds.stripe);
```

### Problem: Your company has 100 people who need API access, but only certain teams should have access to certain APIs.

You can use the Zero web console to group users into teams:

![Teams list in the Zero web console](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9oo64f6to84vn39zxne2.png)

Then, you can assign each Zero token to one or more teams:

![Token list in Zero web console](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/c5vb1te1kiq55ug82iks.png)

This permissions model seems very easy to use while also being powerful enough to scale to larger organizations.

### Problem: Security is paramount, so sending keys in emails is a no-go.

All of your credentials are accessed either via the Zero web console or the Zero API, so there is no need to share API tokens via less secure means like email. When paired with the ability to give teams access to only the APIs they need, this design gives Zero the potential to be much more secure than manual API key management.

## How To Use It

Once the beta goes live, using Zero will go something like this:

1. Sign up for a Zero Token.
2. Pick the APIs that you need.
3. Use the Zero SDK in your code to fetch credentials for individual APIs. (See the code sample above for what this will look like.)

One thing you might be wondering is, what if I need to use a less common API that isn't integrated with Zero? Zero fully supports this scenario! Zero is not just a convenient shortcut for accessing APIs — more broadly speaking, it is a secrets management platform. This means Zero can store any secret key, even if it's not for one of the APIs that the service integrates with. Here's a screenshot of the UI you would use to upload a secret key for a less common API:

![Adding an API key manually in Zero](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6msjn18g4itv707bq8v6.png)

## Q&A with the Team

After reviewing the marketing materials, code snippets, and screenshots, I had a number of questions about the service. My contact on the Zero team, Artem, was happy to answer. Below is an edited version of my questions and his answers.

**Q:** I'm unclear about how Zero will connect to each of my accounts / subscriptions. For example, can I use Zero with my preexisting Twilio account?

**A:** Zero does not connect to your existing accounts. We create accounts (like AWS or Twilio) long before you sign up, store them in our safe storage, and then share some set of credentials with you once you sign up.

**Q:** Some teams will want to have different tokens for development, testing, and production. How would you accomplish this in Zero?

**A:** This is one is very good question! The credentials that we create are expected to be used mostly in dev environment. So when you need to start quick, and the only goal is speed, you just sign up to Zero, grab your credentials through SDK, and that’s it, you are ready to build a thing instead of signing up for each service! Later on, when you go to production you’ll need to create new set of credentials in the accounts that we will transfer to you right after sign up.

**Q:** Will Zero release API clients for programming languages other than JavaScript / TypeScript?

**A:** The initial release will have SDKs for JavaScript, TypeScript, and Go. We plan to add SDKs for other languages in the future.

**Q:** Are users able to view preexisting Zero tokens in the web console? I'm asking because many services do not allow you to view secrets after they are created.

**A:** Zero is a secrets manager, so yeah — you’ll be able to observe a full secret in the web UI.

## Concerns

In the interest of providing a balanced take on the service, I'll share a few of my concerns about the service.

1. I'm a bit sad that Zero is not intended to be used in production, as this seems to limit the usefulness of the service. Maybe the team can work on supporting this after the initial launch.
2. It sounds like, when you sign up for Zero, they will transfer the preexisting accounts for each API provider to you. I'm not sure exactly how this process would work, but it seems like Zero will not help you manage the credentials for these service accounts. So even with Zero, there is still _some_ need to manually manage credentials for each API provider.

## Final Thoughts

Zero has the potential to remove a lot of the tedium that's currently required to get access to the APIs you need. At the end of the day, anything that lets us spend less time setting up credentials and more time writing code is a good thing!

Thank you to Zero for sponsoring my post and making it easy to review the service. I have high hopes for you all!
