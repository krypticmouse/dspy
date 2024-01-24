---
sidebar_position: 2
---

# Minimal Working Example

In this blog post, we walk you through a minimal working example using the DSPy library. 

We make use of the GSM8K dataset and the OpenAI GPT-3.5-turbo model to simulate prompting tasks within DSPy.

## Setup

Before we delve into the example, let's ensure our environment is properly configured. We'll start by importing the necessary modules and configuring our language model:

```python
import dspy
from dspy.evaluate import Evaluate
from dspy.datasets.gsm8k import GSM8K, gsm8k_metric
from dspy.teleprompt import BootstrapFewShotWithRandomSearch

gms8k = GSM8K()
turbo = dspy.OpenAI(model='gpt-3.5-turbo-instruct', max_tokens=250)

trainset, devset = gms8k.train, gms8k.dev

dspy.settings.configure(lm=turbo)
```

## Define the Module

With our environment set up, let's define a custom module that utilizes the `ChainOfThought` module to perform step-by-step logical reasoning to generate answers to questions:

```python
class CoT(dspy.Module):
    def __init__(self):
        super().__init__()
        self.prog = dspy.ChainOfThought("question -> answer")
    
    def forward(self, question):
        return self.prog(question=question)
```

## Compile and Evaluate the Model

With our custom module in place, let's move on to optimizing the program by compiling the module using the `BootstrapFewShotWithRandomSearch` teleprompter and evaluating its performance on the dev dataset:

```python
evaluate = Evaluate(devset=devset[:], metric=gsm8k_metric, num_threads=4, display_progress=True, display_table=0)

config = dict(max_bootstrapped_demos=8, max_labeled_demos=8, num_candidate_programs=10, num_threads=NUM_THREADS)
teleprompter = BootstrapFewShotWithRandomSearch(metric=gsm8k_metric, **config)
cot_bs = teleprompter.compile(CoT(), trainset=trainset, valset=devset)

evaluate(cot_bs, devset=devset[:])
```

## Inspect the Model's History

For a deeper understanding of the model's interactions, we can review the most recent interactions through inspecting the model's history:

```python
turbo.inspect_history(n=1)
```

And there you have it! You've successfully created a working example using the DSPy library. 

This example showcases how to set up your environment, define a custom module, compile a model, and rigorously evaluate its performance using the provided dataset and teleprompter configurations. 

Feel free to adapt and expand upon this example to suit your specific use case while exploring the extensive capabilities of DSPy.
