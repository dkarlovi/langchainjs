---
sidebar_label: Agent with AWS Lambda
hide_table_of_contents: true
---

# Agent with AWS Lambda Integration

Full docs here: https://docs.aws.amazon.com/lambda/index.html

**AWS Lambda** is a serverless computing service provided by Amazon Web Services (AWS), designed to allow developers to build and run applications and services without the need for provisioning or managing servers. This serverless architecture enables you to focus on writing and deploying code, while AWS automatically takes care of scaling, patching, and managing the infrastructure required to run your applications.

By including a AWSLambda in the list of tools provided to an Agent, you can grant your Agent the ability to invoke code running in your AWS Cloud for whatever purposes you need.

When an Agent uses the AWSLambda tool, it will provide an argument of type `string` which will in turn be passed into the Lambda function via the `event` parameter.

This quick start will demonstrate how an Agent could use a Lambda function to send an email via [Amazon Simple Email Service](https://aws.amazon.com/ses/). The lambda code which sends the email is not provided, but if you'd like to learn how this could be done, see [here](https://repost.aws/knowledge-center/lambda-send-email-ses). Keep in mind this is an intentionally simple example; Lambda can used to execute code for a near infinite number of other purposes (including executing more Langchains)!

```typescript
import { OpenAI } from "langchain";
import { SerpAPI, AWSLambda } from "langchain/tools";
import { initializeAgentExecutor } from "langchain/agents";
const model = new OpenAI({ temperature: 0 });
const emailSenderTool = new AWSLambda(
  "email-sender", // name: tell the Agent what this tool is called
  "Sends an email with the specified content to testing123@gmail.com", // description: tell the Agent precisely what the tool does
  {
    region: "us-east-1", // the region in which the function is deployed in the AWS cloud
    accessKeyId: "abc123", // the access key for a user which has the IAM permissions necessary to invoke the function
    secretAccessKey: "xyz456", // the secret access key for a user which has the IAM permissions necessary to invoke the function
    functionName: "SendEmailViaSES", // the function name as seen in the AWS Lambda console
  }
);
const tools = [emailSenderTool, new SerpAPI("api_key_goes_here")];
const executor = await initializeAgentExecutor(
  tools,
  model,
  "zero-shot-react-description"
);
const input = `Find out the capital of Croatia. Once you have it, email the answer to testing123@gmail.com.`;
const result = await executor.call({ input });
console.log(result);
```
