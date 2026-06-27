+++
title = "How LLMs Actually Work: Explaining AI to My Mom Over Video Call"
author = "Felipe A."
draft = false
tags = [
    "Felipe A.",
]
date = "2025-03-30"
categories = [
    "AI",
]
menu = "main"
+++

### "So how does this ChatGPT thing actually work?"

My mom asked me this during our weekly video call. Living thousands of kilometers apart means these calls are our main connection, and they often turn into impromptu tech support sessions. She'd been using ChatGPT to help write emails and was genuinely curious about what was happening behind the scenes.

As I started explaining neural networks and transformers, I could see her eyes glazing over through the pixelated FaceTime screen. So I tried again with analogies she could actually follow.

### The library analogy: how LLMs learn

"Mom, imagine someone who has read every book in the world's largest library: billions of books, articles, websites, and conversations.

This person hasn't memorized every word, but they've developed a solid intuition for how language works and which words tend to follow others in different contexts.

That's basically what an LLM is. It's read most of the text on the internet and learned patterns about how language works."

### The word prediction game

To make it more concrete, I played a simple game with her over the call:

```
Me: "Complete this sentence: The cat sat on the..."
Mom: "Mat!"
Me: "How about: I need to go to the store to buy some..."
Mom: "Milk? Bread? Groceries?"
```

"That's exactly what ChatGPT is doing," I explained. "It's predicting what word should come next, based on all the text it's seen before. But it's doing this prediction billions of times in a row, creating entire paragraphs and essays one word at a time."

### The recipe book vs. the chef

My mom loves cooking, so this analogy clicked even through the distance:

"A traditional computer program is like following a recipe exactly. If the recipe says 'add 500 grams of flour,' that's what it does. No improvising.

But an LLM is more like an experienced chef who's studied thousands of recipes. They don't follow exact instructions. They've built up an intuition about cooking. If you ask them to make a pasta dish with whatever's in your fridge, they can wing it, even if they've never made that exact dish before."

### How it actually works: the 5-minute version

When she asked for a bit more detail, I shared my screen and drew a simple diagram as I broke it down into five steps:

1. **Training data**: Feed the AI massive amounts of text: books, articles, websites.
2. **Tokenization**: The AI breaks all that text into small pieces called "tokens," which are words or parts of words.
3. **Pattern learning**: Using some fairly involved math (I spared her the details of matrix multiplication and backpropagation), the AI learns which words tend to follow others in different contexts.
4. **Weights and connections**: The AI builds a huge web of connections between words and concepts, with some connections stronger than others.
5. **Prediction**: When you ask it a question, it uses all those connections to figure out what words should come next in its response.

### Does it actually understand?

My mom then asked the question everyone eventually gets to, through her slightly laggy FaceTime connection: "But does it actually understand what it's saying?"

"That's the big debate," I told her. "It's like a very advanced parrot that's gotten so good at mimicking human language patterns that it can fool you. It doesn't 'understand' in the way humans do. No experiences, no emotions, no consciousness.

But it's so good at predicting language patterns that it can fake understanding very convincingly. It's like how a mirror reflects your image without actually 'seeing' you."

### Why they sometimes make stuff up

This led to her next question: "Why does it sometimes make up facts? It told me a recipe for banana bread that sounded wrong."

"Remember how I said it's predicting the next word? Well, sometimes the most likely next word leads to something that sounds plausible but isn't true.

It's like if I asked you to continue a story about Abraham Lincoln. You'd probably say he lived in the White House, which is true, but you might also add details that sound right but aren't historically accurate."

### Why bigger models are smarter

To explain why some AI models are better than others, I used a comparison she could visualize:

"The bigger models have read more books and have more connections between concepts. GPT-4 has hundreds of billions of these connections, while smaller models might only have a few billion.

It's like comparing someone who's read 10,000 books to someone who's read 100. The first person will generally give you more nuanced and accurate information."

### LLMs don't learn from your conversations

My mom assumed AI models were constantly learning from our conversations, so I had to correct her:

"Most LLMs like ChatGPT are frozen after their initial training. They don't learn from your conversations. It's like they read billions of books, took a final exam, and now they're just using what they studied to answer questions.

Some newer systems have memory or can search the web for updated information, but the core model itself isn't learning from you."

### Why any of this matters

As our call was wrapping up (her cat demanding attention in the background), she asked why she should care about any of this.

"Because if you know it's just predicting words, you stop treating everything it says as fact," I said. "It's a powerful and useful tool, but it's still just a very advanced calculator for language. Not a doctor, not a lawyer, not a researcher."

### Mom's verdict

After my explanation, my mom nodded and said, "So it's like having a very well-read research assistant who sometimes makes things up, doesn't have real experiences, and is really just playing an incredibly sophisticated word prediction game?"

"That's... actually a perfect summary," I replied.

And then she asked me to walk her through setting up ChatGPT via screen sharing, because "even if it's just playing word games, it still writes better emails than I do."
