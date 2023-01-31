---
layout: post
title: 'High-density hosting on Azure App Service'
cover-img: "/assets/images/high-density/cover.jpeg"
thumbnail-img: ""
tags: AppService serverless compute 
---

I have long been a fan of Azure App Service as a compute option for server-side generated web sites and APIs. You get that Heroku-style ease of use, not having to  worry about containerisation, not having to worry about cold starts, on top of the usual serverless benefits like not having to worry about servers.

But, as with most things in architecture, there are trade-offs. App Service has somewhat of a trade-off around cost optimisation. For established workloads that are worthy of their own (or multiple) compute units, the trade-off disappears, but for new, small things, App Service can be an expensive option.

Microsoft somewhat alleviates this by letting you group multiple services together. The scale unit for an App Service is called an App Service Plan. An App Service Plan is the thing you pay for, and once you have created it you can deploy many different App Services to it. You can even deploy your Function Apps there side-by-side with App Services, to stop you needing to pay for both App Service Plans and Function App Plans.

So what might a typical lifecycle look like? Initially we have our App Service Plan hosting an App Service or two, but what happens as we grow?

>When you create an app in App Service, it is put into an App Service plan. When the app runs, it runs on all the VM instances configured in the App Service plan. If multiple apps are in the same App Service plan, they all share the same VM instances.

We aren't going to be able to keep adding App Services to our App Service Plan for ever, sooner or later it is going to start getting full - i.e. run out of memory or CPU. So what are our options?

![Scale Options](/assets/images/high-density/scale_options.png)

We could scale out if we are hitting CPU limits and just need to balance the traffic across some extra nodes, but this is unlikely to help us if we are hitting memory limits.

We could scale up, but this is just kicking the problem down the road. App Service Plans come in a few sizes (i.e. Small, Medium, Large), but there is a limit and we can't scale up forever.

We could create a second App Service Plan, and start putting new services on there, but this creates a whole bunch of other problems that I'd prefer to avoid. How to decide which App Services go on which App Service Plan? You could just put the new ones on the new App Service Plan, but wouldn't you prefer to scale certain services together, etc. Then there are other concerns such as virtual network integration... wouldn't it be much nicer if you could stay within the one App Service Plan?

Then there is one final option, something Microsoft call ["High-density hosting on Azure App Service using per-app scaling"](https://learn.microsoft.com/en-us/azure/app-service/manage-scale-per-app)!

It is somewhat buried away in the docs (red flag!), not mentioned at all in the Microsoft Learn AZ-204 training (red flag!), and not actually configurable via the portal, only Powershell/ARM (red flag!), but I decided to try it out anyways.

The general idea is that when enabled, an App Service no longer runs on all the instances configured in the App Service Plan, instead running on the number of instances that you configure that particular App Service to run on. 

![The General Idea](/assets/images/high-density/general_idea.png)

Sounds straightforward, right? Well, I decided to try it out and see if it does what it says on the box.

## Experiment #1 - Creation

The docs provide an example powershell statement on how to create an App Service Plan with Per Site Scaling.

```
New-AzAppServicePlan -ResourceGroupName $ResourceGroup 
                            -Name $AppServicePlan
                            -Location $Location
                            -Tier Premium 
                            -WorkerSize Small
                            -NumberofWorkers 2 
                            -PerSiteScaling $true
```
            
The number of instances for an App Service is again controlled by a NumberOfWorkers setting. The default appears to be 1, which suits me just fine for this experiment.

I started created App Services, observing the instance they were being deployed to, expecting them to be evenly spread across the two instances.

The first App Service went to a certain instance. Then the second App Service went to the same instance (hmm). Then the third App Service went to the same instance (hmmmmm). Then the fourth, then the fifth, and so on. 

![Experiment 1 - Expects vs Actual](/assets/images/high-density/experiment1-creation.png)

Looking at the App Service Plan in Azure Monitor showed that Azure had only created one instance in the App Service Plan, despite the number of workers being set to two.

Something was clearly wrong.

Digging back into the docs, there is a section at the bottom labelled "Recommended configuration for high-density hosting" that states

>Follow these steps to configure high-density hosting for your apps:
>
>1. Designate an App Service plan as the high-density plan and scale it out to the desired capacity.
>1. Set the PerSiteScaling flag to true on the App Service plan.

Following this steps did lead to having all instances activated, and App Services were then spread evenly across instances. However, needing to know the desired capacity up front is a pretty big turn-off, I don't think that that is how this cloud thing is meant to work at all! How would it handle scaling out my App Service Plan? Time to find out!

## Experiment #2 - Scaling Out

 Next I wanted to see if scaling-out an App Service Plan would result in the existing App Services being rebalanced across all instances.

Confidence was low after Experiment #1, but the following line in the docs gave me hope - 

>Applications are rebalanced only when instances are added or removed from the App Service plan.

I started with an App Service Plan with two instances, each hosting three App Services. I then scaled out the App Service Plan to three instances, hoping that an App Service from each of the previously existing instances would be moved over to the new instance.

![Experiment 2 - Expects vs Actual](/assets/images/high-density/experiment2-scale_out.png)

What happened? Absolutely nothing. The services stayed exactly where they were.

---

I did find ways to get my services to rebalance, but these were beyond what I feel I can reasonably ask of a team. 

Conceptually, I think this capability could further increase the surface area where App Service is suitable as a compute option. However due to the manner in which it has been implemented, at least for my use cases, that potential value is not able to be utilised.
