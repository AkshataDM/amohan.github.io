---
title: "[AI-Evals] Evaluating LLM applications"
description: "Sharing my thoughts formed by evaluating and building AI applications"
publishDate: "15 September 2025"
tags: ["AI", "AI-Evals"]
---

If you've built an AI application, you've probably experienced that moment of uncertainty: "Is this actually working?" So, you test a few prompts, things look good, but how do you know if its actually ready for production? 

Based on insights from Hamel Husain and Shreya Shankar's popular AI Evals course and my own experience building evaluations for the AI apps I built at my day job, I wrote up this guide that will show you how to build evaluation systems for your AI application

## Evals Are A Necessity

While building an AI application, I spend at least 40-50% of my time on manual error analysis and evaluation. I believe that manually reading through your AI output traces might sound a bit excessive, but it's helped me understand my data better. It's important for me to know if my latest prompt  broke something, if the model is hallucinating on edge cases, or if I misunderstood the user requirements.

Going back to what I learned on the evals course, they define three "gulfs" in AI development: 
- Gulf of Comprehension: Understanding the data (go through the inputs and those output traces!)
- Gulf of Specification: Configure the prompt carefully with the desired output. Few shot prompting, structured outputs, instruction prompting are some options.
- Gulf of Generalization: Make sure the LLM does well beyond the data it was trained on by identifying the right technqiue - context engineering, RAG, breaking down instructions into multiple steps, to make sure the LLM generalizes well enough. 

## Start with Error Analysis, Not Metrics

When I first built AI apps, my mistake was jumping right into the metrics. While metrics like relevancy scores, hallucination scores are always useful, it took me a while to identify the root cause. I found that manually reviewing real user traces helped a lot more to uncover upstream issues and patterns. So I now block 30 minutes on my calendar, pull 20-50 real user interactions, and write down what went right and what went wrong.

This manual review accomplishes two things:
1. It grounds me in actual user problems rather than imagined ones
2. It helps identify failure modes that generic metrics would miss


## Building an Eval Pipeline Without Making It a Bottleneck

I've found that integrating an eval pipeline in the development workflow is a lot easier than running evals after deployment. 

### 1. Define Pass/Fail Criteria That Matter

Generic metrics like BERTScore, ROUGE, and cosine similarity are not useful for evaluating LLM outputs in most AI applications. Instead, I've found that  binary pass/fail evals using LLM-as-judge or code-based assertions are better at telling me what is going on in my application. 
Let's take an example of building an AI powered real estate CRM assistant. Here's what might be useful to measure 

- ✅ Do measure: "Does the system suggest only available showings?" (code assertion)
- ✅ Do measure: "Does the response avoid confusing client personas?" (LLM-as-judge)
- ❌ Don't measure: Generic text similarity scores

### 2. Use LLM-as-Judge Thoughtfully

LLM-as-Judge approaches can effectively evaluate outputs when properly validated against human judgments. The trick is alignment - I have found that going through traces manually and labeling it works best for the judge LLM.

### 3. Scale Testing with Synthetic Data

Full disclaimer - I am yet to try this approach! 
Shreya and Hamel talk about synthetic data generation quite a bit in the course. The recommendation is that when you don't have enough real examples, synthetic data fills the gaps. Synthetic data scales fast (you can easily generate thousands of test cases), fills gaps by adding missing scenarios and edge cases, and allows controlled testing to see how AI handles specific challenges. 

The process typically involves two stages: context generation (selecting relevant chunks from your knowledge base) and input generation (creating questions/queries from those contexts). This reverses standard retrieval - instead of finding contexts from inputs, you create inputs from predefined contexts.

## Key Metrics & Takeaways: 

While every application needs custom metrics, here are the essential categories to consider:

### For RAG Systems
RAG metrics measure either the retriever or generator in isolation. Retriever metrics include contextual recall, precision, and relevancy for evaluating things like top-K values and embedding models. Generator metrics include faithfulness and answer relevancy for evaluating the LLM and prompt template.

### Core Universal Metrics
- **Hallucination**: Determines whether an LLM output contains fake or made-up information
- **Answer Relevancy**: Measures how well the response addresses the input in an informative and concise manner
- **Task Completion**: Whether the system achieves its intended goal

### The ROUGE Score Reality Check
Research shows ROUGE exhibits alarmingly low precision for identifying actual factual errors. These overlap based metrics systematically overestimate hallucination detection performance in QA, leading to illusory progress. Traditional NLP metrics weren't necessarily designed for generative AI. It might provide useful insights to begin with but that's about it.

## Categorizing Failure Modes: Manual and Automated Approaches

Understanding why your system fails is as important as knowing that it fails. Start manually:

1. **Manual Categorization**: Review failing cases and group them into patterns (e.g., "fails on multi-step reasoning," "misunderstands temporal queries")
2. **Automated Categorization**: Once you identify patterns, use LLMs to categorize new failures automatically


## Making Evals Part of Your Culture

Well crafted eval prompts effectively become living product requirements documents that continuously test your AI in real time. Iterate on the application as you get feedback signals from the evaluation pipeline. I've found this to be a far better choice than building an entire application with all the embedding AI features only to course correct or change requirements after evaluations or user testing. Start small, iterate quickly. 


## The TLDR:

Start small:
1. Manually review 10-20 real interactions today
2. Identify your top 3 failure modes
3. Write one simple pass/fail eval for each
4. Generate 100 synthetic test cases
5. Run these evals before every deployment

Remember: Nothing beats truly understanding your data! As a data scientist, digging into the data, looking for patterns is in my very DNA and training so it comes naturally to me. However, if you are not from a data background, spending time looking through the data might seem futile but would save you loads of time in the long run. 
---

*Ready to dive deeper? Check out [Hamel and Shreya's AI Evals course](https://maven.com/parlance-labs/evals) for hands-on training in evaluation-driven development.*
