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

Platform teams in any form add tremendous value to an organisation. They reduce cognitive load for application teams, which enables those teams to have faster 'flow'. 

However, reducing cognitive load for application teams generally, in our experience, increases toil for platform team members. The closer a platform team gets to effective enablement of application teams, the higher the likelihood is that behind the scenes, platform team members are stretched beyond capacity. 

Here is an experience you might have on a platform team as you work to support your application teams who need data storage for their applications.

What you aim to deliver is:

### A standardised development tool...
Through a Slack message, you happen to learn that four application teams are all using four different flavours of Postgres. Each team's database is in a different cloud, each has a different backup strategy, and each uses different levels of monitoring.

### ...optimised for scale...
After learning about the four 'shadow IT' Postgres databases, you poll application teams. Turns out there are at least six more teams using a mix of cloud providers to get Postgres databases to suit their needs. You need to manually intervene in each team's backlog to audit the situation and ensure each database is healthy and compliant. 

### ...pre-configured to meet your business requirements...
For each of those Postgres databases, you have complex billing scenarios where you need to enforce quotas (via a script); you need to send a request to an external API to validate permission to bill that particular cost centre (via another script); and you need to inform interested stakeholders via email (via sharing a manual process with each application team).

### ...and secured according to your policies.
In addition to more complex billing scenarios, you need to implement more rigorous security policies. You need to ensure that configuration values are acceptable according to the broader security protocols (manually, one-by-one); you need to verify proper signoff has happened before deployments move forward (manually, one-by-one); and you need to make sure that teams have confidence that they are shipping with dependencies that are risk-free for the organisation (with third-party scanning software that teams need to manage themselves).
<br/>
<br/>

It's clear from this example that there are _a lot_ of platform concerns when delivering a service. With Kratix you can develop a platform that will easily _offer standardised development tools_, _optimised for scale_, that is _pre-configured to meet your business requirements_ and _secured according to your policies_.

Stepping back, the beauty of Kratix is in its flexibility. It allows you to encapsulate a service in a Promise with a robust Request Pipeline. Our example above highlights some of the most common challenges where Kratix has helped platform teams we've worked with move past toil toward higher-value enablement. But we know your organisational challenges are unique, and Kratix is built so that you can adapt it to your context.

## Designing the right Golden Paths with multiple Promises
The value of your platform increases dramatically when you offer application teams tailored Golden Paths. A [“Golden Path”](https://www.syntasso.io/post/paving-golden-paths-on-multi-cluster-kubernetes-part-1-the-theory) is the opinionated and supported path to “build something”. Imagine a complete development environment setup&mdash;networking, integration, security, governance, compliance, and deployment&mdash;all available on-demand. By paving a Golden Path the platform team makes doing the right thing easy for application teams.

Creating a Golden Path on Kratix is easy: decide on, define, and install the individual Promises that are required, then define a higher-level Promise that brings those individual Promises together. 

An application developer goes from making separate Kratix Resource Requests to get access to separate service instances to being able to make one single Resource Request to get pre-configured, ready-to-go instances of everything they need. 

We believe composable Promises are at the core of the value that Kratix provides to platform teams. 

Take a moment to imagine the most valuable bundle of services that your platform could offer to your application teams. Now that you've had experience building a platform with Kratix, translate that bundle of services into a composed Promise using Kratix. 

## Learn more 
If the idea of treating your platform as-a-Product is new concept to you, watch this short talk by [Paula Kennedy](https://twitter.com/PaulaLKennedy) at Devoxx UK: [Crossing the Platform Gap](https://youtu.be/pAk5GReIs90) or read our two-part series about [Paving Golden Paths](https://www.syntasso.io/post/paving-golden-paths-on-multi-cluster-kubernetes-part-1-the-theory) on the Syntasso blog.

## Get in touch
💭&nbsp;&nbsp; If learning about Kratix and platforms as products sounds intriguing and you'd like to chat with us, we'd love to hear from you. Please reach out on https://www.syntasso.io/contact-us.
