# Hur man använder 1minAI – en nybörjarguide

_Publicerad av [Marcus Medina](https://www.linkedin.com/in/marcusmedina/) – Kodpedagog, AI-nörd och smått beroende av bra API:er_

---

## Lär dig AIs i en gratismiljö?

Har du också tröttnat på att köpa tokens som om det vore godis på lösvikt? 1minAI är ett **enkelt, snabbt och generöst AI-API** där du kan leka fritt med språkmodeller som GPT-4o, Claude, Gemini och Mistral.

Vill du sedan utveckla större saker så finns möjligheten att [betala för fler tokens](https://1min.ai?referrer_id=9b12faba-f8a3-49ea-bd25-d8343f1fd965#pricing).

Då har jag ett hett tips till dig: **[1minAI](https://1min.ai?referrer_id=9b12faba-f8a3-49ea-bd25-d8343f1fd965)**

Det är gratis att komma igång, du får **15 000 tokens varje dag** bara av att logga in.

## Vad är 1minAI?

1minAI är en **allt-i-ett AI-plattform** som låter dig chatta med olika språkmodeller (som GPT-4, ChatGPT, Claude m.fl.) via ett enda API. Tanken är att du snabbt ska kunna integrera AI-funktioner i dina egna program eller projekt. I den här guiden visar jag hur du kommer igång med 1minAIs API, steg för steg.

Vi kommer att fokusera på hur man **skapar konversationer** och skickar meddelanden via API:et. Jag visar exempel i både Node.js och C# – två populära språk för backend-utveckling. Under resans gång förklarar jag också viktiga koncept som _conversationId_ (konversations-ID) och _konversationsminne_ (hur AI:n minns vad ni pratat om tidigare). Så om du vill bygga en smart chatbot som kommer ihåg vad du sagt, då har du kommit rätt!

Du kan:

- Chatta med GPT-4o och Claude 3 gratis
- Generera bilder, kod och text via samma endpoint
- Välja bland massor av modeller
- Bygga egna appar utan att bekymra dig om OpenAI-priser
- Utnyttja specialfunktioner unikt för olika modeller

> **Tips: Se till att logga in varje dag för att få mer tokens.**

💡 Bonus: Om du registrerar dig via länken ovan får både du och jag **200 000 extra credits** att leka med direkt – perfekt för att testa GPT-4o, Claude och andra avancerade modeller utan att det kostar något!

## Kom igång på 3 minuter

Du behöver:

- Ett gratis 1minAI-konto ([1minAI](https://app.1min.ai?referrer_id=9b12faba-f8a3-49ea-bd25-d8343f1fd965))
- En [API-nyckel](https://app.1min.ai/api?referrer_id=9b12faba-f8a3-49ea-bd25-d8343f1fd965) (från team-menyn)
- En miljövariabel `ONEMINAI_API_KEY` eller `.env`-fil (aldrig committa denna!)
- Några rader kod – se exempel nedan

## Skapa en konversation med API:et

Innan vi kan börja chatta med AI:n via koden behöver vi skapa en **konversation** på 1minAIs server. Tänk på en konversation som en chattrumstråd – den har ett unikt ID (conversationId) som låter 1minAI hålla reda på vad som sagts tidigare.

För att skapa en ny konversation skickar man en HTTP POST-förfrågan till endpointen `/api/conversations`. I JSON-kroppen **måste** man ange `"type": "CHAT_WITH_AI"`. Det talar om för 1minAI att vi vill starta en vanlig chatt med en AI-modell (det finns andra typer för bildgenerering m.m., men här fokuserar vi på chatten).

Ett minimaliskt exempel på JSON för att starta en konversation ser ut så här:

```json
{
  "type": "CHAT_WITH_AI"
}
```

Det är allt som krävs för att sparka igång en konversation! 🚀 När servern tar emot denna förfrågan kommer den att skapa en ny konversation åt dig. **Du kan (och ska) inte skicka med något eget här – servern sköter det.**

### Varför genereras conversationId automatiskt?

Varje gång du startar en konversation genererar 1minAI ett unikt ID (en UUID) för just den konversationen. Detta ID returneras i svaret, vanligtvis som `conversation.uuid` i JSON-svaret. Exempelvis kan svaret innehålla något i stil med:

```json
{
  "conversation": {
    "uuid": "e4f1b2d0-1234-4abc-90ef-98765dcba0f1"
  },
  "message": "Conversation created successfully"
}
```

Det viktiga här är **du får conversationId från servern**, och det är detta ID vi ska använda framöver för att fortsätta chatta i just den tråden.

**Kan man sätta `conversationId` själv manuellt?** Nej. Försök inte vara smart genom att hitta på ett eget ID – 1minAI ignorerar det (eller kastar fel). Jag erkänner, jag testade själv först att skicka med ett eget `conversationId` i tron att jag kunde styra det, men servern sa blankt nej. 😉 Alltså: **låt servern skapa ID:t åt dig.** Detta garanterar att ID:t är unikt och att konversationsminnet fungerar som det ska.

Summa summarum: gör en POST till `/api/conversations` med `type: "CHAT_WITH_AI"`. I svaret plockar du ut `conversation.uuid`. Spara detta ID, för vi behöver det när vi skickar meddelanden så att AI:n vet vilken konversation den pratar i. I nästa avsnitt ska vi se hur vi använder detta i praktiken.

## Exempel i Node.js: starta och återanvänd en konversation

Låt oss börja med ett Node.js-exempel. Här skriver vi ett litet skript som:

1. Kollar om vi redan har ett sparat conversationId (från en tidigare körning).
2. Om inte, skapar en ny konversation via API:et och sparar det erhållna conversationId i en fil.
3. Använder conversationId för att skicka ett meddelande till AI:n och få svar.
4. Sparar conversationId så att om vi kör igen kan vi återanvända samma konversation (AI:n minns då tidigare meddelanden).

Koden nedan demonstrerar detta. Vi använder paketet **axios** för HTTP-förfrågningar (men du kan lika gärna använda inbyggda `fetch` om du kör Node 18+). Vi använder också Node:s inbyggda `fs` modul för filhantering.

```javascript
// 📦 Importera moduler
const fs = require("fs/promises");
const fetch = require("node-fetch");

// 🔐 Läs API-nyckel från miljövariabel
const apiKey = process.env.ONEMINAI_API_KEY;
if (!apiKey) {
  console.error("⚠️  Sätt miljövariabeln 'ONEMINAI_API_KEY'.");
  process.exit(1);
}

const filePath = "test-session-123.txt";
let conversationId = null;

// 🔁 Läs eller skapa konversation
async function getConversationId() {
  try {
    const data = await fs.readFile(filePath, "utf-8");
    conversationId = data.trim();
    console.log(`🔁 Återanvänder konversation: ${conversationId}`);
  } catch {
    // 🧠 Skapa ny konversation
    const payload = {
      title: "test-session",
      type: "CHAT_WITH_AI",
      model: "gpt-4o-mini",
    };

    const res = await fetch("https://api.1min.ai/api/conversations", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "API-KEY": apiKey,
      },
      body: JSON.stringify(payload),
    });

    const json = await res.json();

    if (!json.conversation || !json.conversation.uuid) {
      console.error("❌ Kunde inte skapa konversation (uuid saknas):", json);
      process.exit(1);
    }

    conversationId = json.conversation.uuid;
    await fs.writeFile(filePath, conversationId);
    console.log(`✅ Skapade ny konversation: ${conversationId}`);
  }
}

// 💬 Kör chattloop
async function chatLoop() {
  const readline = require("readline").createInterface({
    input: process.stdin,
    output: process.stdout,
  });

  console.log("💬 Skriv något till AI:n (skriv 'exit' för att avsluta)");

  const ask = (query) =>
    new Promise((resolve) => readline.question(query, resolve));

  while (true) {
    const input = await ask("\n> ");
    if (input.toLowerCase() === "exit") {
      readline.close();
      break;
    }

    const payload = {
      type: "CHAT_WITH_AI",
      model: "gpt-4o-mini",
      conversationId,
      promptObject: {
        prompt: input,
        temperature: 0.7,
        max_tokens: 200,
        top_p: 0.9,
      },
    };

    try {
      const res = await fetch("https://api.1min.ai/api/features", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "API-KEY": apiKey,
        },
        body: JSON.stringify(payload),
      });

      const reply = await res.json();

      if (reply.error) {
        console.error(`❌ Fel: ${reply.error}`);
      } else {
        const result =
          reply.aiRecord?.aiRecordDetail?.resultObject?.[0] || "🫥 Inget svar.";
        console.log(`\n🤖 AI: ${result}`);
      }
    } catch (err) {
      console.error(`🚨 Kunde inte tolka svaret: ${err.message}`);
    }
  }
}

// 🏁 Main
(async () => {
  await getConversationId();
  await chatLoop();
})();
```

Låt oss gå igenom vad som händer:

- Först försöker vi läsa filen `conversationId.txt`. Om den finns innebär det att vi har ett tidigare sparat conversationId, och vi kan återanvända det. Vi loggar att vi återanvänder befintlig konversation.
- Om filen **inte** finns (första gången vi kör, eller om vi vill starta en ny session), skickar vi en POST till `https://api.1min.ai/api/conversations` med kroppen `{ "type": "CHAT_WITH_AI" }`. Lägg märke till att vi sätter header **Content-Type: application/json** och även **API-KEY** (det är så vi autentiserar oss mot API:et – använd din egna nyckel här).
- När svaret kommer tillbaka plockar vi ut `conversationId` från `response.data.conversation.uuid`. Vi loggar det och sparar det till fil så att vi kan köra om skriptet senare och fortsätta samma chatt.
- Därefter förbereder vi ett _prompt_-meddelande (här `"Hej, hur mår du?"` som exempel). Vi bygger upp en ny JSON med `type`, väljer en `model` (här har jag använt `"gpt-4o-mini"` – 1minAI har olika modeller, du kan byta efter behov), och under `promptObject` anger vi själva prompten. I promptObject kan man också skicka med saker som bilder (`imageList`), be AI:n göra webbsökning (`webSearch`: true/false), m.m. I vårt fall håller vi det enkelt.
- Vi skickar denna som en POST till `https://api.1min.ai/api/features?isStreaming=false`. (Query-parametern `isStreaming=false` anger att vi vill ha svaret som en hel klump och inte streamat stegvis.)
- I svaret (`messageResponse`) får vi tillbaka AI:ns svar på vår fråga. Svaret innehåller vanligtvis själva textsvaret från AI:n och ibland metadata. Vi här plockar bara ut hela `data`-objektet och loggar ut det.

När du kör skriptet bör det alltså, första gången, skapa en ny konversation och svara på din prompt. Andra gången du kör (med filen kvar) bör det **inte** skapa en ny konversation utan istället säga "Återanvänder befintlig konversation" – och AI:n kommer då ihåg att du redan hälsat. Om du t.ex. andra gången frågar något relaterat som _"Kan du utveckla det?"_ så vet AI:n vad "det" syftar på, tack vare att du återanvänder samma conversationId.

> **Tips:** Om du vill starta en helt ny konversation från scratch (t.ex. för ett nytt samtalsämne), kan du ta bort/ändra filen som sparar conversationId så att skriptet skapar en ny. Du kan också spara olika filnamn för olika sessioner om du vill hålla isär flera parallella konversationer.

## Exempel i C#: spara konversations-ID mellan sessioner

Okej, dags att titta på samma logik i C#. Principen är exakt densamma:

- Kontrollera om en fil (t.ex. `test-session-123.txt`) med ett sparat conversationId finns.
- Om inte, gör en POST till `/api/conversations` för att få ett nytt ID, och spara det i filen.
- Använd sedan ID:t för att skicka en prompt till AI:n och få svar.
- Skriv ut svaret och spara om nödvändigt ID:t för framtida bruk.

I C# använder vi klassen **HttpClient** för att göra HTTP-anrop. Vi kommer också att använda `System.Text.Json` för att parsa JSON-svaret och hämta ut `conversationId`. Glöm inte att lägga till `using System.Net.Http;` och `using System.Text.Json;` i din kod. Kommenterar på svenska inline för att förklara stegen:

```csharp
using System;
using System.IO;
using System.Net.Http;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;

static class Program
{
    static readonly HttpClient client = new();

    static async Task Main()
    {
        var apiKey = Environment.GetEnvironmentVariable("ONEMINAI_API_KEY");
        if (string.IsNullOrWhiteSpace(apiKey))
        {
            Console.WriteLine("⚠️  Sätt miljövariabeln 'ONEMINAI_API_KEY'.");
            return;
        }

        client.DefaultRequestHeaders.Add("API-KEY", apiKey);

        string path = "test-session-123.txt";
        string conversationId;

        // 🔁 Läs eller skapa samtal
        if (File.Exists(path))
        {
            conversationId = File.ReadAllText(path).Trim();
            Console.WriteLine($"🔁 Återanvänder konversation: {conversationId}");
        }
        else
        {
            // 🧠 Rätt payload – med "type"
            var createJson = JsonSerializer.Serialize(new
            {
                title = "test-session",
                type = "CHAT_WITH_AI",
                model = "gpt-4o-mini"
            });

            var res = await client.PostAsync("https://api.1min.ai/api/conversations",
                new StringContent(createJson, Encoding.UTF8, "application/json"));

            var content = await res.Content.ReadAsStringAsync();
            var doc = JsonDocument.Parse(content);

            if (!doc.RootElement.TryGetProperty("conversation", out var conv) ||
                !conv.TryGetProperty("uuid", out var idProp))
            {
                Console.WriteLine("❌ Kunde inte skapa konversation (uuid saknas):");
                Console.WriteLine(content);
                return;
            }

            conversationId = idProp.GetString();
            File.WriteAllText(path, conversationId);
            Console.WriteLine($"✅ Skapade ny konversation: {conversationId}");
        }

        Console.WriteLine("💬 Skriv något till AI:n (skriv 'exit' för att avsluta)");

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
                promptObject = new
                {
                    prompt = input,
                    temperature = 0.7,
                    max_tokens = 200,
                    top_p = 0.9
                }
            });

            using var res = await client.PostAsync(
                "https://api.1min.ai/api/features",
                new StringContent(json, Encoding.UTF8, "application/json")
            );

            var reply = await res.Content.ReadAsStringAsync();

            try
            {
                var doc = JsonDocument.Parse(reply);
                if (doc.RootElement.TryGetProperty("error", out var err))
                {
                    Console.WriteLine($"❌ Fel: {err.GetString()}");
                }
                else
                {
                    var result = doc.RootElement
                        .GetProperty("aiRecord")
                        .GetProperty("aiRecordDetail")
                        .GetProperty("resultObject")[0]
                        .GetString();

                    Console.WriteLine($"\n🤖 AI: {result}");
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"🚨 Kunde inte tolka svaret: {ex.Message}");
            }
        }
    }
}
```

Vi gör alltså precis samma sak som i Node-exemplet men på C#-sätt:

- Kollar om filen `test-session-123.txt` finns. Om ja, läser vi in den och får vårt `conversationId`. Om nej, skickar vi en begäran för att skapa konversation.
- För att skapa konversationen bygger vi en `HttpRequestMessage` med metoden POST mot `/api/conversations`. Vi lägger till vår API-nyckel som header (`API-KEY`) och en JSON-content som innehåller `{"type":"CHAT_WITH_AI"}`. (Notera: vi behöver inte ange något conversationId här – det fixar servern åt oss när vi inte skickar med något.)
- Vi skickar begäran och väntar på svar. Svaret (`responseJson`) är en JSON-sträng. Vi använder `JsonDocument` för att plocka ut `conversation.uuid` värdet och sparar det som vår `conversationId`. Därefter skriver vi det till filen `test-session-123.txt` för senare bruk.
- Sedan gör vi i ordning vår _prompt_-förfrågan. Här valde jag att använda `JsonSerializer.Serialize` för att slippa hand-knacka JSON-strängen (det är lätt att göra fel med citationstecken annars). Vi serialiserar ett anonymt objekt med samma struktur: `type`, `model` och `promptObject` med prompttexten.
- Denna JSON skickar vi som en POST till `/api/features?isStreaming=false`, återigen med API-nyckel i header. (Vi skickar med `isStreaming=false` för enkelhet, så får vi hela svaret på en gång.)
- Slutligen tar vi emot svaret i `answerJson`. Just nu skriver vi bara ut hela råa JSON-svaret från AI:n. Om du vill kan du förstås parsa ut just svaret i textform och skriva ut det. (Det brukar finnas någon property som innehåller AI:ns svarstext.)

Kör du programmet i C# får du samma beteende som i Node: Första körningen skapar ny konversation, senare körningar återanvänder den. Du kan öppna `test-session-123.txt` och se ditt konversations-ID (som en lång UUID-sträng).

**TIPS:** Vill du börja en ny konversation i C#-exemplet kan du radera eller byta namn på `test-session-123.txt` så kommer koden skapa en ny konversation nästa gång.

## Så fungerar konversationsminnet i 1minAI

Du kanske undrar vad som händer "bakom kulisserna" när vi återanvänder ett conversationId. Hur kommer det sig att AI:n plötsligt minns vad ni pratade om tidigare? Hemligheten ligger i **konversationsminnet** som 1minAI hanterar åt dig.

När du skickar meddelanden med ett conversationId håller 1minAI reda på all dialog som skett i den konversationen. Varje meddelande (din prompt och AI:ns svar) sparas på serversidan kopplat till konversationens unika ID. Nästa gång du skickar en prompt med samma conversationId, plockar 1minAI fram den historiken och ger den till AI-modellen som kontext. På så sätt _kommer AI:n ihåg_ vad som sagts tidigare, utan att du själv behöver skicka med hela konversationshistoriken varje gång. Smidigt, eller hur?

**Varför är detta viktigt?** Tänk dig att du bygger en chatbot där användaren kan ställa följdfrågor. Utan konversationsminne skulle AI:n behandla varje fråga som fristående – den skulle bli som Doris i "Hitta Nemo" med minne på 10 sekunder! 🐠 Du skulle behöva upprepa all relevant information i varje fråga för att få vettiga svar. Med konversationsminnet aktiverat slipper du det:

- **Kontinuitet:** AI:n kan referera till tidigare nämnda ämnen, personer eller detaljer. Frågar du "_Kan du förklara det där lite mer?_" vet den vad _"det där"_ syftar på om det finns i historiken.
- **Sparar tid och promptutrymme:** Du slipper skicka med hela chatthistoriken varje gång (till skillnad från OpenAIs renodlade API där man själv måste skicka med tidigare dialog om man vill ha minne). 1minAI sköter lagringen åt dig.
- **Bättre användarupplevelse:** Samtalet känns mer naturligt. Användaren kan chatta precis som med en människa, där båda parter minns vad som sagts tidigare.

Under huven har dock alla AI-modeller en gräns för hur mycket text de kan komma ihåg (en så kallad _kontekstfönster_ eller token-limit). 1minAI hanterar detta genom att sannolikt begränsa hur långt tillbaka i historiken AI:n "ser". Äldre meddelanden kan falla bort om konversationen blir väldigt lång, för att hålla sig inom modellens gränser. För praktiska syften i normallånga samtal märker man oftast inte av detta – men det kan vara bra att känna till att minnet inte är oändligt.

**Sammanfattning:** Konversationsminnet i 1minAI gör att din AI-kompanjon slipper guldfiskminne 🐟. Genom att återanvända `conversationId` vid varje API-anrop inom samma chatt-session, kan AI:n bygga vidare på tidigare konversation. Det är en av de största fördelarna med att använda just 1minAIs API för chat – du får en _kontextmedveten_ AI med minimal ansträngning.

## Så var det med det

Nu har vi gått igenom hur man använder 1minAIs API för att skapa konversationer och skicka meddelanden, både i Node.js och C#. Vi har lärt oss att:

- Alltid initiera en ny konversation via POST `/api/conversations` med `"type": "CHAT_WITH_AI"` och låta servern ge oss ett `conversationId`.
- Spara det ID:t och återanvända det för följdmeddelanden, så att AI:n minns tidigare dialog.
- Inte försöka sätta `conversationId` själv – servern är envis med att generera unika ID:n åt oss.
- Förstå att konversationsminnet är nyckeln till att få en sammanhängande och trevlig dialog med AI:n över flera frågor/svar.

Förhoppningsvis känner du dig redo att integrera 1minAI i ditt eget projekt nu. 🎉 Var inte rädd för att experimentera med olika modeller (`model`-parametern) och funktioner som webbsökning eller bildanalys som 1minAI erbjuder. Och framför allt – ha kul med din AI-utveckling!

_Lycka till och happy coding!_ 🚀

---

## Lista på modeller

```text
gpt-4o, gpt-4o-turbo, gpt-3.5-turbo, gemini-1.5-flash, gemini-1.5-pro,
o1-mini, o3-mini, o1-preview,
claude-3-5-haiku-20241022, claude-3-5-sonnet-20240620,
claude-3-opus-20240229, claude-3-sonnet-20240229, claude-3-haiku-20240307,
mistral-large-latest, mistral-nemo, pixtral-12b,
open-mixtral-8x22b, open-mixtral-8x7b,
meta/meta-llama-3.1-405b-instruct, meta/llama-2-70b-chat,
deepseek-chat, gemini-1.0-pro
```

---

## Vad är konversationsminne?

1minAI lagrar tidigare frågor och svar kopplat till ett `conversationId`, vilket gör att AI:n kan svara kontextmedvetet. Nästa gång du skriver "Kan du utveckla det?" förstår den vad "det" syftar på, eftersom historiken är sparad.

Detta gör det enkelt att bygga chatbotar, undervisningshjälp, eller AI-assistenter som förstår samtalsflödet över tid.

Vill du rensa kontexten? Skapa en ny konversation.

## Avslutning

Jag använder 1minAI både privat och i undervisning. Det är stabilt, snabbt och billigt (gratis!). Tack vare deras tokenmodell kan du prova på avancerade modeller utan stress. Och ja – det fungerar faktiskt.

👉 [Testa gratis idag](https://app.1min.ai?referrer_id=9b12faba-f8a3-49ea-bd25-d8343f1fd965) och låt mig veta hur det gick!

---

_Tryck “Stjärna” ✨ om du gillar artikeln. Eller dela med en kollega som behöver en AI-booster i vardagen._
