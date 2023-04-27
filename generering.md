# Modellbasert generering av tekst.

Eksemplene i artikkelen der vi genererer fortsettelse av tekst bruker OpenAIs API. Python-koden som ble brukt er dokumentert nedenfor. Koden forutsetter en "egen_openai_api_key".

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

Hvis vi ber modellen om å gi fem sannsynlige forrsettelser for "Hovedprinsippet i norsk avtalerett er" får vi:

| Fortsettelse          |   Sannsynlighet |
|-----------------------|-----------------|
| at kontrakter sk      |         0.00906 |
| at partene er fritt   |         0.00536 |
| at partene selv sk    |         0.00488 |
| at det skal legg      |         0.00467 |
| at partene har frihet |         0.00352 |

Så kan vi legge inn fortsettelsen "at kontrakter". Det ber vi modellen om å gi fem sannsynlige fortsettelser på "Hovedprinsippet i norsk avtalerett er at kontrakter".

| Fortsettelse      |   Sannsynlighet |
|-------------------|-----------------|
| skal oppfylles    |         0.04556 |
| skal tolkes obj   |         0.03115 |
| skal tolkes inn   |         0.00517 |
| skal tolkes d     |         0.00266 |
| er bindene. Dette |         0.00125 |

Tre av de videre alternativene fortsetter med "skal tolkes". La oss legge til dette og be modellen om å fortsette følgende tekst "Hovedprinsippet i norsk avtalerett er at kontrakter skal tolkes"

| Fortsettelse         |   Sannsynlighet |
|----------------------|-----------------|
| objektivt.           |         0.11662 |
| objektivt,           |         0.05470 |
| objektivt,           |         0.05469 |
| på en objekt         |         0.00253 |
| i henhold til den av |         0.00036 |


La oss gi mer kontekst. Vi legger inn følgende tekst (her har vi økt max_tokens til 10 for å få hele sammenhengen):

<code>
Peder og Marte er uenige om salget av en bil. Peder hadde lagt ut en Volvo til salgs. Marte var interessert i å kjøpe den. De har sendt mange SMSer til hverandre og har forhandlet om pris. Peder hadde først lagt ut bilen til kr 90000,–, men Marte prutet og bød til slutt kr 74 000,– for bilen i en SMS. Peder syntes budet var for lav, men sendte etter tre dagers nøling et svar på SMS til Marte der det stod «OK. Det er greit! Du kan hente bilen i morgen.» Peder hørte ikke noe mer fra Marte og ringte for å spørre når hun henter bilen. Marte sier at hun ikke fikk noe svar fra Peder og at hun nå har kjøpt en annen bil. Peder sier at de har en avtale og at hun må stå ved den. Hovedprinsippet i norsk avtalerett er
</code>

Nå gir modellen følgende mulige fortsettelser:

| Fortsettelse                 |   Sannsynlighet |
|------------------------------|-----------------|
| at avtaler som er innga      |         0.01083 |
| at avtaler som inngå        |         0.00957 |
| at avtaler som er kommet til |         0.00844 |
| at avtaler som er kommet til |         0.00842 |
| at avtaler skal holdes. Det  |         0.00469 |


Hvis vi ber om bare ett svar, men øker lengden til 1000 tokens får vi følgende svar:

<code>
...at man skal holde det man har avtalt.
<p>
I dette tilfellet har Peder og Marte kommet til en avtale om salget av bilen. Dette er bekreftet av den avtalen som har blitt inngått gjennom SMSer. Derfor er det nødvendig at begge parter står ved denne avtalen. Det er viktig at man holder det man har avtalt, og at man ikke forsøker å gjøre endringer i avtalen uten å ha kommet til enighet med den andre parten. Hvis Marte ikke ønsker å stå ved avtalen, kan hun bli tvunget til å gjøre det av domstolene.</p>
</code>
