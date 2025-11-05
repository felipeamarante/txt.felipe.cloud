+++
title = "Multimodal AI: When Text-Only LLMs Aren't Enough"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-04-15"
categories = [
    "AI",
]
menu = "main"
+++

### "Can you tell me what's wrong with this circuit diagram?"

![circuit diagram](/images/circuit.gif)

I sent this message to ChatGPT along with a photo of a circuit board I was troubleshooting. The response? "I'm sorry, but I can't see any images you've shared."

Right. Text-only AI has its limits.

But the latest generation of AI models, multimodal models, can see images, hear sounds, and even understand video. They're changing what's possible with AI, and they're already more accessible than you might think.

### What exactly is multimodal AI?

Traditional LLMs like early versions of GPT only understand text. Multimodal models can process and generate multiple types of data:

```
Text-only LLM:
Input: Text → Model → Output: Text

Multimodal AI:
Input: Text + Images + Audio → Model → Output: Text + Images + Audio
```

### The four capabilities that make multimodal AI game-changing

After experimenting with models like GPT-4V, Claude 3, and Gemini, I've found these capabilities most useful:

1. **Visual understanding**: Analyzing charts, diagrams, screenshots, and photos
   ```
   "What's wrong with this error message?"
   "Can you explain this graph trend?"
   "What species of plant is this?"
   ```

2. **Document analysis**: Extracting information from PDFs, receipts, and forms
   ```
   "Summarize the key points from this research paper"
   "Extract the line items from this invoice"
   ```

3. **Visual creation**: Generating or editing images based on text descriptions
   ```
   "Create a flowchart showing how JWT authentication works"
   "Edit this logo to use blue instead of red"
   ```

4. **Cross-modal reasoning**: Connecting concepts across different formats
   ```
   "Does this code match the architecture diagram?"
   "Is this UI implementation consistent with the wireframe?"
   ```

### Real-world example: Debugging a React app in 2 minutes instead of 20

Last week, I was struggling with a React component that wasn't rendering properly. After 15 minutes of fruitless debugging, I took a screenshot of both my code and the broken UI, then asked GPT-4V:

"What's causing this rendering issue? Here's my component code and the current output."

In seconds, it spotted the problem: I was destructuring props incorrectly, causing a key prop to be undefined. What would have been another 20 minutes of debugging was solved in 2 minutes.

### Getting started with multimodal AI today

You don't need special hardware or complex setups. Here's how to use GPT-4 with Vision through the API:

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4-turbo",
    "messages": [
      {
        "role": "user",
        "content": [
          { "type": "text", "text": "Describe this image" },
          { "type": "image_url", "image_url": { "url": "https://example.com/image.png" } }
        ]
      }
    ]
  }'
```

This simple command lets you send both text and images to the model and get back detailed analysis.

### Pro tip: Multimodal prompting techniques

The way you prompt multimodal models significantly affects results:

1. **Be specific about what to look for**:
   ```
   Bad: "What do you see in this dashboard screenshot?"
   Good: "In this AWS CloudWatch dashboard screenshot, identify any metrics that show abnormal spikes in the last 24 hours"
   ```

2. **Provide context for images**:
   ```
   Bad: [Image only]
   Good: "This is a network diagram for our AWS infrastructure. Are there any security issues?"
   ```

3. **Request structured output**:
   ```
   "Analyze this sales chart and provide: 
   1. The overall trend
   2. Any anomalies
   3. Recommendations based on the data"
   ```

### The hidden benefit: Accessibility improvements

One unexpected benefit I've found is using multimodal AI to improve accessibility:

```
# Generate alt text for images
"Create detailed alt text for this product image that would be helpful for visually impaired users"

# Convert visual information to text
"Describe this architectural diagram in text form that could be read by a screen reader"
```

### The future is multimodal

As these models continue to improve, we'll see:

- Video understanding becoming more sophisticated
- Real-time analysis of visual data streams
- More specialized models for domains like medicine and engineering

The days of text-only AI are numbered. The most powerful applications will combine multiple modalities, just like humans do.

Whether you're debugging code, analyzing documents, or creating content, multimodal AI offers capabilities that text-only models simply can't match. And the best part? You can start using these tools today.
