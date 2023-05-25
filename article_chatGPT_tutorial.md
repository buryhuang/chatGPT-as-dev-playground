**Discover how chatGPT can boost your daily programming tasks and hackathons.**

As a seasoned engineering builder, I've witnessed the transformative power of chatGPT in AI hackathons and everyday programming. Mastering this technology can greatly enhance an engineer's efficiency, making it a crucial skill to acquire.

Today, I'll share my insights on leveraging chatGPT to excel in AI hackathons and daily tasks. Let's dive in!

### Harnessing chatGPT for AI Hackathons
Utilize chatGPT to generate skeleton code for your ideas within minutes. For instance, consider a project with these components:
- Frontend: create react app
- Backend: Create REST API service in Python
- Blockchain: Create a smart contract

In this post, we'll focus on creating the frontend webapp.

#### Tip 1: Start Simple
Familiarize yourself with what Language Models (LLMs) can and cannot do before exploring alternatives. Avoid starting with open-source projects like HuggingFace or AutoGPT; focus on using AI effectively instead of reinventing the wheel.

#### Tip 2: Use Playground over Chat Interface
The playground interface allows you to interact with the model without coding, streamlining your experience.

### Real Example:
Let's build a frontend for this hackathon idea:
*A shared chatGPT portal verified with an email address on blockchain.*
This project aims to create a shared chatbot interface powered by OpenAI's GPT-3, featuring user identity verification via blockchain-stored email addresses for enhanced security and privacy.

#### Setup Steps
Visit https://platform.openai.com/playground, select "Chat" under "mode," choose Model "gpt-3.5-turbo" (access GPT4 through FutureBuilders events).

##### Creating a Frontend
Input the following in the "System" bar:
```
You are a Senior Frontend Developer, SME of React App.
- Read the given specification
- Output an React App, separated into small code pieces in multiple files according to the given functional spec and existing code
- The react app shall use tailwinds as styling
```

Enter this in the main message panel:
```
Here is the react app page spec:
- A login page to enter email
- A modal dialog will be popped, showing progress, telling users to wait for email verification on-chain
- A chatGPT chatbot interface after login
- The chat interface has a side bar to input “System” message same as chatGPT playground
- The chat interface has a messages box going back and forth between user and the app
```

Watch chatGPT work its magic! Create corresponding React files at https://playcode.io/react and run your working app in under 3 minutes. Spend your weekend refining your ideas!

Do you think chatGPT should be part of an engineer's job description? Share your thoughts!

Join FutureBuilders' AI hackathon this June/July. Follow or comment for FREE GPT4 access during the event.
