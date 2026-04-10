# Evaluating LLM Bias in Criminal Sentencing

## CS 2880: AI for Social Impact Project Proposal

 Team: MB Samuel, Brit Biddle, Lorraine Bichara

As AI systems are increasingly explored for legal applications - from triaging cases to focus human judgment on the most complex ones, to assisting with sentencing recommendations and legal research - understanding whether these systems encode societal biases becomes critical. In federal criminal sentencing, decades of research have documented disparities along racial and demographic lines, even after controlling for legally relevant factors like offense severity and criminal history. If large language models (LLMs) are trained on data that reflects these disparities, they may reproduce or even amplify them when applied to legal tasks. This project investigates whether LLMs exhibit racial bias when asked to predict federal criminal sentences, and how that bias compares to documented disparities in actual sentencing outcomes.

Using the JUSTFAIR dataset, a public database of nearly 600,000 federal criminal sentencing records linked with defendant demographics, offense characteristics, and judge information, we design an ablation study to isolate the effect of demographic information on LLM predictions. We present an LLM with structured case facts (offense type, criminal history, sentencing guideline range) and ask it to predict the sentence, systematically varying whether the prompt includes the defendant's race, a racially-associated name, or no demographic information at all. By comparing predictions across these conditions, we can measure whether and how much demographic cues shift the LLM's output. We then benchmark this LLM bias against actual sentencing disparities in the data, asking: does the model amplify real-world bias, reproduce it, or reduce it? Our findings have direct implications for the responsible deployment of AI in legal contexts, particularly as courts and legal institutions consider AI-assisted tools for case triage and decision support.

We will test this across a wide range of LLMs (OpenAI, Anthropic, and 1-2 open source models, e.g. Grok) to compare results.
