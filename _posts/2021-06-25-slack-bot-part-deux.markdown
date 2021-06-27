---
layout: post
title: "Slack Bot - Part Deux"
date: 2021-06-25 13:48:30 -0400
categories: Node Slack-bots Javascript
---

## A Wee Follow Up To Slack Bots

Ahoy there - I made a post a few months ago about creating a Slack bot to help with dependency management for a shared component library we use at work. Here's a little tldr from that post: we were running into the issue where changes in our component library weren't reflected in consuming applications because developers forgot to bump the version of the library. So I decided to create a Slack bot to help manage this developer workflow/chore. The first reason was to make this less of a chore and the second reason is because its fun to create bots!

After a few months of tinkering on this Slack tool (which we call Legos Manager), I'm pretty happy with where we've gotten it. It was originally intended as a very simple "command line tool", wherein you could invoke a slash command (`/legos {APP_NAME}`) inside of a Slack channel, and it would do all the dirty work of pulling the right repo, updating package.json and package lock files (or yarn.lock, depending), and making a pull request. And that worked! And we were happy. But we wanted more...

So this is where we are at now:

![oooh fancy modal oooohh](/assets/img/update_to_slack_bot.png)

Here's what Legos Manager 2.0 is capable of:

1. Rather than a slash command, it invokes Slack's Block Element API and shows the user a modal.

2. The modal allows you, the user, to select any number of internal applications and *any number of internal packages*.

3. The Slack Bot, living in a kubernetes pod in our cloud infrastructure, checks if each app is up to date with each package, then lets the user know its working to make PRs for each selected app.

4. A PR is created with a single commit whose message includes *all JIRA ticket numbers associated with the changes between the currently installed version of the internal packages and the latest version about to be installed*.

5. Legos Manager, our faithful companion, then sends the invoking user a message with the links to those PRs -- or it simply merges to develop if you choose the "force" option in the modal element.

A few notes about the above. For number two, I realized that while our component library is by far the most important internal package that we maintain, we have many others that encapsulate specific areas of work - things like accessibility helpers or shared search. It was a relatively straight forward feature enhancement to refactor the node application to accept an array of packages, and update to the latest version for each.

For number four, having the commit message include JIRA ticket numbers was quite important. For one thing, without those ticket numbers, we have no visibility in the consuming app about what the changes to a package might entail. And even more importantly, we have release management tools that use those JIRA project-ticket labels in commit messages to parse which apps should be in which releases. This ended up being a big win for our team, since before the Legos Manager we'd normally create a manual PR with a commit message that looks like this:

`chore: Update X package`

and it will now look like this, based on some git parsing done by the Slack bot:

`chore: Update X, Y, Z package which includes work on ABC-123, XYZ-456`

This is an example of why I really enjoy iterative work, focusing on the idea of making small, doable wins over time. I wanted to start with something relatively simple that did only *one* thing, and did it well. Then, we gradually added additional features based on feedback from the team. It's become something we can all take ownership and be proud of. Our little slack bot is all grown up!
