# Login into IBM Cloud
This exercise will prepare you to get your hands dirty with IBM Cloud Functions.

*Once you have completed this exercise, you will be…*

- **Ready to experience FaaS.**

Once this exercise is finished, we will be able to create simple serverless functions using IBM Cloud Functions by following the rest of
the exercises!

## Table Of Contents
* [Background](#Background)
* [IBM Cloud login](#Login)

## Background
IBM Cloud Functions allows the users to create their functions (actions and triggers) to reside or be bundled under `namespace`. Here in, namespace is the combination of your CF org and CF space in this way = `<CF org>_<CF space>`. So, we need to login and target the org and space to get started.

## Login

On your terminal, enter `ibmcloud login --sso`.
This will generate a One time login passcode for you.

```
▶ ibmcloud login --sso
API endpoint: https://cloud.ibm.com

Get One Time Code from https://identity-2.uk-south.iam.cloud.ibm.com/identity/passcode to proceed.
Open the URL in the default browser? [Y/n]>
```
Enter `y` and you would see your browser window open. Copy the passcode and paste it in your terminal

Next step: Select the account
Next step: Select the region (pick either us-south, us-east, eu-gb or eu-de, where ever you have created a **CF org and CF space**)

After the success, you will get the details about the account.

Please notice the tip
```
Tip: If you are managing Cloud Foundry applications and services
- Use 'ibmcloud target --cf' to target Cloud Foundry org/space interactively, or use 'ibmcloud target --cf-api ENDPOINT -o ORG -s SPACE' to target the org/space.
- Use 'ibmcloud cf' if you want to run the Cloud Foundry CLI with current IBM Cloud CLI context.
```

We will now follow the above instruction and do `ibmcloud target --cf`.
Here, select any org and then select any space.

You are now ready.
