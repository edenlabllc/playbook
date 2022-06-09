# Collaboration rules

## Agile mindset

We organize the development process based on the [agile manifest](http://agilemanifesto.org/principles.html)

But we do not strictly follow any methodology or geek religious. We apply some rules only if it works for us.

Our teams frequently experiment with different tools and techniques, so we can evolve our skills. This applies not only to development work, but to everything we do at EdenLab, from user research to business operations.

We recognise the importance of planning, compliance and governance, so we work those activities into the process, every step of the way.

## Development sprints
We work in sprints, which are fixed-length periods of delivery work with a particular team.

Sprint duration - 2 weeks

## Freedom and responsibility
This is one of our values. But simply having it on a poster does not help creating this feeling of freedom nor sharing responsibilities.

Projects are shaped and prioritised in advance and product teams are free to organize the way they want in order to meet chosen goals within given time and boundaries.

We work as a one team and anyone can perform any task if he has proper skills and will.
The common goal for the whole team is to make sprint done in time and in the committed scope.

Usually we have System Analysts and Developers in the team who works together on the product delivery.

### Product Owner (PO)
* Collect, analyze and document business requirements
* Business processes design
* Study best practices
* Manage project team/scope/budget
* Backlog of User Stories

### System analyst (SA)
* Collect, analyze and document business requirements
* Business design
* Technical specs design - jointly with Developer
* Support and track development process
* Testing
* Documentation

### Developer (Dev)
* Technical specs design - jointly with SA
* Code development
* Unit testing
* functional/integration testing on Dev environment based on the acceptance criteria.

### Maintainer
* Keep master branch at production quality at all times.
* Keep develop branch at deployment quality at all times.
* Keep `readme.md` and `environment.md` up to date.
* Accept only [high quality pull-requests](https://github.com/edenlabllc/ehealth.api/blob/develop/docs/CONTRIBUTING.md#pr-review-rules). Provide code reviews and guidance on incoming pull requests.
* Take part in the new feature design process and be responsible for the proposed technical solution.
* Any public facing API or UI or architectural or significant changes requires approval from the architect.


**The Definition of Done - ticket has been developed, tested and deployed to the dev environment**

## Sprint planning
Sprint planning session should happen before sprint starts. All the project team members should take part in this session.
PO defines the high-level sprint goal.
SA presents the user_story or task purpose/details and specs. Any team member can ask questions to clarify the task statement and agree on the implementation options.
After discussion and evaluation task should be estimated using the Planning Pocker [technique](https://en.wikipedia.org/wiki/Planning_poker)

## Daily stand-ups
All the project team members are required to attend scrum meetings. Anyone else is allowed to attend, but is there only to listen. This makes scrum meetings an excellent way for a Scrum team to disseminate information.
The daily scrum meeting is not used as a problem-solving or issue resolution meeting. Issues that are raised are taken offline and usually dealt with by the relevant subgroup immediately after the meeting. During the daily scrum, each team member answers the following three questions:
* What did you do yesterday?
* What will you do today?
* Are there any impediments in your way?

## Issue tracking
By default we use Github as an issue tracker for all the projects

Usually we decompose the project into:
* Epic - item from the scope that has been agreed with the customer. Every epic should have lebel in the GH project. 
  * **User_Story** - natural language description of one or more features of a software system. GitHub issues with the label `kind/user_story` are used to track. Each User Story should have `epic/` label
    * **Task** - what has to be done to implement user story. GitHub issues with the label `kind/task` are used to track.
      * Personal task - decomposed steps of task. Not tracked. It is up to everyone how to organize his personal everyday activities. Looks liuke CheckLists in the GH issues are a very convenient way to do that.
       
We use only below issue types in GitHub. All of them can be used at the same time in one project depends of the need. `kind/` group of labels are used to classify issues.
   1. __User_Story__ - should be started from the business expression of the need - link to Confluence. And gradually is enriched by SA or Dev with Business process, ERD, Apiary spec, etc. Usually whenever it is needed something to be developed, new ticket with the `kind/user_story` should be created
     * Must use
     * complex WF
     * can be assigned to SA, Dev 
     * Can be closed only by SA
   2. __release__ - task to deliver 
     * Must use
     * Simple WF - backlog, in_progress, done
     * Can be assigned to DevOps, Dev
     * Can be closed by anyone
   3. __Bug__ - used to register bugs that has been detected after release
     * Must use
     * Simple WF - backlog, in_progress, done
     * Can be assigned to Dev
     * Can be closed by anyone
   4. __Design__ - used to register and track all the activities that need to be done before development starts
     * Optional use
     * Simple WF - backlog, in_progress, done
     * Can be assigned to SA, Dev
     * Can be closed by anyone
   5. __Infra__ - tasks related to the infrastructure setup/configuration
     * Optional use
     * Simple WF - backlog, in_progress, done
     * Can be assigned to DevOps
     * Can be closed by anyone
   6. __task__ - can be used instead of Design, Infra tasks
     * Optional use
     * Simple WF - backlog, in_progress, done
     * Can be assigned to anyone
     * Can be closed by anyone
   7. __technical_debt__
     * optional use
     * complex WF
     * can be assigned to SA, Dev 
     * Can be closed only by SA
   8. __project_office__ - to mark the tasks of the external teams
     * Optional use
     * Simple WF - backlog, in_progress, done
     * Can be assigned to anyone
     * Can be closed by anyone
`User story` style should be used by default whenever new feature is created: "As a <role>, I want <goal/desire> so that <benefit>"

Business process diagram in the [BPMN 2.0 notation](https://en.wikipedia.org/wiki/Business_Process_Model_and_Notation) usually is a result of the design phase.


### User story stages
We use github labels `status/` to track the ticket status

Below statuses should be used only for the `kind/feature` tickets. 
* backlog - any new ticket usually is created with this status lable
* design - means that ticket is under the design phase. Requirements are collected and described at that stage. Should be marked by SA.
* specs - means that ticket is under the specification phase. ERD, API specs and `acceptance criteria` should be developed at that phase. May be performed by SA or Dev. Anyway it should be reviewed by another. 
* todo - means that ticket is ready to be estimated and for the development to be started.
* in_progress - means that development has been started. Should be marked by Dev.
* test - means that feature has been completely developed, covered by tests, reviewed, deployed to `Dev` environment and `acceptance criteria` has been checked by Dev
* bug - means that it has been tested and some bug was found. In this case it shoud be reassigned to Dev. 
* wontfix - means that feature has been rejected because of any reason.

Labels `kind/` and `status/` are really important and should be used as it is stated in the playbook.

Also we may use other labels just to make our life easier:
 * `priority/`
 * `value/`
 * `complexity/`
 * `priority/`
 * `bug`
 * `help!`
 * `do-not-merge`
 

## Sprint retrospective
The sprint retrospective is usually the last thing done in a sprint. The entire team should participate. 
Each team member should share his impression on the sprint that has just been finished. 
Usually it is structured as an answers to the following questions:
 * What did I do right
 * What did I do wrong
 * What the team did right
 * What the team did wrong
 * What we should start doing
 * What we should stop doing
 * What we should continue doing
