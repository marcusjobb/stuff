# Gratis AI-API varje dag? Ja tack!

*Publicerad av [Marcus Medina🔗](https://www.linkedin.com/in/marcusmedina/) – Kodpedagog, AI-nörd och smått beroende av bra API*\*:er\* 

## Lär dig AIs i en gratismiljö?

Har du också tröttnat på att köpa tokens som om det vore godis på lösvikt? 1MinAI är ett **enkelt, snabbt och generöst AI-API** där du kan leka fritt med språkmodeller som GPT-4o, Claude, Gemini och Mistral.
Vill du sedan utveckla större saker så finns möjligheten att [betala för fler tokens](https://1min.ai/?referrer_id=9b12faba-f8a3-49ea-bd25-d8343f1fd965#pricing)

Då har jag ett hett tips till dig: **[1minAI](https://1min.ai/?referrer_id=9b12faba-f8a3-49ea-bd25-d8343f1fd965)**
Det är gratis att komma igång, du får **15 000 tokens varje dag** bara av att logga in – och om du använder min referenslänk får du dessutom en **välkomstbonus på 200 000 tokens**.

## Vad är 1minAI?

**1minAI** är ett lättanvänt API som låter dig:

* Chatta med GPT-4o och Claude 3 gratis
* Generera bilder, kod och text via samma endpoint
* Välja bland massor av modeller
* Bygga egna appar utan att bekymra dig om OpenAI-priser
* Den har en hel frös med specialfunktioner som kan köras mot olika modeller

> Tips: Se till att logga in varje dag för att få mer tokens.

## Kom igång på 3 minuter

Du behöver:

* Ett gratis 1minAI-konto (signa upp här: [1minAI](https://1min.ai/?referrer_id=9b12faba-f8a3-49ea-bd25-d8343f1fd965))
* Du skapar API-nyckel genom att klicka på ditt team högst upp till vänster, och i menyn API.
* En miljövariabel `ONEMINAI_API_KEY` på din dator eller en `.env`-fil (som du absolut aldrig committar till GitHub)
* Några rader kod – se nedan!
* Logga in dagligen för mer [gratis credits](https://1min.ai/?referrer_id=9b12faba-f8a3-49ea-bd25-d8343f1fd965#free-credits)
Observera att koden inte har minne för meddelanden, det kommer i nästa exempel. Jag lägger upp exempel för att deras API inte fungerar exakt som OpenAIs.

---

## Exempelkod i Node.js

```js
// aiChat.js
import readline from 'readline';
import fetch from 'node-fetch';

const apiKey = process.env.ONEMINAI_API_KEY;
if (!apiKey) {
  console.error("Sätt miljövariabeln ONEMINAI_API_KEY.");
  process.exit(1);
}

const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
console.log("Skriv något till AI:n (skriv 'exit' för att avsluta)");

const ask = () => rl.question('\n> ', async (input) => {
  if (input.trim().toLowerCase() === 'exit') return rl.close();

  const payload = {
    type: "CHAT_WITH_AI",
    model: "gpt-4o-mini",
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

---

## Samma sak i C\#

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

        while (true)
        {
            Console.Write("\n> ");
            var input = Console.ReadLine();
            if (input?.ToLower() == "exit") break;

            var json = JsonSerializer.Serialize(new
            {
                type = "CHAT_WITH_AI",
                model = "gpt-4o-mini",
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

---

## Vilka AI-modeller kan du använda?

1minAI låter dig välja bland en lång lista av kraftfulla språkmodeller – både från OpenAI, Google, Anthropic, Meta, Mistral, och andra. Här är några exempel:

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

### Tokenkostnader varierar

Ju nyare och mer avancerad modellen är, desto fler tokens kan den kosta per anrop. Exempelvis:

* **gpt-4o-mini** → låg kostnad, perfekt för enkla experiment
* **Claude 3 Opus** → hög kapacitet, men dyrare
* **Gemini 1.5 Pro** → avancerad Google-modell, kostar mer tokens
* **Mixtral 8x22b** → experimentell open source med bra prestanda

Du ser inte tokenkostnaden direkt i API\:t, men det märks i hur snabbt dina dagliga tokens används upp. Håll utkik efter deras nyhetsbrev – där listas nya modeller löpande!

---

## Kliv in i AI-världen utan kostnad

Det är sällsynt med så pass generösa AI-tjänster som faktiskt fungerar smidigt direkt. 1minAI har blivit ett av mina favoritsätt att testa kodexempel med studerande,
experimentera med olika modeller, och till och med bygga egna appar – utan att det kostar ett öre. Vilket är viktigt för studerande.

👉 Testa själv via:
🔗 [https://1min.ai/?referrer\_id=9b12faba-f8a3-49ea-bd25-d8343f1fd965](https://1min.ai/?referrer_id=9b12faba-f8a3-49ea-bd25-d8343f1fd965)

---

### PS: Vill du se fler artiklar, kodexempel och AI-hacks?

Tryck gärna på ⭐ om du gillar projektet.

