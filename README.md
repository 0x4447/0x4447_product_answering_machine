# ☎️ Answering Machine

This is a small demo which tray's to simulate an automated answering machine build on top of Amazon Connect and AWS Lex. Basically this stack pushes to the limit what you can do with Amazon Connect, since Connect was never designed to a scnraio like this, But since at 0x4447 we do like to test the limits of what the cloud has to offer - we did it anyway, to see where the stack brakes. Enjoy the read.

# Overview

### AWS Lex

Lex is the AWS service that powers Amazon Alexa, a Machine Learning model that tries to understand the human voice by extracting the intent in the detected voice. The result can be received by a AWS Lambda, or directivity sent back to Amazon Connect. This allows us to create dynamic call flow based on what a user says back.

### Amazon Connect

Is a virtual call center service, that allows you co create phone call flows based on the user input, time of the day, active agents and more. This means that when someone calls a phone number that you bought, you can screen the caller with a basic number wall where they chose the right option for them. Or if we combine Connect with Lex, the caller can actually talk back, based on questions that we ask them.

# All together

On top of this two services, if we add a bunch of Lambda functions, we gain complete control over Lex and Connect, and can:

- intercept actions from Lex, and
- get notified by Connect of all the interactions with the caller.

This way we can Ask for the caller name if we don't have it in the database, or greet them with their name if the calling number have a matching name.

# Known Issues

Since we are pushing Connect to the limit there are issues that can't be solved, or would require more code to solve, which would be an over kill for this type of project. Regardless bellow is a list of those issues:

- There isn't a delay after the "I'm looking for...". Since there isn't a delay block in connect. You could solve this with a Lambda and a timeout.
- More often then not, Lex will struggle to get the message at the end of the flow. Since there isn't a general intent type that would just accept whatever is said. Lex is designed to find out meaning in a sentence. This could be solved only by AWS.

# DISCLAIMER!

This stack is available to anyone at no cost, but on an as-is basis. 0x4447 LLC is not responsible for damages or costs of any kind that may occur when you use the stack. You take full responsibility when you use it.

# Deploy

This project is described using a CloudFomration file, but sadly Lex and Connect are not yet supported by CloudFormation. before we deploy the stack we need to perform some manual work, so we have all the necessary details for the CF stack itself.

### AWS Lex

1. Go to the [Lex console](https://console.aws.amazon.com/lex/home).
1. Click the blue button `Create`.
	1. Select `Custom bot`.
	1. Type the name of your bot.
	1. Select an `Output voice` that you like. Joanna seams the more natural of them all.
	1. Set 5 min in the `Session timeout` field.
	1. And select `No` for the `COPPA` section.
	1. Click `Create`.
1. Now, on the new page, click `Create Intent`
	1. On the popup select `Create intent`.
	1. Name it `GetName`
	1. In the `Slots` section we need first to create a new entry called `first_name`. With a `Slot type` of `AMAZON.US_FIRST_NAME`, and with `Prompt` set as `Can I have your name`.
	1. Click the + button next to the slot to save it.
	1. Now that we have the Slot, in the `Sample utterances`, we are going to write a bunch of possible sentences a caller can say. You can come up with more if you'd like:
		1. `Yes sure my name is ​{first_name}​`
		1. `I'm ​{first_name}​`
		1. `My name is ​{first_name}​`
		1. `​{first_name}​`
		1. Scroll down to the bottom and click `Save intent`.
1. We are going to make another Intent, this time called `GetMessage`.
	1. For the `Slots` we are going to type `message`.
	1. For the `Slot type` we are going to select: `AMAZON.Festival` because we want the most general type possible, since we want to get the whole voice transcript, and not let Lex try to understand the meaning of what was said.
	1. For `Prompt`, type anything, since Lex won't be asking the question, we are going to fulfill the intent using our custom Lambda.
	1. For `Sample utterances`, write just one: ​`{message}​`.
	1. For `Lambda initialization and validation`, select the Message Lambda, and when the modal drops down, click `Ok`.
	1. Do the same for `Fulfillment`, where you select `AWS Lambda function`, and the Message lambda from the drop down.
	1. Scroll down to the bottom and click `Save intent`.
1. On the same page, from the left menu select `Error Handling`.
	1. For `Clarification prompts`, type something like:
		1. `Sorry, I didn't hear you.`
		1. `What's that?`
		1. `Could you please repeat that?`
	1. Set the `Maximum number of retries` to 5,
	1. And for the `Hang-up phrase` write: `Sorry, the connection is to bad. Please go to our contact page to send us an email.`
	1. Click the `+` button next to the message.
	1. Click the `x` button for the default disconnect message.
1. On the Setting tab, go to `Aliases`, and create an alias named `production` and select the latest `Bot version`. This alias is important for building the our setup.
1. On the top right corner of the page click `Build`. Once the process is done, you can test your bot, by tying any string that infers your name. In return only your name should be extracted from the message.
1. The last step is to publish the bot.
	1. Click `Publish` found in the top right corner next to `Build`
	1. From the modal, select the alias that we created in the step above.
	1. Once the process is done, the bot is ready to be used by Connect.

### Amazon Connect

1. Go to the [Connect console](https://console.aws.amazon.com/connect/home).
1. Click the blue `Add an instance` button, or `Get Started`.
	1. Check `Store users within Amazon Connect`, and write the name for the `Access URL`, and then click `Next step`.
	1. On the `Create an Administrator` page, fill out the form for a new admin user - this has nothing to do with a IAM user. Once ready click `Next step`.
	1. On the `Telephony Options` check all the options if you want, then click `Next step`.
	1. Keep the `Data storage` page as is by clicking `Next step`.
	1. Review the setup and once ready click `Create instance`
1. Click on your new Connect setup, and on the left menu, at the bottom, click `Contact flows`.
	1. In the `Amazon Lex` section select the region where you created the bot.
	1. From the `Bot` drop down menu, select the Lex bot we created in the previous setup.
	1. Click `+ add Lex Bot`.

After all of this is done we have Lex linked with Connect. And we also have the Connect Instance ARN for CloudFormation.

### CloudFormation

<a target="_blank" href="https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=zer0x4447-S3-Email&templateURL=https://s3.amazonaws.com/0x4447-drive-cloudformation/answering-machine.json">
<img align="left" style="float: left; margin: 0 10px 0 0;" src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png"></a>

All you need to do to deploy this stack is click the button to the left and follow the instructions that CloudFormation provides in your AWS Dashboard. Alternatively you can download the CF file from [here](https://s3.amazonaws.com/0x4447-drive-cloudformation/answering-machine.json).

#### What will deploy?

![Answering Machine Diagram](https://raw.githubusercontent.com/0x4447/0x4447_product_answering_machine/assets/diagram.png)

The stack takes advantage of AWS SES, AWS Lambda and DyamoDB. You'll get:

- 4x AWS Lambdas (1x CodeBuild and 1x CodePipeline for each Lambda to support auto-deployment)
- 1x DynamoDB table
- 1x SES Topic
- 1x SES Subscription

All the project resources can be found [here](https://github.com/topics/0x4447-product-answering-machine).

#### Auto deploy

The stack is set up in a such a way that any time new code is pushed to a selected branch, the CodePipeline picks up the change and updates the Lambdas for you. These are the available branches:

- **master**: the latest stable code
- **development**: unstable code that we test in our test environment - we don't recommend that you use this branch

### Amazon Connect Flow

![Call Flow](https://github.com/0x4447/0x4447_product_answering_machine/blob/assets/call_flow.png)

This part of the setup requires you to edit a JSON file to update it to your setup. But before we do that, let's start with the basic.

1. Use the URL that you created to access the Connect dashboard.
1. Once logged in, hover over the 3th icon from the left menu, and select: `Contact flows`.
1. On the `Contact flows` page, click ` Create contact flow`.
1. On the new page, name your flow, for example `Offcie`.

At this stage you can download our [contact flow that we create](https://raw.githubusercontent.com/0x4447/0x4447_product_answering_machine/assets/call_flow.json). This is the file that you'll have to edit, and replace some ARNs to lambda functions to your owns. Sadly there isn't an automatic way of doing this.

This is a list of places where you have to replace the ARNs:

- modules[2].parameters[0].value = the Arn of the `0x4447_answering_machine_name_get` lambda
- modules[12].parameters[0].value = the Arn of the `0x4447_answering_machine_name_save` lambda
- modules[16].parameters[0].value = the Arn of the `0x4447_answering_machine_notification` lambda

Once the file is changed you can import the flow: at the top right corner you have the `Save` button, which has an arrow pointing down. Click on it and select `Import flow (beta)`. Select the file, and upload it. Once the flow will show up on the page, **don't forget** to click `Publish`. At this stage we have to attach the flow we made with a phone number. To do so, follow this steps:

1. From the left menu list, select the 3th icon, and and click `Phone numbers`.
1. Click `Claim a number` that you can find on the right side of the page.
1. On the new page, select the Country
1. From the list of numbers, select one that you like the most.
1. Add a simple `Description`
1. From the `Contact flow / IVR` drop down, select the flow that you created.
1. Click `Save`.

# Pricing

All resources deployed via this stack will potentially cost you money. But you'd have to do the following for this to happen:

- Invoke Lambdas over 1,000,000 times a month
- Send over 1000 SNS emails a month
- Send over 1 GB of data a month through SNS
- Exceed 100 build minutes on CodeBuild
- $1 per active CodePipeline (must run at least once a month to be considered active)

The only payment you'll encounter from Day One is with Connect, since they bill per minutes of use, check their [pricing](https://aws.amazon.com/connect/pricing/) page to get an idea of costs.

# How to work with this project

When you want to deploy the stack, the only file you should be interested in is the `CloudFormation.json` file. If you'd like to modify the stack, we recommend that you use the [Grapes framework](https://github.com/0x4447/0x4447-cli-node-grapes), which was designed to make it easier to work with the CloudFormation file. If you'd like to keep your sanity, never edit the main CF file 🤪.

# The End

If you enjoyed this project, please consider giving it a 🌟. And check out our [0x4447 GitHub account](https://github.com/0x4447), where you'll find additional resources you might find useful or interesting.

## Sponsor 🎊

This project is brought to you by 0x4447 LLC, a software company specializing in building custom solutions on top of AWS. Follow this link to learn more: https://0x4447.com. Alternatively, send an email to [hello@0x4447.email](mailto:hello@0x4447.email?Subject=Hello%20From%20Repo&Body=Hi%2C%0A%0AMy%20name%20is%20NAME%2C%20and%20I%27d%20like%20to%20get%20in%20touch%20with%20someone%20at%200x4447.%0A%0AI%27d%20like%20to%20discuss%20the%20following%20topics%3A%0A%0A-%20LIST_OF_TOPICS_TO_DISCUSS%0A%0ASome%20useful%20information%3A%0A%0A-%20My%20full%20name%20is%3A%20FIRST_NAME%20LAST_NAME%0A-%20My%20time%20zone%20is%3A%20TIME_ZONE%0A-%20My%20working%20hours%20are%20from%3A%20TIME%20till%20TIME%0A-%20My%20company%20name%20is%3A%20COMPANY%20NAME%0A-%20My%20company%20website%20is%3A%20https%3A%2F%2F%0A%0ABest%20regards.).
