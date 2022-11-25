---
layout: post
title: How to write better (User) Stories
subtitle: Agile Development
tags: [agile, stories]
published: true
---

User stories, technical debt and requirements - oh my!  If you've ever followed some sort of 'agile' process, you've likely heard of 'stories', where you break down the work that you're going to accomplish into bite-sized pieces and complete in your (usually) 2-week sprint (or if you're on Kanban, maybe your 2-week release cycle).

Trick is that they're pretty difficult to write, cause everyone seems to do their own thing, and as with any 'good' process, there isn't a right-or-wrong way to do it.  There are however some good rules of thumb to try and make things better!

1. User stories should NOT talk about solution, but discuss the problem to be fixed.
    - Planning Stage == describe the problem
    - Coding Stage == solve the problem
    - Testing Stage == ensure that the problem is solved
    - adopting Test Driven Development or Business Driven Development helps establish this

2. User stories should be written as a contract, simple description of problem, and should be a placeholder for conversation(s).
    - George Bernard Shaw - "The single biggest problem in communication is the illusion that it has taken place"
    - Collaboration between teams and team members is critical to success.

3. A story should be a small increment of work, that ideally fits inside a 2-week (sprint) period.
    - If the story requires work outside the individual (or team), you can open a linked-workitem and assign it to the other assignee.
    - Don't limit yourself to user stories.  Itemize system and other non-functional requirements with 'Technical' stories.

Try to have different types of stories (user, technical debt, support, network, security, etc.) to centralize planning and provide visibility to business of the various supporting activities that need to also happen during sprints.  Product owners can then have a more complete picture of the work that needs to happen, and can prioritize, scope and plan accordingly.

4. Business value might not need completed stories, and partially completed stories might be just as (or more) valuable to users
    - increment in small steps, to more quickly build value in each step
    - continuous reinforcement of value and completed work
    <!-- - WHAT the system does, not HOW the system does it -->
    <!-- - "50 quick ideas to improve your user stories" - ebook -->

5. Try to minimize dependencies between stories.
    - stories should be atomic, can be implemented in any order
    - if there is a dependency between stories, then you might to scale out a bit
        - group stories into 'features' or 'epics'

Atlassian Jira Software is very popular to track Agile software (and other team) projects, and supports multiple item types (issuetypes), linking those items together, and lots of other functionality for tracking agile artifacts.
In Jira hierachy:

- Epic issuetype
    - the "container" that the stories are going to describe work
    - example - "User Login"
- Story issuetype
    - describing what the requirements are, but at a more abstract level
    - custom field to select story type (user, network, system, security, etc.)
    - example - "As a user, I need to enter credentials and authenticate against a backend service."
- Work Item (Subtask) issuetype
    - breakdown of story to more functional, detailed level
    - example 
        - "Form based security, need a text box for user/pass and submit button"
        - "Need to authenticate form submission against AD, Azure AD and Okta"
        - "Integrate to Google Auth, for SSO"
- Tests (Subtask) issuetype
    - describe the system under test, picked up QA team/members
    - include success criteria, desired (happy) path
    - feedback/results can be passed back to dev teams with NEW work items/bugs, generated by qa
    - bugs take precedence over work items (usually)
    - BDD-style "when x, then y" descriptions
    - example
        - "When submitting valid AD credential by filling in user/pass and click submit, then user will be validated against remote AD system and logged in."
        - "When I click the icon to login with Google creds, then I will be presented with Google Auth form.  When I provide proper credentials, then I will be successfully logged in."