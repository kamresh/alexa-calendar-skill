# How to Build a Calendar Reader for Alexa using node js

The First prerequisite is to have node js installed. This application will work on node js version above 0.10.36. Post node js installation alexa sdk needs to be installed which makes development easier.
Command to install alexa sdk along with ical module (used for calendar reading) is npm install --save alexa-sdk ical .


The user of this skill will be able to ask things like:

   * "What is happening tonight?
   * "What events are going on next Monday?"
   * "Tell me more about the second event."

One can plug own public calendar feed (an .ICS file) into the sample provided, so that it can interact with provided calendar in the same way.

Using the [Alexa Skills Kit](https://developer.amazon.com/alexa-skills-kit), one can build an application that can receive and respond to voice requests made on the Alexa service. In this tutorial, you’ll build a web service to handle requests from Alexa and map this service to a skill in the Amazon Developer Portal.

After completing this tutorial, below functions can be done

   * Create a calendar reader skill - This tutorial will walk first-time Alexa skills developers through all the required steps involved in creating a skill that reads calendar data, called "My Calendar".
   * Understand the basics of VUI design - Creating this skill will help understand the basics of creating a working Voice User Interface (VUI) while using a cut/paste approach to development.
   * Use JavaScript/Node.js and the Alexa Skills Kit to create a skill.
   * Test skill using service simulator 
   * Interact with a calendar (.ics file) using voice commands.
  

## Step 1. Setting up Alexa Skill in the Developer Portal
   
Skills are managed through the Amazon Developer Portal. Lambda function created above needs to be linked to a skill defined in the Developer Portal.

1.  Navigate to the Amazon Developer Portal. Sign in or create a free account.

    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-01-001._TTH_.png)

2.  Once signed in, navigate to Alexa and select **"Getting Started"** under Alexa Skills Kit.

    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-01-002._TTH_.png)
 
3.  Here is where one can define and manage your skill. Select **"Add a New Skill"**

    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-01-003._TTH_.png)
 
4.  There are several choices to make on this page, so we will cover each one individually.
    1. Choose the language to start with. 
    2. Make sure the radio button for the Custom Interaction Model is selected for “Skill Type”.
    3. Add the name of the skill. Give your skill a name that is simple and memorable, like "My Calendar." The name will be the one that shows up in the Alexa App when users are looking for new skills. 
    4. Add the invocation name. This is what users will actually say to start using skill. Like in Step #5, use one or two words, because users will have to say this every time they want to interact with your skill.
    5. Under "Global Fields," select "no" for Audio Player, as our skill won't be playing any audio.  
    6. Select **Next**.
   
    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%203.png)

5.  Next, we need to define our skill’s interaction model. Let’s begin with the intent schema. In the context of Alexa, an intent represents an action that fulfills a user’s spoken request.
   
    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-01-005._TTH_.png)
 
6.  Check the Intent Schema below. This is written in JSON and provides the information needed to map the intents we want to handle programmatically. 
    
    Below a collection of built-in intents to simplify handling common user tasks is shown, and then two additional custom intents for querying our calendar source. Intents can optionally have arguments called slots.  For our two custom intents, "searchIntent" and "eventIntent," we will use these slots to define the data type that we are expecting the user to provide.
    
    Slots are predefined data types that we expect the user to provide.  This helps resolve data to a standardized format (like an enum).  For example, you could say "next Monday," and it would be able to return a specific date.  This data also becomes training data for Alexa's Natural Language Understanding (NLU) engine.
   
   
    ```JSON
    {
      "intents":  [
        { "intent": "AMAZON.HelpIntent", "slots": [] },
        { "intent": "AMAZON.StopIntent", "slots": [] },
        { "intent": "AMAZON.RepeatIntent", "slots": [] },
        { "intent": "AMAZON.CancelIntent", "slots": [] },
        { "intent": "AMAZON.YesIntent", "slots": [] },
        { "intent": "AMAZON.NoIntent", "slots": [] },
        { "intent": "searchIntent", "slots": 
                        [{ "name": "date", "type": "AMAZON.DATE" }] },
        { "intent": "eventIntent", "slots": 
                        [{ "name": "number", "type": "AMAZON.NUMBER" }]}
      ]
    }
    ```
    
    Here we have defined six different built-in intents: Help, Stop, Repeat, Cancel, Yes, and No.  Our two custom intents, searchIntent and eventIntent, each have a slot defined for them.  This means that we expect a specific data type from the user when they use these intents.  You will see how this works more clearly when we define our sample utterances below.

7.  The next step is to build the utterance list.  This is meant to be an thorough, well-thought-out list of the ways users will try to interact with your skill.  You don't have to get every possible phrase, but it is important to cover a variety of ways so that the NLU engine can best understand your user's intent.

    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-01-006._TTH_.png)

8.  Given the flexibility and variation of spoken language in the real world, there will often be many different ways to express the same request. Providing these different phrases in sample utterances will help improve voice recognition for the abilities.

    ```
    searchIntent get me stuff happening {date}
    searchIntent get me events for {date}
    searchIntent whats on {date}
    searchIntent whats happening {date}
    searchIntent tell me whats happening {date}
    searchIntent what is happening {date}
    searchIntent what is happening on {date}
    searchIntent what events are happening {date}
    searchIntent what events are happening on {date}

    eventIntent tell me about event {number}
    eventIntent whats event {number}
    eventIntent number {number}
    '`
    

9.  Select **Save**. You should see the interaction model being built (this might a take a minute or two). If you select Next, your changes will be saved and you will go directly to the Configuration screen. After selecting Save, it should now look like this:

    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-01-007._TTH_.png)

Next we will configure the AWS Lambda function that will host the logic for our skill.
 
## Step 2: Creating Your Skill Logic using AWS Lambda

### Installing and Working with the Alexa Skills Kit SDK for Node.js (alexa-sdk)

To make the development of skills easier, we have created the ASK SDK for Node.js. We will be using this module to deploy the sample. The alexa-sdk is available on [github here](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs) and can be deployed as a node package from within your Node.js environment.

1.  node.js needs to be installed installed  and post that install the ASK-SDK (Alexa SDK). Install this in the same directory as  Calendar Reader src/index.js file. Change the directory to the src directory of your skill, and then in the command line, type:
 
    ```
    npm install --save alexa-sdk ical
    ```
    Once this is installed you will need to include the **node_modules** directory with the source code for your skill when you compress the src for uploading to AWS Lambda. 
    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-02-000a._TTH_.png)

### Create an AWS Account
 
 ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-02-001._TTH_.png)
    
 **Note: If you already have an AWS account, Just sign in to your console.**

1.  Open [aws.amazon.com](aws.amazon.com) and then choose **‘Create an AWS Account’**

    1. You will need a Valid Credit Card to set up your account (note the AWS Free Tier will suffice however. You can find out more about the free tier [here](https://aws.amazon.com/free/?sc_ichannel=ha&amp;sc_ipage=signin&amp;sc_iplace=body_link_text&amp;sc_icampaigntype=free_tier&amp;sc_icampaign=ha_en_free_tier_signin_2014_03).)
    2. Part of the sign-up procedure involves receiving a phone call and entering a PIN using the phone keypad.
    
2.  Sign in to the AWS Console

3.  It can sometimes take a couple minutes for new AWS account to go live. You will receive an e-mail when your account is active.

### Create an AWS Lambda Function

AWS Lambda lets you run code without provisioning or managing servers. You pay only for the compute time you consume - there is no charge when your code is not running. With Lambda, you can run code for virtually any type of application or backend service - all with zero administration. Just upload your code and Lambda takes care of everything required to run and scale your code with high availability.

**Note: Lambda Getting Started Guide can be found at(http://docs.aws.amazon.com/lambda/latest/dg/getting-started.html).**

1.  **IMPORTANT**: Select **US East (N. Virginia)** region, or the **EU (Ireland)** region (upper right corner). These are the only regions that currently support Alexa Skill development.

    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%209.png)

2.  Select **Lambda** from AWS Services (under Compute)

    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-02-003._TTH_.png)

3.  Select **“Create a Lambda Function”** to begin the process of defining your Lambda function.
 
    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%2011.png)

4.  Select the **Blank Function** option on the Select Blueprint screen.

    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%2012.png)

5.  Now, you need to configure the event that will trigger your function to be called. As we are building skills with the Alexa Skills Kit, click on the gray dash-lined box and select Alexa Skills Kit from the dropdown menu.

    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%2013.png)

6.  Choose **Next** to continue.

    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%2014.png)

7.  You should now be in the **"Configure Function"** section. Enter the Name, Description, and Runtime for your skill as in the example below.  Your runtime should be "Node.js 4.3."

    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%2015.png)

8.  Select the **‘Code Entry Type’** as **‘Upload Zip File’** and upload the zip file containing the example you created in Step 1. **Note:** This zip file should contain the contents of the src directory, including the node_modules subfolder.

    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%2016.png)

9.  Set your handler and role as follows:

    * Keep Handler as ‘index.handler’
    * Drop down the “Role” menu and select **“Create a custom role”**. (Note: if you have already used Lambda you may already have a ‘lambda_basic_execution’ role created that you can use.) This will launch a new tab in the IAM Management Console.
        

10. You will be asked to set up an Identity and Access Management or “IAM” role if you have not done so. AWS Identity and Access Management (IAM) enables you to securely control access to AWS services and resources for your users. Using IAM, you can create and manage AWS users and groups, and use permissions to allow and deny their access to AWS resources. The IAM role will give your lambda function permission to use other AWS Services at runtime, such as Cloudwatch Logs, the AWS logs collection and storage service. In the Role Summary section, select "Create a new IAM Role" from the IAM Role dropdown menu. The Role Name and policy document will automatically populate.

    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-02-012._TTH_.png)

11. Select **“Allow”** in the lower right corner and you will be returned to your Lambda function.

    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-02-011._TTH_.png)

12. Keep the Advanced settings as default. Select **‘Next’** and review. You should see something like below. Then select **‘Create Function’**:

    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%2018.png)

13. Congratulations, you have created your AWS Lambda function. **Copy** the Amazon Resource Name (ARN) for use in the Configuration section of the Amazon Developer Portal.

    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%2020.png)

## Step 3: Add Your Lambda Function to Your Skill

1.  Navigate back to [developer.amazon.com](http://developer.amazon.com) and select your skill from the list. You can select the skill name or the edit button.
 
    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%2023.PNG)

2.  Select the Configuration section, and make sure to choose the AWS Lambda ARN region that corresponds to your AWS Lambda function's region.  Add the ARN from the Lambda function you created in the AWS Console earlier. Select the **Lambda ARN (Amazon Resource Name)** radio button. Then, select **“No”** for account linking since we will not be connecting to an external account for this tutorial. Paste the ARN you copied earlier into the Endpoint field. Then select **Next**.

    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-03-002._TTH_.png)

3.  You will be asked if you want to "Save Global Changes."  This happens because you are changing values that would apply to every version of your skill (in every language.)  You can click "Yes, Apply" to complete this step.

    ![](https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/tutorials/calendar-reader/calendar-reader-03-003._TTH_.png)
 
4.  You have now completed the initial development of your skill. Now it is time to test.

## Step 4: Testing Your Skill

1.  In the Test area, we are going to enter a sample utterance in the service simulator section and see how Alexa will respond. In this example, we have called the skill ‘Stanford Calendar,’ because we will be retrieving events from Stanford University's public events calendar. This is the ‘Invocation Name’ we set up on the Skill Information line in the “Skill Information” section.

    * In the Service Simulator, type **‘open Stanford Calendar’** and click the **“Ask Stanford Calendar”** button.

    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%2021.png)

2.  You should see the formatted JSON request from the Alexa service and the response coming back from your Lambda function. Verify that you get a correct Lambda response.
  
    ![](https://github.com/kamresh/alexa-calendar-skill/blob/master/images/Alexa%2022.png)
  
### Skills / Lambda Troubleshooting (getting an invalid response)?
 * Do you have the right ARN copied from your Lambda function into your Developer Portal / Skill?
 * Are you calling the right invocation name?
 * Are you saying launch, start or open (followed by your invocation name)?
 * Are you sure you have no other skills in your accounts with the same invocation name?
 
 References:
 - Standford Calendar Skill (https://github.com/alexa/skill-sample-nodejs-calendar-reader).
