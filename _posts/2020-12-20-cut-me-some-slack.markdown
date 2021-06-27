---
layout: post
title: "Creating A Slack-Bot To Help Manage A Shared Component Library"
date: 2020-12-20 10:15:00 -0400
categories: Node Slack-bots Javascript
---

## Creating A Slack-Bot To Help Manage A Shared Component Library

At work, we have a shared component library that lives in a private registry filled with Vue components for a variety of different UI elements. This component library, which we call “Legos”, includes things as simple as buttons to more complicated items like API-aware lists of news cards.
We’ve worked very hard to make these components reusable, well-tested, and accessible, and we’ve tried to make the development and implementation of these components as easy as possible.

One final workflow hurdle is that, because the component library is imported as an NPM package, in order to see the changes from recent work done on a shared component, our developers need to make a pull request that “bumps” the version of the library to the latest version. Is this a huge impediment? No, but it is something that can cause a little bit of cognitive dissonance when trying to communicate with QA or project management. Take this example:

1. Developer A gets a ticket to add an aria-label to an icon on Page A.
2. Developer A realizes that the element in question on Page A is actually inside of a “Lego”.
3. Developer A implements the change in our shared package, PRs it, and has it merged.
4. The package goes through our CI system, and a new minor version of the component library is published.

It’s at this moment that Developer A might seem to be done with their work, since the PR is merged and the actual development is done. But there is an additional chore that needs to be completed, as described above, which is to import the latest version in the consuming project that contains Page A. And this process is repeated manifold for all the various interfaces across all of our applications - dozens of PRs across multiple applications that are simply to have the necessary changes manifest themselves in the required way.

With that in mind, I decided to build a Slack app that would help us manage this process. We already have some Slack integrations to help with builds and deployments, so it’s not a foreign tool for our team. Additionally, I wanted something that would be right at our fingertips, so that anyone (even, potentially, non-developers) could rectify the common situation of work being done in Legos, but not appearing in a relevant project.

What I wanted from the app was simple: a user could issue a slash command (/legos) followed by the name of an app. Then the Slack app would take care of the rest, alerting the user if the app was up to date with its version of our component library, and if not doing the updating itself and then pushing that update to a remote working branch.

![Initializing your Slack app](/assets/img/initializing_slack_app.png)


I went about building the app in Node, and used Slacks voluminous documentation as a starting point. The major challenges were:

1. Determining if the current version of our component library was the latest available - not as easy as it seems! One could get the latest version of a package, but comparing that effectively to the current used version took a little bit of problem solving.
2. Authenticating requests to our Bitbucket server - we would need to both pull down and push to our Bitbucket instance in order to update repos accordingly.
3. Passing configuration into a Docker container living in EKS - The node app would be living in a pod in our Elastic Kubernetes Service, and we would need to pass sensitive configuration such as environment variables in order to do the authenticating described above
4. A final major challenge was getting the workflow established for how to properly test a Slack app during local development! I created a test workspace, registered my app at api.slack.com, and then initialized a slash command. But the command itself needs a publicly accessible web address to proxy calls to, meaning I couldn’t just pass in localhost:3000 and start testing away.

A tool called ngrok came to the rescue in this case. Their tag line, I think, says all you need to know: “secure introspectable tunnels to localhost”. Essentially, I run ngrok from the command line with a flag for the local port I want bound to, it provides a tunnel to a web address, and that web address is provided to my slack command. It literally works like magic.

![Ngrok to the rescue!](/assets/img/slack_api_screenshot.png)

Currently, my Slack application works as intended locally. I can pass in the name of one of our consuming applications, have it feedback whether the shared library is up to date, and have it successfully “bump” to the latest version if not. While all is well working within a Docker container running on my local machine, the next step is to move the application into Kubernetes running in AWS. I’m sure that will have its own challenges and learning opportunities -- and perhaps a blog post of its own.

Thanks for reading!
