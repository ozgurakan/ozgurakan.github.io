---
layout: bootstrap_post
title: The Agile Dilemma
date: 2015-05-12 21:10:00
author: Oz Akan
abstract: Would you build a feature which will block you later on?
categories:
    - Development
---

Not knowing is simple. You want to know just enough so you can take a step without even being ready for the next one.

The beauty of agile development lies underneath the ability to change direction quickly. Communication is essential in agile development. You write, not to communicate, you write so team can remember the reasons behind the decisions  later on. You are always present in stand-ups, not to report your work but to get an update where team members stand, what problems they are facing so maybe, maybe you can offer help or find out something that will affect what you will do next.

<sub>Example: You are designing the database schema. You learn that John is planning to change the API for a resource to store a new field in database. You inform John that you already remember seeing that field in database. You both have a quick follow up discussion after the stand-up call and figure out that there is another API URL which covers the case. John won't need to make a change.</sub>

Agile development is all about communication and that is to ensure team members know about each others work. They are there to ask hard questions to others. Each member needs to have an understanding on what matters the most for the product so that each developer can decide what will come next.

Sometimes what comes next isn't what it seems it is and you will have to fight for it as it will be invisible for the most.

Consider your team is takes with building a building. Task is something like this:

`Build me 5 story building.`

These are the basic principles on our imaginary buildings:

- Buildings have two components
  - Foundations
  - Stories
- Foundation defines how many stories can the building have.
- The stronger the foundation is, the more stories can be build on it.
- The stronger the foundation is, the longer it takes to build it.
- After a story build on the foundation, foundation can not be developed any further.
- If all stories on the foundation are destroyed, foundation can be developed and new stories can be build on top of the improved foundation.
- Building the foundation to hold one story is 4 effort points
- Building a story is 2 effort points
- Destroying a story is 1 effort point

Based on the information above, to build a 5 story building we will have to accomplish 30 points of work _( (4 + 2) * 5 = 30)_.

An experienced architect will ask, are we really going to stop at 5th floor or is there a chance we will have to add more stories later on? If we will add, how many stories we might add?

Let's say we started with the assumption of 5 stories being the final and then found that we will have to add 5 more stories. After **30 effort points**, we will have to destroy 5 stories, and improve the foundation to hold 5 more stories. Then we will build 10 stories. This adds 45 effort points. In total we will accomplish **75 effort points** of work to have 10 story building.

If we build a building that will have 5 stories but capable of holding 10, it requires **50 effort points**. Then if we add 5 more stories, it is **60 effort points** of work.

   > To build the 10 story building, in the first approach we have to invest 15 more effort points than the second approach.

   > On the other hand, first approach gave us 5 story building with 20 points less effort than the second approach.

Would you fight to build the foundation for 10 stories first? Deciding on which approach to take depends on the business requirements. Question is;

<p class="highlight">Is there a real immediate need for the first 5 stories so you will have to build something you know you will destroy soon after?</p>

Dilemma lies in how agile you want to be to get to the first 5 stories.