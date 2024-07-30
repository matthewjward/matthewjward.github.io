---
layout: post
title: 'Finding a use case for the Premium tier of Azure Functions'
cover-img: ""
thumbnail-img: ""
tags: Azure Functions Pricing
---

[Launched]("https://azure.microsoft.com/en-us/blog/introducing-azure-functions/") in the mid-2010's, seemingly as a response to the instantly popular AWS Lambda, Azure Functions is Microsoft Azure's Function-as-a-Service offering.

>Azure Functions is an event driven, compute-on-demand experience that extends the existing Azure application platform with capabilities to implement code triggered by events occurring in Azure or third party service as well as on-premises systems.

Microsoft, with App Serice, had a much stronger Platform-as-a-Service offering than AWS had, making it harder for Azure Functions to eek out a home for itself.

##confused person

One strength is the programming model, where if you can put aside concerns about vendor lock-in (as most organistations can in 2023) you can leverage a collection of out of the box 'bindings' that accelerate your development.

This model conceptually bring Azure Functions more into alignment with Azure Logic Apps in the integration space than as a compute alternative to Azure App Service.

Once you have decided to use Azure Functions, your next decision is which hosting plan to choose. 

##confused person

There are essenitally three options - Consumption, Premium, and Dedicated.

Consumption is the 'true' serverless offering and is billed accordingly, you pay for Total Executions and Execution Time.

It, however, is limited. Mainly, that there are cold start issues and there is no private networking, limiting your ability to build a properly secure solution.

These issues are addressed by the Premium Tier offering, but in order to do so the pricing model moves from the pure serverless consumption model to more of a PaaS-pricing model, where you pre-provision compute by the hour.

This is where things get interesting, because the Dedicated hosting plan basically offers the same thing, only in a slightly more BYO fashion. Oh yeah, and it is also MUCH cheaper.


// ridiculously simple comparison picture?

Comparing the two hosting plan options highlights two main differences, the first is the aforementioned price, and the second is the scaling behaviour. The Premium plan (and Consumtion plan) uses Azure Functions Event driven scaling, while the Dedicated option uses, well, whatever scaling mechanism the BYO PaaS layer brings, but for the sake of this post lets assume it is Azure App Service autoscaling.

----

So what does Azure Functios Event driven scaling bring to the table? Remember those bindings that were part of the programming model desribed earlier? Well they also enable more intelligent scaling than the Memory / CPU consumption that you get in App Service. Have an Azure Function that is triggered by a message arriving in Azure Service Bus? Great! Azure Functions can look at the lenghth of your queue and scale appropriately.

The question is - can we simulate any traffic behaviour where this is worthwhile.

## Experiment #1 - Steady traffic

On the one hand, this is the obvious place to start, as it's the simplest traffic pattern. On the other hand, it's a tad unfair, as we've just established that the stength of Azure Functions Premium is the event driven scaling.

![Alt text](/assets/images/functions-premium/steady.png)

## Experiment #2 - Business day traffic

![Alt text](/assets/images/functions-premium/business-day.png)

## Experiment #3 - Burst traffic

## Experiment #4 - Crazy burst traffic




----

It feels like the Dedicated tier came along as a third wheel some time later to quieten people who were saying "Hey, I already pay for pre-provisioned compute, why can't I run my Azure Functions there too". Which is a fair question, but I wonder if by providing this option Microsoft basically removed the need for their Premium hosting plan.




