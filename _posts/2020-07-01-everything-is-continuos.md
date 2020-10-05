---
layout: post
title: |
  Everything is Continuous: A Simple Strategy on Software Delivery.
date: 2020-07-01
author: assay.it
description: |
  Why is modern software engineering talking about Continuous Integration, Continuous Delivery and Continuous Deployment? Everything is Continuous!
# tags:
#   - case study
#   - continuous deployment
thumbnail: https://source.unsplash.com/collection/8363627/2000x1322?a=.png
---

Modern software engineering is talking about Continuous Integration, Continuous Delivery and Continuous Deployment. Why should we distinguish them? **"Everything is Continuous"** defines a right philosophy and commitment that ensures the always ready state of your code. It also implements pipelines to deploy every commit straight to sandbox with the following promotion to production.

This approach delivers few measurable benefits to any Software as a Service business:

1. **Reduce Cost** by eliminating toil activities from engineering processes. Team focuses their effort towards feature development and other business valuable activities. By coding the entire delivery chain, the team accomplishes automated and repeatable processes. 

2. **Increase Speed** of iteration cycles. It shortens delivery time and unlocks an opportunity to release software at any point in time.

**"Everything is Continuous"** does not invent any special workflow. It just emphasizes deployment and quality assessment as a key feature along the development process. Continuous proofs of the quality helps to eliminate defects at earlier phases of the feature lifecycle. It impacts on engineering teams philosophy and commitments, ensuring that your microservice(s) are always in a release-ready state. The right implementation of the workflow deploys every commit to disposable sandbox environment with the following promotion to production. 

![A simple strategy to achieve an automation with GitHub Actions](/img/posts/2020-07-01-everything-continuous-workflow.svg)

The workflow does not differ at all from [forking or branching git workflow](https://www.atlassian.com/git/tutorials/comparing-workflows#forking-workflow). It just emphasizes continuous deployment as a key feature along the delivery process. It supports quality assessments and helps to eliminate all related issues at earlier phases of feature delivery process:

1. The `main` branch of your project is always the latest deployable snapshot of a microservice. CI/CD have to automate the `main` snapshot deployments every time when a new feature is merged.

2. The feature integration into `main` is implemented through pull request (no exceptions whatsoever). CI/CD executes automated pull request deployment to the sandbox environment every time new changes are proposed (each commit). The sandbox environment is a disposable deployment dedicated only for pull request validation.

3. The merge of the pull request triggers the deployment of the `main` branch into the development environment. Again, every modification of the development environment triggers quality assessments before features are delivered to production.

4. Finally, the delivery of the development environment to production is automated using git tags. A provisioning of a new tag caused a new immutable deployment of microservice(s) to the live environment.

5. In each step, CI/CD triggers [https://assay.it](https://assay.it/) to executes quality assessments jobs against the right environment.

The successful adoption of "Everything is Continuous" requires **Continuous deployment**, which shall be aligned with other CI/CD delivery pipelines. **Infrastructure as a Code** and **immutable deployments** are two engineering practices to consider as part of the deployment automation.

**Infrastructure as a Code** is the only right way to manage resources in an automated manner. It is preferable to use **declarative IaC**, which describes the infrastructure using generic programming languages (strictly type safe language is strongly advised) without explicit definition of commands to be performed. A familiar development tool chains speed-up development and static type checks validates correctness of the infrastructure.

The risk of occasional outage is always there due to regression in software quality. The **immutable deployment** is the solution for availability and fault tolerance. CI/CD never changes anything at running systems. Immutable deployment making changes by rebuilding microservices, it deploys a new copy in parallel stack. The rollback of defective software  happens in a matter of seconds. Another advantage is the ability to split traffic between parallel deployments for quality assurance. 

The actual workflow implementation depends on tool chain used by the engineering team, especially CI/CD system. There is [a reference implementation](/doc/case-study/everything-is-continuous) of this workflow with AWS CDK and GitHub Actions. At the glance, the workflow requires steep learning curve, the knowledge about deployment automation is required before jumping into it. The time investment is worth it due to measurable benefits: cost reduction, faster execution and mitigation of risks.

![A reference implementation of quality automation](/img/posts/2020-07-01-everything-continuous-design.svg)

We build [https://assay.it](https://assay.it/) to help developers with quality assessment of microservices in the distributed environment. The service is designed to perform a formal and objective proof of the quality using Behavior as a Code paradigm every time changes are applied to your environment.