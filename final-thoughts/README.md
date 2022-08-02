## What's next?

One option is to continue refining exising promises to see how Kratix can support scaled and complex scenarios. For example, while our pipeline example is quite simple, you could:

1. Send a request to an external API to validate the user sending the request has permission to bill that particular cost centre.
2. Verify any quotas that may have been setup, and fire an email to inform an interested party of this action

Furthermore, instead of editing the script being executed by the `postgres-request-pipeline` image, you could move logic from each step into its own dedicated image and just add these images to the `xaasRequestPipeline`. This would allow you to re-use the logic in all other Promises you publish in your platform.

We added validations that are executed when a new resource request is received by the platform cluster. In a production environment, we will want to put more robust validations in place, such as only accepting specific values. A more complete list of possible validations can be found [here](https://github.com/OAI/OpenAPI-Specification/blob/main/versions/3.0.0.md#schemaObject).

You can also dive right in to designing your platform. This requires thinking about the right level of abstraction for the capabilities you want to deliver. For example, you may want every database to include a backup strategy, a monitoring dashboard, and a UI client. You'll need to treat your platform as a product, and reach out to your platform users and stakeholders to ensure you're providing the products they need. If this is a new concept for you, you may want to learn more by watching a short talk on the topic by [Paula Kennedy](https://twitter.com/PaulaLKennedy) at Devoxx UK: [Crossing the Platform Gap](https://youtu.be/pAk5GReIs90).

If that sounds intriguing and you'd like to chat with us about anything Platform, we'd love to hear from you. 

💭&nbsp;&nbsp; Please reach out on https://www.syntasso.io/ and we'll be happy to schedule a call.