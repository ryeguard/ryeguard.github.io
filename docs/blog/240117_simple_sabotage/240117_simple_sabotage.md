# Simple Sabotage

## _or, How to Prevent It_

## Introduction

This text was inspired by a [blog post](https://erikbern.com/2023/12/13/simple-sabotage-for-software.html) I recently read that made me aware of the existence of the [Simple Sabotage Field Manual](https://www.cia.gov/news-information/featured-story-archive/2012-featured-story-archive/simple-sabotage.html). The manual was written in 1944 by the Office of Strategic Services (OSS), the predecessor to the CIA. It was written to help OSS agents sabotage the enemy. The manual was declassified in 2008, and is now available online.

The blog post goes on to describe what the author calls "simple sabotage for software", bringing the pamphlet into modern day software development organizations. As the author points out, the blog post is obviously rather an example of what _not_ to do. After discussing the field manual and blog post with a few colleagues, I decided to write this post. Rather than focusing on what _not_ to do, I will focus on what to do. I hope the examples can open your eyes to some of the behaviors that might be occuring in your organization, to some degree or another.

## The Manual

The manual, which is a great read in itself, goes into depth about motivating the saboteur and the saboteur's tools, targets, and timing. Most of this is not relevant to us, as we are not saboteurs, but rather anti-saboteurs. However, there's a few passages I would like to highlight:

> This type of activity, sometimes referred to as the “human element,” is frequently responsible for accidents, delays, and general obstruction **even under normal conditions**. The potential saboteur should discover what types of faulty decisions and the operations are **normally found in this kind of work** and should then devise his sabotage so as to enlarge that “margin for error.” - Section 1. Introduction, 5th paragraph

"This type of activity" refers the "second" type of sabotage as listed in the manual: indirect sabotage (the type we will focus on). The important bit here is "even under normal conditions" and "normally found in this kind of work". This tells us that the examples we will study below occur even when people are _not_ trying to sabotage anything. This is really why this manual is relevant to us, it helps us highlight what might be occurring in our organizations without us paying attention to it.

> Purposeful stupidity is contrary to human nature. He frequently needs pressure, stimulation or assurance, and information and suggestions regarding feasible methods of simple sabotage. - Section 3. Motivating The Saboteur, 2nd paragraph

This is observation is important, as it tells us that sabotage is not something that comes naturally to people. In extension, it means that the behaviors we will describe below are not anything people (normally) do with intent, but rather something they do by accident. This is important to keep in mind, as it means that we can hopefully prevent unintentional sabotage by making sure people are aware of their behaviors.

### Specific Examples

The section we will focus on is Section 5. Specific Suggestions For Simple Sabotage. The list of suggestions is extensive and covers everything from starting fires to ruining tires. The parts we will focus on is 5.11) General Interference With Organizations And Production and 5.12) General Devices For Lowering Morale And Creating Confusion. The lists below are the original suggestions from the manual, here prefixed with "Don't". After each "Don't" I have added a "Do", which is my (more or less opinionated) interpretation of what the opposite would be.

When reading this, ask yourself (or better yet, discuss with your colleagues!):

- Is this something that occurs in my organization/team?
- Is there something similar that occurs in my organization/team?
- Am I doing this?
- What are some examples of this occurring?

#### 11) General Interference With Organizations And Production

##### (a) Organizations and Conferences

1. Don't: Insist on doing everything through “channels.” Never permit short-cuts to be taken in order to expedite decisions.

   Do: Permit short-cuts to be taken in order to expedite decisions.

2. Don't: Make “speeches.” Talk as frequently as possible and at great length. Illustrate your “points” by long anecdotes and accounts of personal experiences. Never hesitate to make a few appropriate “patriotic” comments.

   Do: Be short and concise in your communication. Make sure you have a clear point, and that you get to it quickly.

3. Don't: When possible, refer all matters to committees, for “further study and consideration.” Attempt to make the committees as large as possible—never less than five.

   Do: Be open to taking decisions on your own, in your team, or in the group discussing the matter. Aim to keep the number of people involved as low as possible. Typically, the people discussing the matter are the ones who are best suited to make the decision.

4. Don't: Bring up irrelevant issues as frequently as possible.

   Do: Stay on topic.

5. Don't: Haggle over precise wordings of communications, minutes, resolutions.

   Do: Be open to accepting the wording of others, and don't get stuck on the details.

6. Don't: Refer back to matters decided upon at the last meeting and attempt to re-open the question of the advisability of that decision.

   Do: Accept decisions that have been made, and move on. Only revisit the decision if it (repeatedly) comes up as a problem.

7. Don't: Advocate “caution.” Be “reasonable” and urge your fellow-conferees to be “reasonable” and avoid haste which might result in embarrassments or difficulties later on.

   Do: Be open to taking risks and making mistakes. Mistakes are a great way to learn. Iterate often.

8. Don't: Be worried about the propriety of any decision—raise the question of whether such action as is contemplated lies within the jurisdiction of the group or whether it might conflict with the policy of some higher echelon.

   Do: (Again, from point 3) Be open to taking decisions on your own, in your team, or in the group discussing the matter. Typically, the people discussing the matter are the ones who are best suited to make the decision.

##### (b) Managers and Supervisors

1. Don't: Demand written orders.

   Do: Be open to informal communication where appropriate. Don't get stuck on formalities.

2. Don't: “Misunderstand” orders. Ask endless questions or engage in long correspondence about such orders. Quibble over them when you can.

   Do: Do your best to understand the orders you are given or the request/ask someone is giving. If you don't understand, ask questions. Do not get caught up in the details and remember no [bike-shedding](https://en.wikipedia.org/wiki/Law_of_triviality)!

3. Don't: Do everything possible to delay the delivery of orders. Even though parts of an order may be ready beforehand, don’t deliver it until it is completely ready.

   Do: Deliver early and often. Relatable to one of the [12 principles of the Agile Manifesto](https://agilemanifesto.org/principles.html).

4. Don’t order new working materials until your current stocks have been virtually exhausted, so that the slightest delay in filling your order will mean a shutdown.

   Do: Regularly update your software development tools and libraries to the latest versions. Additionally, don't wait with technical debt until it becomes a problem.

5. Don't: Order high-quality materials which are hard to get. If you don’t get them argue about it. Warn that inferior materials will mean inferior work.

   Do: Be open to using the tools and technologies that are in use and available. It's easy to look for the new, shiny thing and think it will solve all your problems.

6. Don't: In making work assignments, always sign out the unimportant jobs first. See that the important jobs are assigned to inefficient workers of poor machines.

   Do: Assign the important tasks first. If assigning an important task to a less efficient worker, make sure to provide the necessary support and training to ensure the task is completed successfully.

7. Don't: Insist on perfect work in relatively unimportant products; send back for refinishing those which have the least flaw. Approve other defective parts whose flaws are not visible to the naked eye.

   Do: Accept that not everything will be perfect and accept "good enough". Again, no bike-shedding.

8. Don't: Make mistakes in routing so that parts and materials will be sent to the wrong place in the plant.

   Do: Take care to route incoming tasks/messages/requests to the correct recipient(s).

9. Don't: When training new workers, give incomplete or misleading instructions.

   Do: Put effort into training new coworkers. On a similar note, enable and encourage knowledge sharing between coworkers.

10. Don't: To lower morale and with it, production, be pleasant to inefficient workers; give them undeserved promotions. Discriminate against efficient workers; complain unjustly about their work.

    Do: Be fair and just in your treatment of coworkers. Be open to feedback, and be open to giving feedback.

11. Don't: Hold conferences when there is more critical work to be done.

    Do: Don't hold meetings for the sake of holding meetings. Make sure there is a clear purpose for the meeting, and that the meeting is necessary to achieve that purpose.

12. Don't: Multiply paper work in plausible ways. Start duplicate files.

    Do: Streamline paper work, documentation processes, and bureaucracy. Avoid duplication when writing documentation by aiming to "update" rather than "add" or "append".

13. Don't: Multiply the procedures and clearances involved in issuing instructions, pay checks, and so on. See that three people have to approve everything where one would do.

    Do: Streamline non-value creating processes and bureaucracy such as approvals, task management, time/expense reporting, etc.

14. Don't: Apply all regulations to the last letter.

    Do: Be open to bending the rules when it makes sense to do so.

## Links

- [Amazon Leadership Principles](https://www.amazon.jobs/content/en/our-workplace/leadership-principles)
- [This is water](https://thewholehearted.org/2018/11/30/this-is-water/)
- [Simple Sabotage Field Manual](https://www.cia.gov/news-information/featured-story-archive/2012-featured-story-archive/simple-sabotage.html)
- [Simple Sabotage for Software](https://erikbern.com/2023/12/13/simple-sabotage-for-software.html)
