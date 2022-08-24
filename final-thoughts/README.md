👈🏾&nbsp;&nbsp; Previous: [Enhancing a Kratix Promise](/enhancing-a-promise/) <br />

# What's next?

Our last hands-on session went through how to [enhance a Kratix Promise](/enhancing-a-promise/README.md). 

Thinking back to the questions at the start of the workshop, you are the best person to decide what is most valuable to continue exploration in Kratix. 

In your context, what were the answers to these questions:

* What is the highest value service your platform provides to your application development teams? 
* How easy is it for you to provide and maintain that service?
* How easy is it for application developers to use that service?
* How can you enhance flow for your application developers and reduce effort for you and your platform team?

## Opportunities

### Configuring your platform offerings with business requirements
We saw that Postgres could be configured to identify cost centres. Perhaps you have more complex billing scenarios. Or perhaps there are quotas associated with your platform's offerings. 

Imagine customising the pipeline for one of your services, like we did with Postgres, so that it:
* sends a request to an external API to validate the user sending the request has permission to bill that particular cost centre.
* verifies any quotas that may have been setup and sends an email to inform interested parties.

### Securing your platform
As an enhancement to our Postgres Promise, we added validations that are executed when a new resource request is received by the platform cluster. 

Imagine customising your platform to have more rigorous validation so that it:
* has robust validations for resource requests coming to production environments&mdash;for example, only specific values are accepted in the request. A list of possible validations can be found [here](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.0.md#schemaObject).

### Optimising your platform
It's easy to imagine improvements to your service request pipeline and pipeline assets to make the platform more consistent and extensible.

Imagine customising the pipeline for one of your services, like we did with Postgres, so that it:
* has reusable logic in all the Promises you publish in your platform. Instead of editing the script being executed by the `postgres-request-pipeline` image, you could move logic from each step into its own dedicated image and just add these images to the `xaasRequestPipeline`. 

### Designing the right Promise for your users
Core to designing the right platform is understanding your users and what they need&mdash;you need to treat your platform as a product. 

Imagine discovering that four application teams all use the same database but each team's database has different backup strategies and different levels of monitoring. 

With Kratix, you can offer this database as-a-Service as a Promise on your platform. The Promise definition ensures that every database instance includes the paved golden path backup strategy and monitoring.

### Paving Golden Paths with complex Promise composition
Extending the idea that your platform as-a-Product provides value by giving teams what they need when they need it, Kratix makes it easy to [pave Golden Paths](https://www.syntasso.io/post/paving-golden-paths-on-multi-cluster-kubernetes-part-1-the-theory) through complex Promise composition.

A “Golden Path” is the opinionated and supported path to “build something”. Imagine a complete development environment setup&mdash;networking, integration, security, governance, compliance, and deployment&mdash;all available on-demand. By paving a Golden Path the platform team makes doing the right thing easy for the application teams.

Creating your Golden Paths on Kratix is easy: decide on, define, and install the individual Promises that are required, then define a composed Promise that brings everything together. 

An application developer goes from making separate Kratix Resource Requests to get access to separate service instances to being able to make one single Resource Request to get pre-configured, ready-to-go instances of everything they need. 

We believe composable Promises are at the core of the value that Kratix provides to platform teams. Can you imagine the most valuable composed Promise your platform could offer to your application teams? 

## Learn more 
If the idea of treating your platform as a product is new concept to you, watch this short talk by [Paula Kennedy](https://twitter.com/PaulaLKennedy) at Devoxx UK: [Crossing the Platform Gap](https://youtu.be/pAk5GReIs90) or read our two-part series about [Paving Golden Paths](https://www.syntasso.io/post/paving-golden-paths-on-multi-cluster-kubernetes-part-1-the-theory) on the Syntasso blog.

## Get in touch
💭&nbsp;&nbsp; If learning about Kratix and platforms as products sounds intriguing and you'd like to chat with us, we'd love to hear from you. Please reach out on https://www.syntasso.io/contact-us.
