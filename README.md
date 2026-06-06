# browser-harness-wrap

> Python/TypeScript SDK wrapper for **[Browser Harness](https://github.com/browser-use/browser-harness)** — a self-healing harness that enables LLMs to complete any browser task.

## What is Browser Harness?

Browser Harness is a Python library that connects an LLM (like Claude, GPT-4, etc.) directly to a real Chrome browser via a thin DevTools Protocol (CDP) harness. Instead of the agent guessing what to do, the harness **watches every step** and fills in what's missing during execution. The harness then improves itself every run — it learns from failures.

**Key features:**
- **Self-healing:** Agent writes helper scripts when it misses something; the harness auto-corrects
- **Real browser:** Connects to real Chrome (no Playwright/Selenium abstraction layer)
- **Domain skills:** Community-contributed per-site skills teach the agent real selectors, flows, and edge cases
- **Cloud browser ready:** Works with Browser Use Cloud (3 concurrent browsers, no care required)
- **Zero new syntax:** Agent just writes Python — harness translates to browser actions
- **Cross-platform:** ~1k lines across 4 core files, lightweight and maintainable

## What This Wrap Provides

This wrapper gives you a **Python SDK** and **TypeScript SDK** to:

- Launch browser harness sessions programmatically
- Register custom domain skills
- Stream agent interactions
- Integrate browser automation into AI agent pipelines

## Installation

```bash
# Python
pip install browser-harness-wrap

# Or from source
pip install .
```

```bash
# TypeScript / Node.js
npm install browser-harness-wrap
# or
yarn add browser-harness-wrap
```

## Python Demo

### Basic Usage

```python
import asyncio
from browser_harness_wrap import BrowserHarness

async def main():
    harness = BrowserHarness(
        chrome_url="chrome://inspect/#devices",
        remote_debugging_port=9222,
    )

    # Connect to your Chrome browser
    await harness.connect()

    # Run a task — the agent will interact with the real browser
    result = await harness.run_task(
        task="Go to github.com and search for the repository 'facebook/react'",
        model="claude",  # or "gpt-4", "gemini", etc.
    )

    print(result)
    await harness.close()

asyncio.run(main())
```

### With Custom Domain Skills

```python
import asyncio
from browser_harness_wrap import BrowserHarness, DomainSkill

async def main():
    harness = BrowserHarness(
        chrome_url="chrome://inspect/#devices",
        remote_debugging_port=9222,
    )

    # Register a custom domain skill for a specific site
    await harness.register_skill(
        DomainSkill(
            domain="linkedin.com",
            skills=[
                {
                    "name": "search_people",
                    "description": "Search for people on LinkedIn",
                    "selectors": {
                        "search_box": "input[role='combobox']",
                        "results": "span[aria-label='Search results']",
                    },
                    "flow": [
                        {"action": "click", "target": "search_box"},
                        {"action": "type", "target": "search_box", "value": "{query}"},
                        {"action": "press", "key": "Enter"},
                    ],
                }
            ]
        )
    )

    result = await harness.run_task(
        task="Search for 'machine learning engineers' on LinkedIn and extract the first 5 results",
    )
    print(result)
    await harness.close()

asyncio.run(main())
```

### With Browser Use Cloud

```python
import asyncio
from browser_harness_wrap import BrowserHarness

async def main():
    # Use Browser Use Cloud (no local Chrome needed)
    harness = BrowserHarness(
        api_key="your-cloud-api-key",  # From cloud.browser-use.com
    )

    result = await harness.run_task(
        task="Order a large pepperoni pizza from Domino's website",
    )
    print(result)
    await harness.close()

asyncio.run(main())
```

## TypeScript Demo

```typescript
import { BrowserHarness } from 'browser-harness-wrap';

async function main() {
  const harness = new BrowserHarness({
    chromeUrl: 'chrome://inspect/#devices',
    remoteDebuggingPort: 9222,
  });

  await harness.connect();

  const result = await harness.runTask({
    task: 'Search GitHub for "openai/chatgpt-retrieval-plugin" and star the repository',
    model: 'claude',
  });

  console.log('Result:', result);
  await harness.close();
}

main().catch(console.error);
```

### Registering Custom Skills (TypeScript)

```typescript
import { BrowserHarness, DomainSkill } from 'browser-harness-wrap';

async function main() {
  const harness = new BrowserHarness({
    chromeUrl: 'chrome://inspect/#devices',
    remoteDebuggingPort: 9222,
  });

  await harness.connect();

  await harness.registerSkill({
    domain: 'amazon.com',
    skills: [
      {
        name: 'add_to_cart',
        description: 'Add a product to Amazon cart',
        selectors: {
          addButton: '#add-to-cart-button',
          quantity: '#quantity-select-1',
        },
        flow: [
          { action: 'click', target: 'quantity' },
          { action: 'select', target: 'quantity', value: '2' },
          { action: 'click', target: 'addButton' },
        ],
      },
    ],
  });

  const result = await harness.runTask({
    task: 'Add3 units of "Sony WH-1000XM5" headphones to cart',
  });

  console.log(result);
  await harness.close();
}

main().catch(console.error);
```

## How It Works

1. **Connect** — Establish CDP connection to Chrome (local or Browser Use Cloud)
2. **Task** — The LLM writes `agent_help.py` describing what it wants to do
3. **Harness** — Monitors execution and self-corrects when the agent makes mistakes
4. **Learn** — Domain skills store what works per site for future runs

## Original Project

- **GitHub:** https://github.com/browser-use/browser-harness
- **Docs:** https://docs.browser-harness.com
- **License:** MIT

## License

MIT License — see original project for details.