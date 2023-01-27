---
layout: post
title: 'Taking Azure AD B2C ‘Seamless Migration’ for a spin'
tags: IDaaS b2c Identity Auth
---

For many organisations, cloud is not simply heading off to fresh pastures, but instead entails complex migrations out of ‘on-prem’ data centres.

A core part of this migration, especially for customer-facing organisations, is moving users. Good customer experience demands a frictionless approach, a-la ‘Seamless Migration’, but migrating user passwords that you (hopefully!) do not have in plain text, or do not know the hashing algorithm for, presents a challenge.

## Azure AD B2C

[Azure AD B2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/overview) is Azure’s Identity as a Service (IDaaS) customer identity access management service.

AD B2C uses standards-based authentication protocols including OpenID Connect and OAuth 2.0. By serving as the central authentication authority for your web applications, mobile apps, and APIs, Azure AD B2C enables you to build a single sign-on (SSO) solution for them all.

![AD B2C Overview](/assets/images/seamless-migration/azureadb2c-overview.png "AD B2C Overview")

Its competitors in the IDaaS space are Okta, probably the leader in the space (at least according to Gartner), but more expensive, and Ping. [This Okta blog post](https://www.okta.com/resources/whitepaper/not-all-identity-clouds-are-created-equal/), comparing themselves to Azure AD B2C is insightful, but should of course be read with a grain of salt.  

There are a variety of open-source options in the self-hosted space, including Duende IdentityServer which is popular in the .NET ecosystem.

Importantly however, a lot of cloud migrations have core principles of utilising managed services, and of leveraging as much of their chosen cloud vendor as possible, so for anyone leaning towards Azure it makes sense to start by evaluating AD B2C.

## Seamless Migration

The AD B2C docs does have [a page](https://docs.microsoft.com/en-us/azure/active-directory-b2c/user-migration) on migrating users to AD B2C which discusses Seamless Migration.

There are two main components - first, an initial 'pre-migration' of all user accounts into AD B2C is performed. This is followed by creating a callback to your existing identity solution to validate a user's password.

Then, using an AD B2C 'custom policy', each time a user logs in a call is triggered to the legacy system. If it returns that the password is correct then AD B2C stores the password, marks the user as migrated, and no longer needs to callback for that user.

If your legacy solution is not accessible via an API call then you are out of luck.

![Diagram Illustrating Seamless Migration](/assets/images/seamless-migration/seamless-migration.png "AD B2C Seamless Migration")

For me, the existence of this page in the offical documentation initially implied that Seamless Migration was 'built-in' functionality. However the docs eventually link you to [a GitHub repo](https://github.com/azure-ad-b2c/user-migration/tree/master/seamless-account-migration) which has the rather ominous disclaimer.

“The migration application is developed and managed by the open-source community in GitHub. The application is not part of Azure AD B2C product and it's not supported under any Microsoft standard support program or service. This migration app is provided AS IS without warranty of any kind.”

## Let's try it out

With the exception of being slightly out of date, the repository readme does an excellent job of guiding you through what you need to do to perform Seamless Migration.

In order to track whether a user has been migrated, a [custom attribute](https://docs.microsoft.com/en-us/azure/active-directory-b2c/user-flow-custom-attributes) is created in Azure AD B2C. It is great that there exists the ability to create custom attributes, however it would be really helpful to be able to see the value of a custom attribute for a user from within the portal, and to be able to sort/filter all users on a custom attribute. I ended up creating my own management web app to display this value.

![Custom Attribue](/assets/images/seamless-migration/custom-attribute.png "AD B2C Custom Attribute")

The pre-migration step was relatively straightforward, utilising the user-friendly Microsoft Graph APIs to create a bunch of users. The name of the custom attribute used when accessing it programatically is a bit odd, but not a big deal.

The biggest challenge came with Azure AD B2C [Custom Policies](https://docs.microsoft.com/en-us/azure/active-directory-b2c/custom-policy-overview_). I understand that adding extensibility points within products is hard, but this felt like customising a service that was not designed for customisation.

B2C is explicit that you are entering 'identity pro' land, but I am not convinced that slapping a warning on something gets you out of delivering a good product experience! This admission is tellingly called out in the above-mentioned Okta blog post.

![Custom Policies Warning](/assets/images/seamless-migration/custom-policies.png "AD B2C Custom Policies")

There is a large amount of boilerplate point-and-click to enable custom policies in B2C, outside of authoring the policies themselves. There is a [community app](https://b2ciefsetupapp.azurewebsites.net/) to automate this process, but it would be great to see this process automated within B2C itself.

Likewise, there is a (somewhat out of date) [community repo](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack) full of custom policy samples, but no canonical 'starting point', which I believe would be useful. Taking the sample policies and updating them according to the readme results in policies that fail validation, which is not great.

Authoring the policies was challenging. The policies are written in xml, and while there are [fairly strong docs](https://docs.microsoft.com/en-us/azure/active-directory-b2c/trustframeworkpolicy_), a lot of the terms used are not particularly user-friendly. Being honest, I would say I more cut-and-pasted sections from blogs/tutorials/samples than actually wrote much policy.

Lastly, uploading policies triggers a validation process, that in the case of failure (i.e. quite often) resulted in strangely formatted error messages.

![Custom Policies Validatiom Error](/assets/images/seamless-migration/custom-policies-validation-error.png "AD B2C Custom Policies Validation Error")

Combining determination and my knowledge of the domain eventually resulted in policies that both passed validation and worked as desired! I wouldn't say I deeply understand how they are working - I may return and figure it out, I may not.

---

In summary, I achieved what I was looking to achieve, but spent a little longer to do so than I would have liked. I don't particularly value time spent learning how to configure SaaS tools, and feel Azure AD B2C would benefit from a little more investment in the customer experience for some of these more advanced scenarios.
