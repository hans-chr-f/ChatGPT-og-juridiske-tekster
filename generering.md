# Modellbasert generering av tekst.

```
import openai
import math
from tabulate import tabulate

openai.api_key = "egen_openai_api_key"

def predict_top_n_continuations_with_probabilities(prompt, n=5, max_tokens=6, temperature=0.5):
    response = openai.Completion.create(
        engine="text-davinci-003",
        prompt=prompt,
        max_tokens=max_tokens,
        n=n,
        stop=None,
        temperature=temperature,
        logprobs=n,
    )

    completions = []
    for choice in response.choices:
        continuation = choice.text.strip()
        total_logprob = sum(choice.logprobs['token_logprobs'])
        probability = math.exp(total_logprob)
        completions.append((continuation, probability))

    completions.sort(key=lambda x: x[1], reverse=True)
    return completions

prompt = "Hovedprinsippet i norsk avtalerett er"
top_5_continuations_with_probabilities = predict_top_n_continuations_with_probabilities(prompt, n=5)
print(prompt)
headers = ["Fortsettelse", "Sannsynlighet"]
table = tabulate(top_5_continuations_with_probabilities, headers=headers, tablefmt="github", numalign="right", floatfmt=".5f")
print("Top 5 probable continuations with probabilities (sorted):\n", table)

```

Hvis vi ber modellen om å gi fem sannsynlige forsettelser for "Hovedprinsippet i norsk avtalerett er" får vi:

| Fortsettelse          |   Sannsynlighet |
|-----------------------|-----------------|
| at kontrakter sk      |         0.00906 |
| at partene er fritt   |         0.00536 |
| at partene selv sk    |         0.00488 |
| at det skal legg      |         0.00467 |
| at partene har frihet |         0.00352 |

Så kan vi legge inn fortsettelsen "at kontrakter". Det ber vi modellen om å gi fem sannsynlige fortsetteler på "Hovedprinsippet i norsk avtalerett er at kontrakter".

| Fortsettelse      |   Sannsynlighet |
|-------------------|-----------------|
| skal oppfylles    |         0.04556 |
| skal tolkes obj   |         0.03115 |
| skal tolkes inn   |         0.00517 |
| skal tolkes d     |         0.00266 |
| er bindene. Dette |         0.00125 |

Tre av de videre alternativene fortsetter med "skal tolkes". La oss legge til dette og be modellen om å fortsettte følgende tekst "Hovedprinsippet i norsk avtalerett er at kontrakter skal tolkes"

| Fortsettelse         |   Sannsynlighet |
|----------------------|-----------------|
| objektivt.           |         0.11662 |
| objektivt,           |         0.05470 |
| objektivt,           |         0.05469 |
| på en objekt         |         0.00253 |
| i henhold til den av |         0.00036 |
