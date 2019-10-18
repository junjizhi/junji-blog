---
layout: post
title: "Zapier + Pivotal: Is It Really No-code?"
categories: [All, Technical]
tags: [zapier, no-code, pivotal tracker, automation]
fullview: false
excerpt: This post talks about how we use Zapier to automate some steps of the Pivotal Tracker workflow. Because of the limited actions supported by Zapier, we end up using Python code and call Pivotal REST API for posting comments.
comments: true
---

## TL;DR
**Zapier supports most activity triggers for Pivotal Tracker, but not enough types of actions. A little Python code to call Pivotal API helps me get past the limit.**

## Who is the post for?
Before using it, I found it hard to know the boundary of no-code tools like Zapier. You need to know what's possible as is, and what it takes to extend the existing functionality.

This post documents our experience and the limitations. It helps you if:
- you are new to Zapier or evaluating whether to use Zapier, and
- your workflow involves Pivotal Tracker and you want to automate (parts of) the workflow

## The use case
Our team uses Pivotal Tracker to keep track of the features or bugs we are working on. When a Pivotal ticket is accepted, we need to notify a group of reviewers (for auditing purposes).

Currently this is done manually. The goal is to set up Zapier to automate the workflow:
- When a ticket is accepted, each reviewer receives an email
- If any reviewer replies to the email, the reply is automatically posted as comment to the Pivotal ticket.

If the automation works well, everyone will be working with the tools they are most familiar with. It saves everyone time and effort of switching from tools.

## Automating notifications

### Zapier trigger
First, we want to listen to the Pivotal event. There are three types of trigger events:
- New Story
- New Activity
- New Project

![image](https://user-images.githubusercontent.com/2715151/85187365-275ff180-b26d-11ea-970a-45b1ab907603.png)

Since we want the zap to run when a ticket is accepted, and it should belong to the `New Activity` category.

### Zapier filter
By default, any type of activity in Pivotal will trigger the zap to run. But we are only interested in the ticket accepted event, so we add a Zapier filter:

![image](https://user-images.githubusercontent.com/2715151/85187696-6727d880-b26f-11ea-82ac-8b07d3c54eee.png)

### Send an email
Zapier has Gmail integration, so the next step is simple:

![image](https://user-images.githubusercontent.com/2715151/85187712-84f53d80-b26f-11ea-8cfb-f55a60319737.png)

## The story doesn't end here
Because we use Pivotal is usually *the place* we look at our project progress, we add as much information there as possible. So the next requirement is:

- If a reviewer replies the email, add the reply as a comment to the Pivotal ticket

*In reality, the requirements usually kept growing when we automate the workflow partially.*.

To support this, we have to create another zap that listens to Gmail event:

![image](https://user-images.githubusercontent.com/2715151/85187811-84a97200-b270-11ea-85f3-3fc4e4e07dcd.png)

We select the email matching a text pattern. That part is straight-forward.

## We hit the wall of unsupported actions
Next, we want the zap to post the email text to Pivotal. That's where we hit the wall:

![image](https://user-images.githubusercontent.com/2715151/85187822-b8849780-b270-11ea-88aa-913360233cdf.png)

In the action list, Zapier doesn't have the option to post a comment.

## Final solution? Code.

We look at [Pivotal REST API V5](https://www.pivotaltracker.com/help/api/rest/v5) and found that it supports adding comments via API:

```bash
curl -X POST -H "X-TrackerToken: $TOKEN" -H "Content-Type: application/json" -d '{"text":"If this is a consular ship, then where is the ambassador 👅?"}' "https://www.pivotaltracker.com/services/v5/projects/$PROJECT_ID/stories/$STORY_ID/comments"
```

And Zapier supports adding Python code as a step:

![image](https://user-images.githubusercontent.com/2715151/85187859-f7b2e880-b270-11ea-9ef3-7cdf541d6b95.png)

So we write a Python script to post comments to the story:

```python
headers = {'X-TrackerToken': 'xxxx'}
pivotal_url = f"https://www.pivotaltracker.com/services/v5/projects/{project_id}/stories/{story_id}/comments"
comment = input_data['gmail_content']
response = requests.post(pivotal_url, headers=headers, data={'text': comment})
response.raise_for_status()
```

And that solves our problem.

## Closing thoughts

This post talks about how we use Zapier to automate some steps of the Pivotal Tracker workflow. Because of the limited actions supported by Zapier, we end up using Python code and call Pivotal REST API for posting comments.

The overal experience with Zapier is smooth. Zapier supports many apps and programming logic out of the box, like filter, text processing.

However, it can still run into a limit because some triggers or actions are not supported yet. If you are looking for completely no-code solution, you should bear this in mind.

Thanks for your read!

*If you want to learn about Zapier or no-code, subscribe to my newsletter!*