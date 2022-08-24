👈🏾&nbsp;&nbsp; Previous: [Enhancing a Kratix Promise](/enhancing-a-promise/) <br />

# What's next?

Our last hands-on session went through how to [enhance a Kratix Promise](/enhancing-a-promise/README.md). 

Thinking back to the questions at the start of the workshop, you are the best person to decide what is most valuable to continue exploration in Kratix. 

In your context, what were the answers to these questions:

* What is the highest value service your platform provides to your application development teams? 
* How easy is it for you to provide and maintain that service?
* How easy is it for application developers to use that service?
* How can you enhance flow for your application developers and reduce effort for you and your platform team?

## Designing the right Promise for a single service

You may identify with any of the below scenarios, or the whole story might be familiar!

### Standardised development tools...
Through a Slack message, you happen to learn that four application teams are all using four different flavours of Postgres. Each team's database is in a different cloud, each has a different backup strategy, and each uses different levels of monitoring. 

### ...pre-configured to meet your business requirements...
For each of those Postgres databases, you have complex billing scenarios where you need to enforce quotas; you need to send a request to an external API to validate permission to bill that particular cost centre; and you need to inform interested stakeholders via email.

### ...secured according to your policies...
In addition to more complex billing scenarios, you need to implement more rigorous security policies. You need to ensure that configuration values are acceptable according to the broader security protocols; you need to verify proper signoff has happened before deployments move forward; and you need to make sure that teams have confidence that they are shipping with dependencies that are risk-free for the organisation.

### ...and optimised for scale.
After learning about the four 'shadow IT' Postgres databases, you poll application teams. Turns out there are at least six more teams using a mix of cloud providers to get Postgres databases to suit their needs. You need to manually intervene in each team's backlog to audit the situation and ensure each database is healthy and compliant. 
<br/>
<br/>

For any single service, like this database example, there are _a lot_ of platform concerns. With Kratix you can address the platform concerns without endless toil.

Kratix allows you to encapsulate a service in a Promise with a robust Request Pipeline, and with this you can create a platform that _does_ offer standardised development tools that are pre-configured to meet your business requirements; secured according to your policies; and optimised for scale.

## Designing the right Golden Paths with multiple Promises
Extending the idea that your platform as-a-Product provides value by giving teams what they need when they need it, Kratix makes it easy to [pave Golden Paths](https://www.syntasso.io/post/paving-golden-paths-on-multi-cluster-kubernetes-part-1-the-theory) through complex Promise composition.

A “Golden Path” is the opinionated and supported path to “build something”. Imagine a complete development environment setup&mdash;networking, integration, security, governance, compliance, and deployment&mdash;all available on-demand. By paving a Golden Path the platform team makes doing the right thing easy for the application teams.

Creating your Golden Paths on Kratix is easy: decide on, define, and install the individual Promises that are required, then define a composed Promise that brings everything together. 

An application developer goes from making separate Kratix Resource Requests to get access to separate service instances to being able to make one single Resource Request to get pre-configured, ready-to-go instances of everything they need. 

We believe composable Promises are at the core of the value that Kratix provides to platform teams. 

Take a moment to imagine the most valuable bundle of services that your platform could offer to your application teams. Now that you've had experience building a platform with Kratix, translate that bundle of services into a composed Promise using Kratix as the supporting framework. 

## Learn more 
If the idea of treating your platform as-a-Product is new concept to you, watch this short talk by [Paula Kennedy](https://twitter.com/PaulaLKennedy) at Devoxx UK: [Crossing the Platform Gap](https://youtu.be/pAk5GReIs90) or read our two-part series about [Paving Golden Paths](https://www.syntasso.io/post/paving-golden-paths-on-multi-cluster-kubernetes-part-1-the-theory) on the Syntasso blog.

## Get in touch
💭&nbsp;&nbsp; If learning about Kratix and platforms as products sounds intriguing and you'd like to chat with us, we'd love to hear from you. Please reach out on https://www.syntasso.io/contact-us.
