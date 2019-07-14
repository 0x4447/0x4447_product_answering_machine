# ☎️ Answering Machine

This small demo tries to simulate an automated answering machine built on top of Amazon Connect and AWS Lex. The stack basically pushes what you can do with Amazon Connect to the limit because Connect was never designed to be used in a scenario like this. But we do like to test the limits of what the cloud has to offer here at 0x4447, so we did it anyway, just to see where the stack would break. So enjoy the read.

# Overview

### AWS Lex

Lex is the AWS service that powers Amazon Alexa, a Machine Learning model designed to understand the human voice by extracting the intent in the detected voice.
The service also allows interfacing with AWS Lambda to control how Lex processes the intents.

### Amazon Connect

Amazon Connect is a virtual call center service we can use to create phone call flows based on user input, time of day, active agents, and more. This means that when someone calls a phone number we've bought, we can screen the caller with a basic number wall of options the caller can choose from. Or, if Connect is combined with Lex, the caller can talk back based on questions asked.

# All Together

In addition to these two services, we can add a number of Lambda functions to gain complete control over Lex and Connect. We can do the following:

- Intercept actions from Lex
- Receive notifications from Connect about all interactions with the caller

This enables us to ask for the caller's name if it doesn't appear in the database or use the caller's name in greeting if the calling number has a matching name.

# Known Issues

We're pushing Connect to the limit, so there will be issues that can't be solved and issues that require more code to solve, which would lead to overkill for this type of project. Regardless, those issues are listed below:

- There isn't a delay after the "I'm looking for..." since there isn't a delay block in connect. You could solve this with a Lambda and a timeout.
- More often than not, Lex will struggle to get the message at the end of the flow because there isn't a general intent type that will just accept whatever is said. Lex is designed to determine the meaning of a sentence. The only way to solve this would be for AWS to add a new slot that just transcribes what is said.

# DISCLAIMER!

This stack is available to anyone at no cost, but on an as-is basis. 0x4447, LLC is not responsible for damages or costs of any kind that may occur when you use the stack. You take full responsibility when you use it.

# Deploy

This project is described using a CloudFormation file. Sadly, Lex and Connect are not yet supported by CloudFormation, and before deploying the stack, we need to perform some manual tasks to gather all necessary details for the CF stack itself.

### AWS Lex

1. Go to the [Lex console](https://console.aws.amazon.com/lex/home).
1. Click the blue button `Create`.
	1. Select `Custom bot`.
	1. Type `Office` as the name of your bot.
	1. Select your favorite `Output voice`. Joanna seems to be the most natural-sounding of them all.
	1. Set five minutes in the `Session timeout` field.
	1. Select `No` in the `COPPA` section.
	1. Click `Create`.
1. On the new page, click `Create Intent`.
	1. On the pop-up, select `Create intent`.
	1. Name it `GetName`
	1. In the `Slots` section, we first need to create a new entry called `first_name` with a `Slot type` of `AMAZON.US_FIRST_NAME` and a `Prompt` set as `Can I have your name`.
	1. Click the `+` button next to the slot to save it.
	1. Now that we have the Slot, we'll write some possible sentences a caller might use. You can come up with more if you like, but here are a few:
		1. `Yes sure my name is ​{first_name}​`
		1. `I'm ​{first_name}​`
		1. `My name is ​{first_name}​`
		1. `​{first_name}​`
		1. Scroll down to the bottom and click `Save intent`.
1. We're going to make another Intent called `GetMessage`.
	1. For the `Slots`, type `message`.
	1. For the `Slot type`, select `AMAZON.Festival`. We want the most general type possible, because we want to get the entire voice transcript rather than letting Lex try to understand the meaning of what's said.
	1. For `Prompt`, type anything. Lex won't be asking the question, our lambda will.
	1. Click the `+` button next to the slot to save it.
	1. Scroll down to the bottom and click `Save intent`.
1. On the same page, select `Error Hanling` from the left menu.
	1. For `Clarification prompts`, type something like:
		1. `Sorry, I didn't hear you.`
		1. `What's that?`
	1. Set the `Maximum number of retries` to three.
	1. For the `Hang-up phrase`, write: `Sorry, the connection is bad. Please go to our contact page to send us an email.`
	1. Click the `+` button next to the message.
	1. Click the `x` button for the default disconnect message.
1. On the Settings tab, go to `Aliases` and create an alias named `production`. Then select the latest `Bot version`. This alias is important for building our setup.
1. On the page's top right corner, click `Build`. Once the process is done, you can test your bot by tying any string that infers your name. In return, only your name should be extracted from the message.
1. The last step is to publish the bot.
	1. Click `Publish` at the top right corner next to `Build`.
	1. From the modal, select the alias that we created in the step above.
	1. Once the process is done, the bot is ready to be used by Connect.

### Amazon Connect

1. Go to the [Connect console](https://console.aws.amazon.com/connect/home).
1. Click the blue `Add an instance` button or `Get Started`.
	1. Check `Store users within Amazon Connect`, write the name for the `Access URL`, and click `Next step`.
	1. On the `Create an Administrator` page, fill out the form for a new admin user. This has nothing to do with a IAM user. Once ready, click `Next step`.
	1. On `Telephony Options`, check all options if you like, then click `Next step`.
	1. Keep the `Data storage` page as is by clicking `Next step`.
	1. Review the setup and click `Create instance` when ready.
1. Click on your new Connect setup and click `Contact flows` at the bottom of the left menu.
	1. In the `Amazon Lex` section, select the region where you created the bot.
	1. From the `Bot` drop-down menu, select the Lex bot we created in the previous setup.
	1. Click `+ add Lex Bot`.

When this is done, we have Lex linked with Connect and the Connect Instance ARN for CloudFormation.

### CloudFormation

<a target="_blank" href="https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=zer0x4447-Answering-Machine&templateURL=https://s3.amazonaws.com/0x4447-drive-cloudformation/answering-machine.json">
<img align="left" style="float: left; margin: 0 10px 0 0;" src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png"></a>

To deploy this stack, all you need to do is click the button to the left and follow the instructions that CloudFormation provides in your AWS Dashboard. Alternatively, you can download the CF file from [here](https://s3.amazonaws.com/0x4447-drive-cloudformation/answering-machine.json).

#### What Will Deploy?

![Answering Machine Diagram](https://raw.githubusercontent.com/0x4447/0x4447_product_answering_machine/assets/diagram.png)

The stack takes advantage of AWS SES, AWS Lambda, and DyamoDB. You'll get:

- 4x AWS Lambda (1x CodeBuild and 1x CodePipeline for each Lambda to support auto-deployment)
- 1x DynamoDB table
- 1x SES Topic
- 1x SES Subscription

All project resources can be found [here](https://github.com/topics/0x4447-product-answering-machine).

#### Auto Deploy

The stack is set up in a such a way that when new code is pushed to a selected branch, the CodePipeline picks up the change and updates the Lambdas for you. These are the available branches:

- **master**: the latest stable code
- **development**: unstable code that we test in our test environment - we don't recommend that you use this branch

### AWS Lex Update

Now that we've deployed the stack, we have to go back to Lex for a moment to set one of the deployed Lambda to process the intent.

1. Go to the [Lex console](https://console.aws.amazon.com/lex/home).
1. Click on the `Offcie` bot we previously created.
1. For the Intent, click on the `GetMessage`.
1. In the `Lambda initialization and validation` section:
	1. Check `Initialization and validation code hook`.
	1. From the drop down bellow, select the Message Lambda, and click `Ok` when the modal drops down.
1. For the `Fulfillment` section below:
	1. Select `AWS Lambda function`.
	1. From the drop down below, select the Message Lambda.
1. Scroll down to the bottom and click `Save intent`.

### Amazon Connect Flow

![Call Flow](https://github.com/0x4447/0x4447_product_answering_machine/blob/assets/call_flow.png)

This part of the setup requires that you edit a JSON file to update it to your setup. But before we do that, let's start with the basics.

1. Use the URL you created to access the Connect dashboard.
2. Once logged in, hover over the third icon from the left menu and select `Contact flows`.
3. On the `Contact flows` page, click `Create contact flow`.
4. On the new page, name your flow (for example, `Office`).

At this stage, you can download our [contact flow that we create](https://raw.githubusercontent.com/0x4447/0x4447_product_answering_machine/assets/call_flow.json). This is the file that you'll have to edit, replacing some ARNs of the Lambda functions with your own. Sadly, there's no automatic way to do this. You can find the Lambda ARNs in the output section of the deployed stack in CloudFormation.

Once you change the file, you can import the flow. At the top right corner is the `Save` button, which has a downward-pointing arrow. Click on it and select `Import flow (beta)`. Select the file and upload it. Once the flow shows up on the page, **don't forget** to click `Publish`. At this point, we have to attach the flow we made with a phone number. To do so, follow these steps:

1. Select the  third icon on the left menu list and click `Phone numbers`.
1. Click `Claim a number` on the right side of the page.
1. Select the country on the new page.
1. Select the number you like best from the list of numbers.
1. Add a simple `Description`.
1. Select the flow you created from the `Contact flow/IVR` drop-down.
1. Click `Save`.

# Pricing

All resources deployed via this stack will potentially cost you money, but you'd have to do the following for this to happen:

- Invoke Lambdas over 1,000,000 times per month
- Send over 1000 SNS emails per month
- Send over 1 GB of data per month through SNS
- Exceed 100 build minutes on CodeBuild
- Exceed the read and write capacity of DynamoDB
- $1 per active CodePipeline (must run at least once a month to be considered active)

The only payment you'll encounter from Day One is with Connect, since they bill per minutes of use. Check their [pricing](https://aws.amazon.com/connect/pricing/) page to get an idea of the cost.

# How to work with this project

When you want to deploy the stack, the only file you should be interested in is the `CloudFormation.json` file. If you'd like to modify the stack, we recommend that you use the [Grapes framework](https://github.com/0x4447/0x4447-cli-node-grapes), which was designed to make it easier to work with the CloudFormation file. If you'd like to keep your sanity, never edit the main CF file 🤪.

# The End

If you enjoyed this project, please consider giving it a 🌟. And check out our [0x4447 GitHub account](https://github.com/0x4447), where you'll find additional resources you might find useful or interesting.

## Sponsor 🎊

This project is brought to you by 0x4447 LLC, a software company specializing in building custom solutions on top of AWS. Follow this link to learn more: https://0x4447.com. Alternatively, send an email to [hello@0x4447.email](mailto:hello@0x4447.email?Subject=Hello%20From%20Repo&Body=Hi%2C%0A%0AMy%20name%20is%20NAME%2C%20and%20I%27d%20like%20to%20get%20in%20touch%20with%20someone%20at%200x4447.%0A%0AI%27d%20like%20to%20discuss%20the%20following%20topics%3A%0A%0A-%20LIST_OF_TOPICS_TO_DISCUSS%0A%0ASome%20useful%20information%3A%0A%0A-%20My%20full%20name%20is%3A%20FIRST_NAME%20LAST_NAME%0A-%20My%20time%20zone%20is%3A%20TIME_ZONE%0A-%20My%20working%20hours%20are%20from%3A%20TIME%20till%20TIME%0A-%20My%20company%20name%20is%3A%20COMPANY%20NAME%0A-%20My%20company%20website%20is%3A%20https%3A%2F%2F%0A%0ABest%20regards.).
