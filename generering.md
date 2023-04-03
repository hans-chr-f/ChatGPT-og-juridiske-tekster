# Demonstrasjon av generering av tekst.

```
import openai
import math
from tabulate import tabulate

openai.api_key = "egen_openai_api_key"

def predict_top_n_words_with_probabilities(prompt, n=5):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=prompt,
        max_tokens=14,
        n=n,
        stop=None,
        temperature=0.45,
        logprobs=n,
    )

    completions = []
    for choice in response.choices:
        token = choice.text.strip()
        token_logprob = choice.logprobs['token_logprobs'][0]
        token_probability = math.exp(token_logprob)
        completions.append((token, token_probability))

    completions.sort(key=lambda x: x[1], reverse=True)
    return completions

prompt = ""

top_5_words_with_probabilities = predict_top_n_words_with_probabilities(prompt, n=5)

headers = ["Word", "Probability"]
table = tabulate(top_5_words_with_probabilities, headers=headers, tablefmt="grid", numalign="right", floatfmt=".5f")
print("Top 5 probable continuations with probabilities (sorted):\n", table)

