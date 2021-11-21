<a  href="https://www.twilio.com">
<img  src="https://static0.twilio.com/marketing/bundles/marketing/img/logos/wordmark-red.svg"  alt="Twilio"  width="250"  />
</a>
 
# Beyond Inc Customer Satisfaction Survey with Twilio Function, Studio and Airtable


## About

This repo provides our solution for February Build Challenge. Below is a copy of our Twilio Function code and Studio Flow JSON as well as Airtable database that we used to send the customers survey questions over text messages and store the response in Airtable. 

## Problem Statement

Beyond, Inc. provides debt consolidation on behalf of their customers. Beyond works with their customers' creditors to consolidate debt obligations into a single monthly payment. Beyond has reached out to Twilio to address their challenge of sending batch (bulk) follow-up customer satisfaction surveys via SMS. They’d like to better understand their customer’s experience and potential areas for improvement.


### How it works

There are 3 components involved Twilio Functions, Twilio Studio, Airtable. 
A curl command triggers Twilio Function. Twilio Functions is used to read the customer information from Airtable for those customer's that we haven't reached out to in the past.  Twilio Function triggers the Twilio Studio Flow to send the sms asking for an overall rating. Based on the end users response, the Airtable database is updated with the responses using Twilio Funcitons.



## Features

- Using Twilio functions, programmatically read the Airtable DB and execute the Twilio Studio for each customer with survey_status == pending
- Twilio Studio flow handling the logic to send sms and wait to collect the response. Once the response is received, we pass it to Twilio functions.
- Twilio Functions updates the Airtable DB with the received responses.

## Set up



### Requirements

- A Twilio account - [sign up](https://www.twilio.com/try-twilio)
- Airtable Account - [sign up](https://airtable.com/signup)

### Twilio Account Settings

Collect the TW A/c sid and Auth token
| Config&nbsp;Value | Description                                                                                                                                                  |
| :---------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Account&nbsp;Sid  | Your primary Twilio account identifier - find this [in the Console](https://www.twilio.com/console).                                                         |
| Auth&nbsp;Token   | Used to authenticate - [just like the above, you'll find this here](https://www.twilio.com/console).                                                         |

### Setup Airtable 

- Copy [this](https://airtable.com/shry8Lw3YDbl8652e/tbl8x4C5lqTHTBHU4/viwSlkekLmErSZHLV?blocks=hide) Airtable base by hitting “Copy Base” in the upper right hand corner. Name your base “beyondinc” and your table “contacts” & "survey". Note that you will need to add your own phone number in the Phone_Number column for testing purposes. Make sure that you use E164 format (e.g. +1XXXXXXXXXX) for phone numbers stored in Airtable. Also, be sure to set the Survey_Status column to pending for the number(s) you are testing with. 

- You need the Airtable API key and a Base ID for the next step in building the prototype. Generate an [Airtable API key](https://support.airtable.com/hc/en-us/articles/219046777-How-do-I-get-my-API-key-) on your [account page](https://airtable.com/account). Get the [base ID](https://airtable.com/api) from the auto-generated API documentation when you click on the beyondinc base. You can pull it from the URL, or the introduction section; it’s prepended with “app”.



### Add Serverless Code


-  We will copy some code to read and update data from Airtable using [Twilio Functions](https://www.twilio.com/console/functions/overview/services). Open the [Services dashboard](https://www.twilio.com/docs/runtime/functions) and click “Create Service”. Give your new service the name “beyondInc”. Under settings click on Environment Variables and add your Airtable credentials as AIRTABLE_API_KEY and AIRTABLE_BASE_ID
- In settings under Dependencies, add the airtable helper library -- no need to supply a version number
- Now you will create a new Function. Click on the blue Add button and name your function the same as the file name present in folder /functions (keep the leading slash). Delete the placeholder code. Copy paste the code from the file. 
- Follow step 3 and create a function for all the files present in the folder /functions. 
- Make all the files public so that we can make an api call and trigger them.

### Add Studio funciton Code

- Navigate to Twilio studio on the Twilio console and create a flow named beyondinc and click on import json.
- Copy paste the json file present under folder studio flow and paste it.

### Trigger Studio Function

To start your Studio flow, make a REST API call to the start-survey function endpoint. You can do this using a tool like Postman, running a shell script, or running a cURL.

The curl request looks like this 
<em> curl -X POST https://beyondinc-7271.twil.io/start-survey -u "auth token:account sid". </em> 
Replace the URL with your start-survey function URL(click on the function start-survey and click copy url), and the auth token and account SID with your own values, which can be found on the Twilio Console.

## Challenges

- We were following this [blog](https://www.twilio.com/blog/automated-vaccine-appointment-system-twilio-studio) to read and update data to Airtable. We struggled for almost a day to conver a Map to a JSON object. The code shown in this blog doesn't accurately copy/move data from a Map to JSON. We have reached out to Paul Kamp to address this issue.
- We assumed that since we were triggering Twilio Studio from Twilio Functions, we did not need to configure Incoming Message Webhook URL in Twilio Console. We struggled for a few minutes before we realized that Incoming Message Webhook URL needs to be configured even if the flow has already been triggered for a user and the state is maintained by Twilio Studio
- Although the Regex for incoming response for the initial rating was pretty straightforward for this use case ( [1-4] ) . It took us some trial and error to figure this out.
- We somehow managed to run an infinite loop that queued up 1000s of messages. We were eventually able to make it stop by making the functions Private. While it was painful for 10 minutes to have your phone constantly ringing, we learned that we should always proof-read out code before publishing/triggering it to avoid a catastrophy. 
- Although the Regex for incoming response for the initial rating was pretty straightforward for this use case ( [1-4] ) . It took us some trial and error to figure this out.
- Twilio Functions has a 10 second timeout. In order to address this, we broke down our code into smaller tasks. Each task is a separate function. 
- While troubleshooting, we noticed that "Enable Live Log" button would move to disable on it's own in the middle of the testing. We couldn't figure out why this was happening. We suspect it disables itself after sometime. Due to this, we had to restart troubleshooting/testing a few times. 
- Not really a challenge, but we noticed that there were two "Logs" options in Twilio Studio Beta Console, which was confusing. 

<a  href="https://code.hq.twilio.com/bdhami/beyond/blob/master/assets/studio-bug.png">
<img  src="https://code.hq.twilio.com/bdhami/beyond/blob/master/assets/studio-bug.png"  alt="Studio Bug"  width="250"  />
</a>


## Improvements to current design
- Ability to make the questions dynamic, rather than hard coding them directly into a Studio Flow. We could possibly leverage Airtable to store questions for different types of services, have Twilio Functions read the corresponding question based on the Service Used by the end user and then pass this to Twilio Studio to be delivered to the end user.
