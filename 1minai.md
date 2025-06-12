# Gratis AI-API varje dag, med 1minAI

*Publicerad av [Marcus Medina](https://www.linkedin.com/in/marcusmedina/) – Kodpedagog, AI-nörd och smått beroende av bra API\:er*

## Lär dig AIs i en gratismiljö?

Har du också tröttnat på att köpa tokens som om det vore godis på lösvikt? 1MinAI är ett **enkelt, snabbt och generöst AI-API** där du kan leka fritt med språkmodeller som GPT-4o, Claude, Gemini och Mistral.

Vill du sedan utveckla större saker så finns möjligheten att [betala för fler tokens](https://1min.ai).

Då har jag ett hett tips till dig: **[1minAI](https://1min.ai)**

Det är gratis att komma igång, du får **15 000 tokens varje dag** bara av att logga in.

---

## Vad är 1minAI?

**1minAI** är ett lättanvänt API som låter dig:

* Chatta med GPT-4o och Claude 3 gratis
* Generera bilder, kod och text via samma endpoint
* Välja bland massor av modeller
* Bygga egna appar utan att bekymra dig om OpenAI-priser
* Utnyttja specialfunktioner unikt för olika modeller

> Tips: Se till att logga in varje dag för att få mer tokens.

---

## Kom igång på 3 minuter

Du behöver:

* Ett gratis 1minAI-konto ([1minAI](https://1min.ai))
* En API-nyckel (från team-menyn)
* En miljövariabel `ONEMINAI_API_KEY` eller `.env`-fil (aldrig committa denna!)
* Några rader kod – se exempel nedan

---

## Enkel fråga till API\:t med minne

**Input:**

```json
{
  "type": "CHAT_WITH_AI",
  "model": "gpt-4o-mini",
  "conversationId": "test-session-123",
  "promptObject": {
    "prompt": "Om en höna lägger ett ägg på 1 timme, hur många ägg lägger 3 hönor på en halvtimme",
    "temperature": 0.7,
    "max_tokens": 200,
    "top_p": 0.9
  }
}
```

**Svar:**

```json
{
  "aiRecord": {
    "uuid": "...",
    "model": "gpt-4o-mini",
    "type": "CHAT_WITH_AI",
    "conversationId": "test-session-123",
    "status": "SUCCESS",
    "aiRecordDetail": {
      "promptObject": {
        "top_p": 0.9,
        "prompt": "Om en höna lägger ett ägg på 1 timme, hur många ägg lägger 3 hönor på en halvtimme",
        "max_tokens": 200,
        "temperature": 0.7
      },
      "resultObject": [
        "Om en höna lägger ett ägg på 1 timme, så lägger en höna hälften av ett ägg på en halvtimme. Därför skulle tre hönor tillsammans lägga 3 x 0.5 = 1,5 ägg på en halvtimme. Så svaret är 1,5 ägg."
      ]
    }
  }
}
```

> Genom att återanvända samma `conversationId` kan du bygga upp ett minne i konversationen.

Vill du göra nästa fråga baserad på förra svaret? Lägg bara till en ny prompt med samma ID, till exempel:

```json
{
  "conversationId": "test-session-123",
  "promptObject": { "prompt": "Och hur många ägg på två timmar?" }
}
```

## Exempelkod i Node.js

<details>
<summary>Klicka för att visa Node.js-exempel</summary>

```js
import readline from 'readline';
import fetch from 'node-fetch';

const apiKey = process.env.ONEMINAI_API_KEY;
if (!apiKey) {
  console.error("Sätt miljövariabeln ONEMINAI_API_KEY.");
  process.exit(1);
}

const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
console.log("Skriv något till AI:n (skriv 'exit' för att avsluta)");

const conversationId = "test-session-123";

const ask = () => rl.question('\n> ', async (input) => {
  if (input.trim().toLowerCase() === 'exit') return rl.close();

  const payload = {
    type: "CHAT_WITH_AI",
    model: "gpt-4o-mini",
    conversationId,
    promptObject: { prompt: input, temperature: 0.7, max_tokens: 200, top_p: 0.9 }
  };

  try {
    const res = await fetch("https://api.1min.ai/api/features", {
      method: "POST",
      headers: { "API-KEY": apiKey, "Content-Type": "application/json" },
      body: JSON.stringify(payload)
    });

    const data = await res.json();
    const result = data?.aiRecord?.aiRecordDetail?.resultObject?.[0];
    result ? console.log(`\nAI: ${result}`) : console.error("Kunde inte tolka svaret.");
  } catch (e) {
    console.error("Undantag:", e.message);
  }

  ask();
});

ask();
```

</details>

---

## Samma sak i C\#

<details>
<summary>Klicka för att visa C#-exempel</summary>

```csharp
using System.Net.Http;
using System.Text.Json;
using System.Text;

static class Program
{
    static readonly HttpClient client = new();

    static async Task Main()
    {
        var apiKey = Environment.GetEnvironmentVariable("ONEMINAI_API_KEY");
        if (string.IsNullOrWhiteSpace(apiKey))
        {
            Console.WriteLine("Sätt miljövariabeln ONEMINAI_API_KEY.");
            return;
        }

        client.DefaultRequestHeaders.Add("API-KEY", apiKey);
        Console.WriteLine("Skriv något till AI:n (skriv 'exit' för att avsluta)");

        var conversationId = "test-session-123";

        while (true)
        {
            Console.Write("\n> ");
            var input = Console.ReadLine();
            if (input?.ToLower() == "exit") break;

            var json = JsonSerializer.Serialize(new
            {
                type = "CHAT_WITH_AI",
                model = "gpt-4o-mini",
                conversationId,
                promptObject = new { prompt = input, temperature = 0.7, max_tokens = 200, top_p = 0.9 }
            });

            using var res = await client.PostAsync("https://api.1min.ai/api/features",
                new StringContent(json, Encoding.UTF8, "application/json"));
            var reply = await res.Content.ReadAsStringAsync();

            using var doc = JsonDocument.Parse(reply);
            if (doc.RootElement.TryGetProperty("error", out var err))
                Console.WriteLine($"Fel: {err.GetString()}");
            else
            {
                var result = doc.RootElement
                    .GetProperty("aiRecord")
                    .GetProperty("aiRecordDetail")
                    .GetProperty("resultObject")[0]
                    .GetString();

                Console.WriteLine($"\nAI: {result}");
            }
        }
    }
}
```

</details>

---

## Vilka AI-modeller kan du använda?

<details>
<summary>Visa modellista</summary>

```text
gpt-4o, gpt-4o-turbo, gpt-3.5-turbo, gemini-1.5-flash, gemini-1.5-pro,
o1-mini, o3-mini, o1-preview,
claude-3-5-haiku-20241022, claude-3-5-sonnet-20240620,
claude-3-opus-20240229, claude-3-sonnet-20240229, claude-3-haiku-20240307,
mistral-large-latest, mistral-nemo, pixtral-12b,
open-mixtral-8x22b, open-mixtral-8x7b,
meta/meta-llama-3.1-405b-instruct, meta/llama-2-70b-chat,
deepseek-chat, gemini-1.0-pro etc etc...
```

</details>

> Tokenkostnader varierar – nyare modeller drar fler tokens, så experimentera och hitta balansen.

---

## Kliv in i AI-världen utan kostnad

Det är sällsynt med så pass generösa AI-tjänster som faktiskt fungerar smidigt direkt.

1minAI har blivit ett av mina favoritsätt att testa kodexempel med studerande,
experimentera med olika modeller, och till och med bygga egna appar – utan att det kostar ett öre.

👉 Testa själv via: [https://1min.ai](https://1min.ai)

---

### PS: Vill du se fler artiklar, kodexempel och AI-hacks?

Tryck gärna på ⭐ om du gillar projektet.
