---
layout: post
title: "Enabling CitizenOps for Integration Workloads with Azure AI Services "
cover-img: "/assets/images/citizen-ops/cover.png"
thumbnail-img: ""
tags: genai logicapps
---

We build a lot of integration solutions. These solutions focus on moving data from one system to another. Often this is transitory in nature, an integration is about the movement of data, rather than about making a change to one particular system.

![System integration](/assets/images/citizen-ops/integration.png)

A failed integration usuallt does not leave a distinct system in an error state, but it likely does leave your overall organisation in more abstract error state. How do we become aware that a failed integration has happened? How do we investigate and remedy the situation?

Today we would usually build a very technical integration dashboard. An operations engineer would use the dashboard to dig into the issue, understand what had caused the failure, and then pass that information on to a business user where necessary.

![Integration dashboard](/assets/images/citizen-ops/integration-dashboard.png)


## Can we do better?

Very often when an integration fails, it is a data problem. The receiving system may have different validation requirements on a field compared to the sending system, and what was acceptable to one system is not acceptble to the other. In my experience data isses are for more common than more technical issues, such as networking or authenitcation errors.

Given that these errors are often better resolved by a business user than a technical person, I wondered if there is there a way to surface them in a more suitable fashion.

I envisioned a solution where a business user could ask a question such as "Why didn't employee Bob Smith make it to System X?" and receive an answer that was actionable for them.

![The goal](/assets/images/citizen-ops/the-goal.png)

Using Azure AI Services, Azure Integration Services, and a (very) simple web application, I was able to create such a solution!


## So how does it work?

The implementation uses the popular Retrieval Augmented Generation (RAG) pattern, utilising the Azure AI Search and Azure OpenAI services. This pattern is widely used in many "chat with your own data" style applications. 

In this pattern, a company's data is initially loaded into a vector database, which is a kind of database that excels at searching for things that are 'similar'. Then, when a user's query is received, a database lookup is performed to look for similar content, and then the user's query and the database response is fed into a ChatGPT-style Generative AI to create a user-friendly response.

![RAG pattern](/assets/images/citizen-ops/rag.png)

We generally build integrations using Azure Logic Apps. The Logic App team recently announced new connectors for Azure AI Search and Azure OpenAI. This opened the door to easily extending current integration implementations for our users, without needig to build entirely new AI solutions.

I created a new, reusable ingestion workflow. The workflow receives a small payload containing the request and response information from an attempted system integration, as well as some accompanying metadata that provides context to help improve quality of the results. This new workflow leveraged Azure OpenAI to 'vectorise' the payload, and Azure AI Search to store the payload and its vector.

![Ingestion flow](/assets/images/citizen-ops/ingestion-flow.png)

This workflow can then easily be called from any existing Logic App integrations. 

Below is a common integration pattern where a workflow is picking up a pub/sub style message (in this instance from an Azure Service Bus Topic) and attempting to send the message to an operational system. The only addition needed to this workflow to enable this solution is the final action to call the new ingestion workflow. 

![Calling flow](/assets/images/citizen-ops/calling-flow.png)

The other half of the solution is the retrieval workload. This flow takes the user's question, vectorises it, retrieves data from Azure AI Search, then passes it all in to Azure OpenAI to generate that user-friendly content. This flow did not need to be built using a Logic App, but here I wanted to test out the new connectors.

![Retrieval flow](/assets/images/citizen-ops/retrieval-flow.png)

---

I find it interesting to explore the new approaches to building systems that have arrived with AI services, especially being able to leverage so much out-of-the-box with Azure AI Services.