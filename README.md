# LibreTranslate - Local Instance

Self-hosted instance of [LibreTranslate](https://github.com/LibreTranslate/LibreTranslate), a free and open source machine translation API.

## Supported Languages

| Code | Language |
|------|----------|
| en   | English  |
| fr   | French   |
| de   | German   |
| es   | Spanish  |
| ru   | Russian  |
| zh   | Chinese  |

## Prerequisites

- Docker
- Docker Compose

## Getting Started

```bash
docker compose up -d
```

The web interface is available at [http://127.0.0.1:5000](http://127.0.0.1:5000).

The first startup may take a while as translation models are downloaded.

## Stopping

```bash
docker compose down
```

## API Usage

```bash
curl -X POST http://127.0.0.1:5000/translate \
  -d '{"q":"Hello world","source":"en","target":"fr"}' \
  -H "Content-Type: application/json"
```

## Configuration

| Variable       | Value               | Description                        |
|----------------|---------------------|------------------------------------|
| `LT_LOAD_ONLY` | `en,fr,de,es,ru,zh` | Languages loaded at startup        |
| `LT_THREADS`   | `8`                 | Number of threads for translations |

### Tuning `LT_THREADS`
 
`LT_THREADS` controls how many translation requests can be processed in parallel. This is especially useful when translating full web pages, as the browser extension splits the page into multiple chunks.
 
To find the optimal value, check the number of available CPU threads on your machine:
 
```bash
nproc
```
 
Then choose a value based on how much CPU you want to dedicate to translations. Example for a 16-thread machine:
 
| `LT_THREADS` | Impact                                                              |
|--------------|---------------------------------------------------------------------|
| `4`          | Safe default, minimal impact on system performance                  |
| `8`          | Good balance between translation speed and system responsiveness    |
| `12`         | Fast translations, but the system may feel sluggish during requests |
| `16`         | Maximum speed, but the system will be unresponsive during requests  |

## Resources

- Memory is limited to **4 GB**.
- CPU is unlimited to maximize translation speed.
- Models are persisted in the `libretranslate_models` volume.

## Re-downloading Models

```bash
docker compose down
docker volume rm libretranslate_models
docker compose up -d
```

## Browser Integration (Linguist)

The [Linguist](https://linguister.io/) extension allows you to use this instance to translate web pages. To set it up, add a custom translator in the extension options with the following code:

```javascript
class LibreTranslator {
    static apiPath = "http://127.0.0.1:5000";

    translate = (text, from, to) => {
        const form = new FormData();
        form.append("q", text);
        form.append("source", from);
        form.append("target", to);
        form.append("format", "text");

        return fetch(LibreTranslator.apiPath + "/translate", {
            method: "POST",
            body: form,
        })
            .then((r) => r.json())
            .then(({ translatedText }) => translatedText);
    };

    translateBatch = (texts, from, to) =>
        Promise.all(texts.map((text) => this.translate(text, from, to)));

    getLengthLimit = () => 5000;
    getRequestsTimeout = () => 300;
    checkLimitExceeding = (text) => {
        const textLength = Array.isArray(text)
            ? text.reduce((len, t) => len + t.length, 0)
            : text.length;
        return textLength - this.getLengthLimit();
    };

    static isSupportedAutoFrom = () => true;
    static getSupportedLanguages = () => ['en', 'fr', 'de', 'es', 'ru', 'zh'];
}

LibreTranslator;
```