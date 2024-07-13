---
layout: post
title: "Chickadee"
date: 2024-07-12
categories: [Python, ChatGPT, OpenAI]
---

So far, it hasn't been my habit to save questions that I ask ChatGPT of Claude.
I've seen recommendeations for doing this, to create reusable prompts. Well,
yesterday I downloaded my ChatGPT history to see what data I could get, and
found that there was `conversations.json` file with all of my chat history.

I wrote up a Python script, with the help of Claude Sonnet 3.5 and ChatGPT, that
extracts the first user message from each conversation. I use the OpenAI Python library
with gpt-4o to have it analyze all of these questions and generate reusable prompts.

This is the list of 10 prompts that I got back, each contains the prompt and the reasoning that gpt-4o gave for it:

Prompt 1:  
Content: I am facing an issue with [specific technology or problem]. Here are the details: [provide error messages, logs, or specific scenarios]. Can you help me diagnose and solve this problem? If possible, provide potential causes and solutions.  
Reasoning: This prompt consolidates multiple prompts focused on technical troubleshooting and debugging. Including specifics like error messages and logs allows for more accurate assistance and ensures that the response is both detailed and helpful.

Prompt 2:  
Content: Can you review the following code and provide suggestions for optimization, debugging, and best practices: ```<your code here>```  
Reasoning: This combined prompt addresses the need for code review, optimization, and debugging, which were recurring themes. By making the prompt template-driven, it can be reused for various code-related queries, ensuring comprehensive feedback.

Prompt 3:  
Content: Can you provide a detailed guide on how to accomplish the following technical task: ```<task description here>```  
Reasoning: This prompt generalizes the specific task-related queries into a single template, allowing it to be used for a wide range of technical implementations and programming tasks, ensuring the response is practical and instructional.

Prompt 4:  
Content: Explain the following technical concept or term in simple terms, including practical applications if possible: [Insert term or concept]  
Reasoning: This prompt addresses the frequent need for explanations of technical concepts and terms. It is broad enough to cover various subjects, encouraging thorough and accessible explanations.

Prompt 5:  
Content: I need help crafting a professional document (e.g., resume, cover letter, performance review) based on the following information: [provide details]. Can you optimize it for effectiveness and readability?  
Reasoning: This prompt combines various prompts related to resume writing and document optimization. It ensures that users can get tailored, professional content that enhances their career materials.

Prompt 6:  
Content: Can you provide best practices and a step-by-step guide for setting up a project using [specific technology/framework]? Include any essential configurations and tips for optimization.  
Reasoning: This prompt consolidates the project setup and management inquiries into a single version, making it adaptable to various technologies and frameworks, which addresses the user's need for comprehensive startup guidance.

Prompt 7:  
Content: How can I integrate [specific tool/technology] with my Python project? Include examples and potential use cases.  
Reasoning: This combined prompt addresses the integration of tools with Python, a common topic across multiple queries. It provides flexibility to input any tool name and receive tailored suggestions.

Prompt 8:  
Content: Please review the following content for spelling errors, clarity, and overall effectiveness: ```<Insert content here>```. Can you provide feedback and suggest improvements?  
Reasoning: This prompt merges several content review-related prompts into one versatile template, focusing on spelling, clarity, and effectiveness.

Prompt 9:  
Content: Can you provide a learning path and high-level overview for [New Technology/Concept]? Include key resources and steps for mastering it.  
Reasoning: This prompt is designed to help users tackling new technologies, ensuring they receive structured learning paths and key resources for gaining proficiency.

Prompt 10:  
Content: Act as a technical assistant and provide detailed help with the following technical problem or setup: ```[insert technical problem or setup here]```. Ensure your response is informative and includes relevant steps, configurations, or considerations.  
Reasoning: This general technical assistance prompt is tailored to provide comprehensive help across various technical problems, ensuring detailed and informative responses.

If you're curious and want to run it yourself, here's my Github repo:

https://github.com/JoshuaOliphant/chickadee
