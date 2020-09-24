---
title: |
  How To Confirm Quality of Serverless API baked by DynamoDB, AWS Lambda and API Gateway
date: 2020-09-24
description: |
  Quality assurance of serverless applications is more complex than doing it for other runtimes. Engineering teams spend twice as much time maintaining testing environments and mocking cloud services instead of focusing on the product values.
# tags:
#   - case study
#   - continuous deployment
image: https://source.unsplash.com/xyXcGADvAwE/2000x1322?a=.png
author_staff_member: facebadge
---

Serverless technologies are the easiest solution for delivering scalable and robust application programming interfaces. Engineering teams are empowered with a capability of crafting world class interfaces without excessive management of the infrastructure and worrying about operations. An automated delivery accomplished by the Infrastructure as a Code is the only right way to manage serverless resources. Let’s evaluate a classical blueprint of Serverless CRUD API that build with following AWS services.

* DynamoDB is a NoSQL storage backend for CRUD API;
* AWS Identity and Access Management controls read/write access permissions to cloud resources;
* AWS Lambda functions implement api’s “business logic” without provisioning servers or explicit management of containers;
* API Gateway is a front-end proxy layer to access serverless workload from the Internet. 

![](/images/posts/2020-09-24-blueprint-serverless-api.svg)

The blueprint builds [a fictional bookstore](https://github.com/assay-it/blueprint-serverless-api), which is used in this post to discuss issues related to the quality. It implements a naive REST CRUD interface to store, delete, and update books in a DynamoDB database: 
* `GET /books` retrieves collection of all books
* `POST /books` appends a new book into the collection
* `GET /book/{id}` fetches details about single book from collection
* `PUT /book/{id}` updates the book in the collection
* `DELETE /book/{id}` removes the book from the collection

Infrastructure as a Code is one approach to implement deployment automation for your blueprints. [This example](https://github.com/assay-it/blueprint-serverless-api) is not an exception, it uses declarative IaC with AWS CDK. It describes the infrastructure using TypeScript without explicit definition of commands to be performed. 

```bash
npm -C cloud install
npm -C cloud run cdk -- deploy

...

 ✅  bookstore

Outputs:
bookstore.GatewayEndpoint4DF49EE0 = https://xxxxxxxxxx.execute-api.eu-west-1.amazonaws.com/api/

Stack ARN:
arn:aws:cloudformation:eu-west-1:000000000000:stack/bookstore/00000000-0000-0000-0000-000000000000
```

AWS CDK successfully constructed the blueprint. The bookstore Serverless API is available in your AWS Account and ready to serve traffic. All endpoints are public, it is possible to probe them with curl:   

```bash
curl -XPOST https://xxxxxxxxxx.execute-api.eu-west-1.amazonaws.com/api/books \
  -H 'Content-Type: application/json' \
  -d '{"title": "The Lord of the Rings"}'

curl  https://xxxxxxxxxx.execute-api.eu-west-1.amazonaws.com/api/books 
```


## Quality risks

The risk of occasional outage is always there due to regression in software quality despite the Serverless nature. Serverless only helps with operational and infrastructure management processes. However, engineers are still responsible for the quality of their products. Interfaces have to work as planned. It needs to serve api clients according to the value it promises and ensure great experience.

Quality assurance of serverless applications is more complex than doing it for other runtimes. Engineering teams spend twice as much time maintaining testing environments and mocking cloud services instead of focusing on the product values. A typical testing strategy is built around mocks and pale imitations around genuine cloud services. This is the worst form of confirmation bias. Looking into the Serverless blueprint, it becomes a requirement to keep testing environments identical with live systems. Engineers have to mock every nuance of real cloud services, content and behaviour of DynamoDB, API Gateway, IAM and so on. Even if a team makes a moderate push to mock a real system they quickly get out of sync. A typical failures, which are laborious to catch during unit testing, are misconfiguration of IAM roles; uncontrolled changes to storage schema; changes in the API deployment profiles; basically any incompatible change to AWS resources required by your service and software bugs. The best effort simulation of the production serverless environment is the production serverless environment itself. Safe testing in production requires automations, understanding of holistic approaches and proper isolation of test traffic from real one.

## Behavior as a Code

A secret sauce of successful automation strategy is the code. Not only the Infrastructure as a Code but also expressing expected Behavior as a Code. Just write pure functional Golang instead of clicking through UI or maintaining endless XML, YAML or JSON documents. This typesafe approach fits very well to express communication behavior because it gives rich techniques to hide the networking complexity and allows to design the variety of network communication use-cases. The functional approach helps us to compose a chain of network operations and represent them as pure computation, building new things from small reusable elements.

To get started with a declaration, create a folder `suite` and a file `bookstore.go`. This will be the place to implement asserts about desired behaviour of the bookstore interface. The interface needs to ensure store, delete, and update book screarios; methods are named after them:


```go
package suite

import (
  "github.com/assay-it/sdk-go/assay"
  c "github.com/assay-it/sdk-go/cats"
  "github.com/assay-it/sdk-go/http"
  ƒ "github.com/assay-it/sdk-go/http/recv"
  ø "github.com/assay-it/sdk-go/http/send"
)

func Create() assay.Arrow { /* ... */ }

func Update() assay.Arrow { /* ... */ }

func Lookup() assay.Arrow { /* ... */ }

func Remove() assay.Arrow { /* ... */ }

func Lifecycle() assay.Arrow { /* ... */ }
```

The behavior declaration is a pure Golang module. The example above is a skeleton of quality assurance strategy for the bookstore interface. Each assessment is just an exported function of a type `() ⟶ assay.Arrow`. It connects cause-and-effect (Given/When/Then) with the networking concepts (Input/Process/Output): 
* “Given” specifies the communication context and the known state of the expected behavior;
* “When” executes key actions about the interaction with remote component;
* “Then” observes output of the remote component, validates its correctness and outputs results.

These suites become a part of automated and continuous proofs of the quality that is applied every time the team makes a change to the interface implementation. It helps to eliminate defects at earlier phases of the feature engineering lifecycle before the defects are exposed to api clients. This practice impacts on engineering teams philosophy and commitments, ensuring that your interface(s) are always in a release-ready state. The bookstore blueprint uses [https://assay.it](https://assay.it) Software as a Service to implement the continuous quality evaluation strategy. Its Golang SDK guides through the process of suite development as a composition of network operations and represents them as pure computation, building new “things” from small reusable elements:

```go
func Create() assay.Arrow {
  book := Book{
    ID:    "book:hobbit",
    Title: "There and Back Again",
  }

  return http.Join(
    ø.POST("%s/books", sut),
    ø.ContentJSON(),
    ø.Send(&book),
    ƒ.Code(http.StatusCodeOK),
    ƒ.Recv(&book),
  ).Then(
    c.Value(&book.ID).String("book:hobbit"),
    c.Value(&book.Title).String("There and Back Again"),
  )
}
```

All other suites are implemented using a similar style. Please find the entire Golang implementation at [GitHub repository](https://github.com/assay-it/blueprint-serverless-api). Fork the repository to your own GitHub account. 

## Confirm Quality

Software as a Service, such as [https://assay.it](https://assay.it), automates validation of cause-and-effect in loosely coupled topologies: serverless applications, microservices and other systems that rely on interface syntaxes and its behaviors. Let’s first manually execute the quality assessment against the deployed bookstore Serverless API:
* Install [assay command line](https://github.com/assay-it/assay): `go get github.com/assay-it/assay`;
* Sign up for [assay.it](https://assay.it) with your GitHub developer account (check the quick [start guide](https://assay.it/doc/));
* Generate personal access key at your Profile;
* Add your fork of [blueprint-serverless-api](https://github.com/assay-it/blueprint-serverless-api) to assay.it workspace.

```
assay run your-github/blueprint-serverless-api 
--url https://xxxxxxxxxx.execute-api.eu-west-1.amazonaws.com/api 
--key Z2l0aH...DBkdw
```

The command uses a latest snapshot of Behaviour as a Code suite from the repository to run the assessment against the given deployment of the service. Results become visible at service itself. 

![](/images/posts/2020-09-24-quality-assessment.png)


## Automations

The successful adoption of continuous quality proof is an automation along the development pipeline. A simple strategy to achieve an automation with GitHub Actions has been discussed in [previous posts](http://localhost:4000/2020/07/01/everything-is-continuos/). As a key highlight here, the right implementation of the automation workflow is the deployment of every change to an isolated environment with the  following promotion to production. 

![](/images/posts/2020-07-01-everything-continuous-workflow.svg)

Continuous proofs of the quality helps to eliminate defects at earlier phases of the feature lifecycle. It impacts on engineering teams philosophy and commitments, ensuring that your microservice(s) are always in a release-ready state. GitHub Actions allows teams to choose and build automation techniques with a perspective on time to market, cost, and risk. Integration of quality assessment is an essential part of these pipelines, the ready made action [assay-it/github-actions-webhook](https://github.com/assay-it/github-actions-webhook) does everything to execute WebHook on your behalf:

```yaml
- uses: assay-it/github-actions-webhook@latest
  with:
    secret: ${{ secrets.ASSAY_SECRET_KEY }}
    target: https://xxxxxxxxxx.execute-api.eu-west-1.amazonaws.com/api
```

## Afterwords

Customer trust is hard to earn if your Serverless interface is glitching. Interfaces have to serve their clients according to the value it promises and ensure great experience, otherwise customers find another product for their needs. Use of automated solutions to continuously detect issues in every change before a full blow outage is slapped on the customer face, assay.it has you covered. Choose your testing techniques with a perspective on time to market, cost, and risk.

