# Conversational Telugu-to-English Translator Alexa Skill

![Alexa Skill](https://img.shields.io/badge/Alexa-Skill-blue?logo=amazon-alexa)
![AWS Lambda](https://img.shields.io/badge/AWS-Lambda-orange?logo=aws-lambda)
![Node.js](https://img.shields.io/badge/Node.js-18.x-green?logo=node.js)
![Language](https://img.shields.io/badge/Language-Telugu%20%E2%86%92%20English-brightgreen)

This repository documents the end-to-end development of a custom Amazon Alexa skill that provides real-time, conversational translation from spoken Telugu into English. The skill runs on a serverless backend using AWS Lambda and is designed for practical, everyday use on any Amazon Echo device.

This project is a demonstration of building a complete voice application, from initial concept and VUI (Voice User Interface) design to backend development, systematic debugging, and final deployment.

## Table of Contents

- [Project Overview](#project-overview)
- [Core Features](#core-features)
- [Live Interaction Demo](#live-interaction-demo)
- [How It Works: The AI Pipeline](#how-it-works-the-ai-pipeline)
- [Technical Architecture](#technical-architecture)
- [Final Skill Configuration](#final-skill-configuration)
- [The Development Journey: A Retrospective](#the-development-journey-a-retrospective)
- [Step-by-Step Deployment Guide](#step-by-step-deployment-guide)
- [Complete Project Code](#complete-project-code)
- [License](#license)

## Project Overview

The goal of this project was to create a hands-free tool to help my family with language translation. The final product is an Alexa skill that can be invoked with a simple voice command, allowing for a seamless conversational flow. A user can open the skill and translate multiple sentences without needing to re-invoke it each time.

## Core Features

- **Voice-Activated Translation:** Leverages Alexa's ASR and NLU to understand spoken commands.
- **Conversational Sessions:** Uses reprompts to keep the session active for a natural, back-and-forth dialogue.
- **Serverless Backend:** The entire logic is hosted on AWS Lambda, ensuring high availability and zero server management.
- **Real-World Usability:** Deployed and available for use on any Amazon Echo device linked to the developer's account.
- **Low Latency:** Optimized for quick responses, with a configured Lambda timeout to handle network API calls.

## Live Interaction Demo

> **User:** “Alexa, open telugu helper.”
>
> **Alexa:** “Welcome to the English translator. What would you like to translate?”
>
> **User:** “Nee peru enti.”
>
> **Alexa:** “What is your name.”
>
> **User:** “Naku chala aakali ga undi.”
>
> **Alexa:** “I am very hungry.”
>
> **User:** "Stop."
>
> **Alexa:** "Goodbye!"

## How It Works: The AI Pipeline

This skill is a practical application of how different specialized AI systems work in concert:

1.  **Agentic AI (Alexa Platform):** The overarching Alexa system acts as the agent. It perceives the user's voice, determines which skill to activate (using the `telugu helper` invocation name), and orchestrates the workflow.
2.  **Deep Learning (ASR & NLU):** Amazon's pre-trained Deep Learning models are used for:
    - **Automatic Speech Recognition (ASR):** Converting the audio of the user's voice into text.
    - **Natural Language Understanding (NLU):** Parsing the text to identify the user's `TranslateIntent` and extract the `teluguSentence` slot value.
3.  **Generative AI (Translation & Speech Synthesis):**
    - **Neural Machine Translation (NMT):** The core translation is performed by the Google Translate API, which uses a generative model to create a new, grammatically correct sentence in English.
    - **Text-to-Speech (TTS):** The final English text is converted back into natural-sounding audio by another generative model, which Alexa speaks.

## Technical Architecture

The architecture is fully serverless and event-driven:

- **Frontend:** Alexa Developer Console (VUI for Intents, Slots, and Utterances).
- **Backend:** AWS Lambda function running Node.js 18.x.
- **Permissions:** AWS IAM Role (`AWSLambdaBasicExecutionRole`) providing the function with logging permissions.
- **External API:** Google Translate API accessed via the `@iamtraction/google-translate` npm package.
- **Monitoring:** Amazon CloudWatch for real-time logging and debugging.

## Final Skill Configuration

- **Skill Invocation Name:** `telugu helper`
- **Intent:** `TranslateIntent`
- **Slots:**
  - `teluguSentence` (Type: `AMAZON.SearchQuery`)
- **Sample Utterances:**
  - `translate the phrase {teluguSentence}`
  - `ask for the translation of {teluguSentence}`

## The Development Journey: A Retrospective

Developing this skill was an iterative process of building, testing, and debugging. Key challenges and solutions included:

1.  **Dependency Errors:** Initial failures were due to `node_modules` not being included in the Lambda deployment package. Resolved by learning to correctly create a `.zip` archive with all project contents.
2.  **Platform Incompatibility:** The first translation library failed with a `.node` file error, revealing an incompatibility between the macOS compile environment and the Amazon Linux Lambda runtime. Solved by switching to a pure-JavaScript library (`@iamtraction/google-translate`).
3.  **Request Routing Failures:** `Unable to find a suitable request handler` errors were traced back to a mismatch between the intent name in the Alexa Console (`HelloWorldIntent`) and the code (`TranslateIntent`). Deleting the old intent and rebuilding the model fixed this.
4.  **Lambda Timeouts:** The skill would time out on the first request. I resolved this by increasing the default Lambda timeout from 3 seconds to 10 seconds, allowing sufficient time for the API call.
5.  **VUI Refinement:** The user experience was improved by removing robotic text from the response and, most importantly, by adding a `.reprompt()` to the response builder to enable continuous conversational sessions.

## Step-by-Step Deployment Guide

To deploy this skill yourself, follow these high-level steps:

1.  **Alexa Console:** Create a new skill, configure the invocation name, intent, slot, and utterances as detailed above. Build the model and copy the Skill ID.
2.  **AWS IAM:** Create an IAM Role for your Lambda function with `AWSLambdaBasicExecutionRole` permissions.
3.  **AWS Lambda:** Create a Node.js 18.x Lambda function. Add an "Alexa Skills Kit" trigger and paste in your Skill ID. In the **Configuration** tab, increase the timeout to **10 seconds**.
4.  **Local Setup:**
    - Clone this repository.
    - Run `npm install` to get the dependencies.
    - Create a `.zip` file containing `index.js`, `package.json`, and the `node_modules` folder.
5.  **Deploy:** Upload the `.zip` file to your Lambda function's code source.

## Complete Project Code

<details>
<summary><strong>Click to view `index.js`</strong></summary>

```javascript
const Alexa = require('ask-sdk-core');
const translate = require('@iamtraction/google-translate');

// Welcomes the user and opens a session
const LaunchRequestHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'LaunchRequest';
    },
    handle(handlerInput) {
        const speakOutput = 'Welcome to the English translator. What would you like to translate?';
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt(speakOutput) // Keep the session open
            .getResponse();
    }
};

// Handles the core translation logic
const TranslateIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'TranslateIntent';
    },
    async handle(handlerInput) {
        const textToTranslate = Alexa.getSlotValue(handlerInput.requestEnvelope, 'teluguSentence');
        const targetLanguage = 'en'; // Hardcoded to English
        const repromptText = ' What else would you like to translate?';
        let speakOutput = '';

        if (textToTranslate) {
            try {
                const result = await translate(textToTranslate, { to: targetLanguage });
                speakOutput = result.text;
            } catch (error) {
                console.error(error);
                speakOutput = 'Sorry, I had trouble with that translation.';
            }
        } else {
            speakOutput = 'I did not catch that. Please tell me what to translate.';
        }

        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt(repromptText) // Keep the session open for the next command
            .getResponse();
    }
};

// Handles built-in "Help" requests
const HelpIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.HelpIntent';
    },
    handle(handlerInput) {
        const speakOutput = 'You can ask me to translate a phrase, like "translate the phrase nee peru enti".';
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt(speakOutput)
            .getResponse();
    }
};

// Handles "Cancel" and "Stop" requests
const CancelAndStopIntentHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
            && (Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.CancelIntent'
                || Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.StopIntent');
    },
    handle(handlerInput) {
        const speakOutput = 'Goodbye!';
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .getResponse(); // No reprompt, so the session closes
    }
};

// Handles session end events
const SessionEndedRequestHandler = {
    canHandle(handlerInput) {
        return Alexa.getRequestType(handlerInput.requestEnvelope) === 'SessionEndedRequest';
    },
    handle(handlerInput) {
        console.log(`Session ended with reason: ${handlerInput.requestEnvelope.request.reason}`);
        return handlerInput.responseBuilder.getResponse();
    }
};

// A generic error handler for unexpected issues
const ErrorHandler = {
    canHandle() {
        return true;
    },
    handle(handlerInput, error) {
        console.log(`Error handled: ${error.stack}`);
        const speakOutput = 'Sorry, I had trouble doing what you asked. Please try again.';
        return handlerInput.responseBuilder
            .speak(speakOutput)
            .reprompt(speakOutput)
            .getResponse();
    }
};

// Register all handlers
exports.handler = Alexa.SkillBuilders.custom()
    .addRequestHandlers(
        LaunchRequestHandler,
        TranslateIntentHandler,
        HelpIntentHandler,
        CancelAndStopIntentHandler,
        SessionEndedRequestHandler)
    .addErrorHandlers(
        ErrorHandler)
    .lambda();

// Generated JSON
{
  "name": "alexa-telugu-translator",
  "version": "1.0.0",
  "description": "Alexa skill to translate Telugu to English.",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Your Name",
  "license": "ISC",
  "dependencies": {
    "ask-sdk-core": "^2.12.1",
    "@iamtraction/google-translate": "^1.1.2"
  }
}
