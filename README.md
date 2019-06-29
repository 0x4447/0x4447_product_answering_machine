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


# DISCLAIMER!

This stack is available to anyone at no cost, but on an as-is basis. 0x4447 LLC is not responsible for damages or costs of any kind that may occur when you use the stack. You take full responsibility when you use it.

# How to deploy

<a target="_blank" href="https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=zer0x4447-S3-Email&templateURL=https://s3.amazonaws.com/0x4447-drive-cloudformation/s3-email.json">
<img align="left" style="float: left; margin: 0 10px 0 0;" src="https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png"></a>

All you need to do to deploy this stack is click the button to the left and follow the instructions that CloudFormation provides in your AWS Dashboard. Alternatively you can download the CF file from [here](https://s3.amazonaws.com/0x4447-drive-cloudformation/s3-email.json).

# What will deploy?

![S3-Email Diagram](https://raw.githubusercontent.com/0x4447/0x4447-product-s3-email/assets/diagram.png)

The stack takes advantage of AWS S3, AWS SES, AWS Lambda, and the AWS Trigger system to tie everything together. You'll get:

- 4x AWS Lambdas (1x CodeBuild and 1x CodePipeline for each Lambda to support auto-deployment)
- 1x DynamoDB table

All project resources can be found [here](https://github.com/topics/0x4447-product-s3-email).

# Auto deploy

The stack is set up in a such a way that any time new code is pushed to a selected branch, the CodePipeline picks up the change and updates the Lambda for you. These are the available branches:

- **master**: the latest stable code
- **development**: unstable code that we test in our test environment - we don't recommend that you use this branch

# Manual work

Sadly at the time of writing this Lex and Connect are not supported by CloudFormation, this means that once the stack is deployed you will configure this two services by hand.

# AWS Lex

Instructions

# Amazon Connect

Instructions

# How to work with this project

When you want to deploy the stack, the only file you should be interested in is the `CloudFormation.json` file. If you'd like to modify the stack, we recommend that you use the [Grapes framework](https://github.com/0x4447/0x4447-cli-node-grapes), which was designed to make it easier to work with the CloudFormation file. If you'd like to keep your sanity, never edit the main CF file 🤪.

# The End

If you enjoyed this project, please consider giving it a 🌟. And check out our [0x4447 GitHub account](https://github.com/0x4447), where you'll find additional resources you might find useful or interesting.

## Sponsor 🎊

This project is brought to you by 0x4447 LLC, a software company specializing in building custom solutions on top of AWS. Follow this link to learn more: https://0x4447.com. Alternatively, send an email to [hello@0x4447.email](mailto:hello@0x4447.email?Subject=Hello%20From%20Repo&Body=Hi%2C%0A%0AMy%20name%20is%20NAME%2C%20and%20I%27d%20like%20to%20get%20in%20touch%20with%20someone%20at%200x4447.%0A%0AI%27d%20like%20to%20discuss%20the%20following%20topics%3A%0A%0A-%20LIST_OF_TOPICS_TO_DISCUSS%0A%0ASome%20useful%20information%3A%0A%0A-%20My%20full%20name%20is%3A%20FIRST_NAME%20LAST_NAME%0A-%20My%20time%20zone%20is%3A%20TIME_ZONE%0A-%20My%20working%20hours%20are%20from%3A%20TIME%20till%20TIME%0A-%20My%20company%20name%20is%3A%20COMPANY%20NAME%0A-%20My%20company%20website%20is%3A%20https%3A%2F%2F%0A%0ABest%20regards.).
