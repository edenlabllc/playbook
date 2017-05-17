# Collaboration rules

## Agile mindset

We organize the development process based on the [agile manifest](http://agilemanifesto.org/principles.html)

But we do not strictly follow any metodology or geek religios. We apply some rules only if it works for us.

Our teams frequently experiment with different tools and techniques so we can evolve our skills. This applies not only to development work, but to everything we do at EdenLab, from user research to business operations.

We recognise the importance of planning, compliance and governance, so we work those activities into the process, every step of the way.

## Development sprints
We work in sprints, which are fixed-length periods of delivery work with a particular team.

Sprint duration - 2 weeks

## Responsibility
We work as one one team and anyone can perrform any task if he has proper skills and will.

The common goal for the whole team is to make sprint done in time and in the commited scope.

Usually we have System Analysts and Developers in the team who works together on the product delivery.

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
* functional/integration testing on Dev environment based on the aceptance criteria.

## The Definition of Done - ticket has been developed, tested and deployed to the staging environment

## Sprint planning

## Daily stand-ups

## Issue tracking

By default we use Github as an issue tracker for all the projects

We use only below issue types. All of them can be used at the same time in one project depends of the need.

   1. feature - should be started from the business expression of the need - link to Confluence. And gradually is enriched by SA or Dev with Business process, ERD, Apiary spec, etc. Usually whenever it is needed something to be developed, new ticket with the `kind/feature` should be created
     * Must use
     * complex WF
     * can be assigned to SA, Dev 
     * Can be closed only by SA
   2. release - task to deliver 
     * Must use
     * Simple WF - backlog, in_progress, done
     * Can be assigned to DevOps, Dev
     * Can be closed by anyone
   3. bug - used to register bugs that has been detected after release
     * Must use
     * Simple WF - backlog, in_progress, done
     * Can be assigned to Dev
     * Can be closed by anyone
   4. Design - used to register and track all the activities that need to be done before development starts
     * Optional use
     * Simple WF - backlog, in_progress, done
     * Can be assigned to SA, Dev
     * Can be closed by anyone
   5. Infra - tasks related to the infrastructure setup/configuration
     * Optional use
     * Simple WF - backlog, in_progress, done
     * Can be assigned to DevOps
     * Can be closed by anyone
   6. task - can be used instead of Design, Infra tasks
     * Optional use
     * Simple WF - backlog, in_progress, done
     * Can be assigned to anyone
     * Can be closed by anyone

`user story` style should be used by default whenever new feature is created: "As a <role>, I want <goal/desire> so that <benefit>"

Business process diagram in the [BPMN 2.0 notation](https://en.wikipedia.org/wiki/Business_Process_Model_and_Notation) usually is a result of the design phase.



### Feature stages
We use github labels to track ticket statuses

Below statuses should be used only for the `kind/feature` tickets. 
* backlog - any new ticket usually is created with this status lable
* design - means that ticket is under the design phase. Requirements are collected and described at that stage. Should be marked by SA.
* specs - means that ticket is under the specification phase. ERD, API specs and `acceptance criteria` should be developed at that phase. May be performed by SA or Dev. Anyway it should be reviewed by another. 
* todo - means that ticket is ready to be estimated and for the development to be started.
* in_progress - means that development has been started. Should be marked by Dev.
* test - means that feature has been completely developed, covered by tests, reviewed, deployed to `Dev` environment and `acceptance criteria` has been checked by Dev
* bug - means that it has been tested and some bug was found. In this case it shoud be reassigned to Dev. 
* wontfix - means that feature has been rejected because of any reason.



## Sprint retrospective

