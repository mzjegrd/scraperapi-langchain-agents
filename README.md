# web scraping agent langchain: How to Give Your Agent Live Web Access with ScraperAPI — Setup, the Three Official Tools, and What It Really Costs per Request

A LangChain agent can't open a URL on its own. When you ask it to "check today's top story on Hacker News" or "find wireless headphones under $100 on Amazon," the model has to delegate that fetch to a tool, and that tool needs to survive CAPTCHAs, IP blocks, and JavaScript-heavy pages. The shortest route for most developers: ScraperAPI's official `langchain-scraperapi` package, which ships three ready-made tools (web page fetching, structured Google SERP data, structured Amazon product search) and handles the anti-bot infrastructure behind the scenes.

This guide covers the actual setup, the full current plan lineup, and — the part most tutorials skip — what an agent-driven workload actually costs once you know that Google searches burn 25 credits and a standard page burns 1.

> **Quick pick:** Start with the free tier (1,000 credits/month, 5 concurrent connections) or the 7-day trial with 5,000 credits — no credit card required. For a running agent, Hobby at **$49/month** (100,000 credits, 20 concurrent threads) covers most solo projects. 👉 [Start the 7-day free trial with 5,000 credits](https://www.scraperapi.com/signup?fp_ref=coupons)

## What a LangChain scraping agent actually needs

Three things, in order of how often they bite you: a tool interface the LLM can call reliably, protection against getting blocked, and predictable billing. The official integration covers all three, but the billing part deserves more attention than it usually gets — more on that in the credit math section below.

The package is `langchain-scraperapi`, published by ScraperAPI itself. Its pitch is accurate rather than inflated: install it, set your API key, and your agent can browse websites, search Google, and pull Amazon product data. The three tool classes map to three real-world agent jobs:

| Tool class | What it does for your agent |
| --- | --- |
| `ScraperAPITool` | Fetches any public page as HTML, text, or markdown |
| `ScraperAPIGoogleSearchTool` | Returns structured Google SERP data (JSON or CSV) |
| `ScraperAPIAmazonSearchTool` | Returns structured Amazon product-search data |

That covers the two patterns almost every web-aware agent uses: search for something, then read the page you land on. 👉 [See ScraperAPI's full LangChain integration tutorial](https://www.scraperapi.com/integration-tutorials/langchain?fp_ref=coupons) for the official walkthrough with multi-tool agent examples.

## Setup: from pip install to a working agent

Four steps. No proxy configuration, no CAPTCHA-solving service to wire up separately.

**1. Install the packages:**

bash
pip install -U langchain_scraperapi openai langchain_openai langchain


You'll need Python 3.7 or newer, plus an OpenAI key (the integration also supports Anthropic's Claude and Cohere via their respective environment variables — `ANTHROPIC_API_KEY` and `COHERE_API_KEY`).

**2. Get your ScraperAPI key.** Sign up, grab the key from your dashboard, and export both variables:

bash
export SCRAPERAPI_API_KEY="your_key_here"
export OPENAI_API_KEY="your_openai_key"


One note from the official docs: if you use OpenAI as the LLM, OpenAI may require billing info on your account even while you're on ScraperAPI's free trial. That's OpenAI's requirement, not ScraperAPI's — worth knowing before it surprises you mid-setup.

**3. Wire up a single tool.** The minimal direct invocation looks like this:

python
from langchain_scraperapi import ScraperAPITool

tool = ScraperAPITool()
result = tool.invoke({
    "url": "https://example.com",
    "output_format": "markdown",
    "country_code": "US",
    "render": True,   # run JavaScript before returning content
    "wait": 5000      # wait 5 seconds for the page to load
})


The parameter set on `ScraperAPITool` is worth knowing at a glance: `url` (required), `output_format` (`"text"` or `"markdown"`, with plain HTML as the default), `country_code`, `device_type` (`desktop`/`mobile`), `premium` (premium proxies), `render`, and `keep_headers`. For markdown-hungry LLM pipelines, `output_format="markdown"` is usually what you want.

**4. Hand the tool to an agent.** The official repo example uses `gpt-4o` with `create_tool_calling_agent`:

python
from langchain_openai import ChatOpenAI
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_scraperapi.tools import ScraperAPITool

tools = [ScraperAPITool()]
llm = ChatOpenAI(model_name="gpt-4o", temperature=0)

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant that can browse websites. Use ScraperAPITool to access web content."),
    ("human", "{input}"),
    MessagesPlaceholder(variable_name="agent_scratchpad"),
])

agent = create_tool_calling_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
response = agent_executor.invoke({"input": "Browse hackernews and summarize the top story"})


Or, if you prefer the older `initialize_agent` style, the tutorial's multi-tool version loads all three tools and runs queries like "Search Google for 'Python web scraping' and give me the top 3 results" and "Find wireless headphones on Amazon under $100" in one agent.

The Google and Amazon tools take their own modifiers: `ScraperAPIGoogleSearchTool` accepts `query` plus `country_code`, `tld`, `num`, `hl`, and `gl`; the Amazon tool accepts `query`, `country_code`, `tld`, and `page` for pagination.

## The full plan lineup (current official pricing)

Here's everything currently listed on ScraperAPI's pricing page, with annual billing folded in — annual payment takes a flat 10% off every tier:

| Plan | Monthly price | With annual billing | Credits/mo | Concurrent threads | Geotargeting | Get it |
| --- | --- | --- | --- | --- | --- | --- |
| Hobby | $49 | $44.10/mo | 100,000 | 20 | US & EU | [Start with Hobby](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Startup | $149 | $134.10/mo | 1,000,000 | 50 | US & EU | [Start with Startup](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Business | $299 | $269.10/mo | 3,000,000 | 100 | Country-level | [Start with Business](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Scaling | $475 | $427.50/mo | 5,000,000 | 200 | Country-level | [Start with Scaling](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Professional | $975 | $877.50/mo | 10,500,000 | 300 | Country-level | [Start with Professional](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50/mo | 21,500,000 | 500 | Country-level | [Start with Advanced](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | Country-level | [Talk to ScraperAPI sales](https://www.scraperapi.com/contact-sales?fp_ref=coupons) |

Before the free trial and free plan: signing up gets you 1,000 free API credits per month with up to 5 concurrent connections, and the first 7 days come with 5,000 credits for larger-scale testing — no credit card required for the trial. ScraperAPI's docs also mention a Student monthly plan, though no public price is listed for it on the pricing page.

Every paid plan includes the same core feature set: JavaScript rendering, premium residential and mobile IPs, CAPTCHA and anti-bot handling, JSON auto-parsing, custom headers and sessions, desktop/mobile user agents, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee.

There's also a 7-day, no-questions-asked refund policy if the service doesn't work for you, and you can cancel anytime from the dashboard without being charged. That makes the first month fairly low-risk even past the trial window.

## What your agent really costs: the credit math

This is the part that catches people off guard. One credit buys one standard page — but your LangChain agent probably isn't scraping standard pages.

ScraperAPI's published costs per request:

- **Standard page:** 1 credit
- **Amazon:** 5 credits
- **Google and Bing** (including subdomains): 25 credits
- **LinkedIn:** 30 credits
- **Sites behind Cloudflare, Datadome, or PerimeterX:** +10 credits per request when the protection is bypassed

Now plug in agent behavior. An agent that researches a topic typically runs a Google search, then fetches two or three results. One search-plus-read cycle costs 25 + 2 = 27 credits, roughly. A Hobby plan's 100,000 credits covers about 3,700 such cycles in a month — fine for a personal project, thin for a cron job running hourly.

Two built-in tools help you keep this under control: the Domain Cost Estimator in your dashboard shows the exact credit cost for any URL before you scrape it at scale, and the `max_cost` parameter caps what any single request can spend. If your agent hits a lot of protected targets, that second one is worth setting from day one.

The other cost lever is billing frequency. Annual billing saves 10% across every plan, so Hobby works out to $44.10/month paid annually versus $49 monthly — about $58.80 saved over the year.

## How ScraperAPI stacks up for LangChain agents

A few alternatives come up constantly in this space, and the honest answer is that which one fits depends on your targets and budget. All figures below are from third-party comparisons published in 2026, so treat them as directional rather than exact.

- **Firecrawl** is built around LLM-friendly output (clean markdown from crawled pages) and starts cheaper — around $16/month for 45,000 credits, with a free tier of a few thousand credits per month. It's a reasonable fit when your agent mostly needs to ingest whole pages and you don't need deep structured SERP or Amazon endpoints.
- **ZenRows** starts around $69/month and leans on a strong anti-bot reputation; its free allocation is a one-time grant of about 5,000 requests rather than a recurring monthly allowance.
- On estimated cost per thousand pages, third-party measurements put ScraperAPI around $1.10, ZenRows around $1.40, Apify around $1.00, and Firecrawl highest at roughly $3.20 — though these figures swing heavily based on which domains you're hitting and whether protection bypass is involved.

Where ScraperAPI has a concrete edge for this specific use case: the LangChain integration is official and maintained by the company itself, with structured Google and Amazon endpoints designed for exactly the search-then-extract pattern agents use. If your workload is mostly plain-page ingestion and budget is the top constraint, Firecrawl's cheaper entry tier is genuinely worth considering; if you need SERP data and marketplace data as first-class structured outputs, ScraperAPI's tool set saves you the parsing work.

## Limits and gotchas worth knowing before your agent goes live

Concurrency, not credits, is often the first wall agents hit. When a request comes in above your plan's concurrent limit, ScraperAPI returns a 429 and asks you to retry after the count drops — the request is rejected, not queued, and you aren't charged for it. For agents that fire parallel tool calls, this means either pacing your agent's requests or sizing the plan to your parallelism. The official docs explicitly flag this: make sure your plan's concurrency supports what the agent might attempt, and set sensible timeouts in your LangChain configuration.

Geotargeting has a hard split. Hobby and Startup can only target the US and EU. Country-level geotargeting anywhere in the world starts at the $299 Business plan. If your agent needs to see region-specific prices or search results from outside the US/EU, the cheaper two tiers won't do it.

Analytics retention also changes by tier — Hobby and Startup keep analytics for 30 days; Business and up keep them indefinitely.

When credits run out, scraping pauses by default until renewal, rather than silently billing you. You have opt-in ways to keep going: Auto-Upgrade (available on Hobby through Professional) automatically moves you to the next tier, which usually has a better price per credit; Pay-As-You-Go (Scaling and up) lets you continue at a fixed per-credit rate with a monthly spending cap you set — ScraperAPI is explicit that the cap is a spending limit, not a pre-charge. If you'd rather your agent just stop than spend more, do nothing; the default behavior already does that.

## FAQ

**Does the integration work with Claude or only OpenAI?**
Both. The official examples use OpenAI's `ChatOpenAI`, and the tutorial documents Anthropic and Cohere as supported LLM providers through their own API key environment variables.

**Can I test it without paying?**
Yes — 1,000 free credits per month on the free plan (5 concurrent connections max), plus a 7-day trial with 5,000 credits at signup, no credit card required. If you need more testing credits, ScraperAPI support can grant additional ones on request.

**What happens if my agent burns through credits mid-month?**
Scraping pauses until your credits renew, unless you've enabled Auto-Upgrade or Pay-As-You-Go. Nothing gets billed automatically by default.

**Can I get a refund if it doesn't work for my use case?**
ScraperAPI offers a 7-day no-questions-asked refund — contact support and they process it. Subscriptions can also be cancelled anytime from the dashboard without further charges.

## Which plan makes sense for which reader

A practical decision rule based on the verified numbers:

- **Just prototyping an agent**: stay on the free tier. 1,000 credits a month is enough to validate that your tool calls work, even if it only covers a handful of SERP-heavy runs.
- **A personal agent running weekly jobs**: Hobby at $49/month (or $44.10 with annual billing) gives 100,000 credits — roughly 3,700 search-then-read cycles, or far more if your agent sticks to standard pages.
- **Production workloads with real traffic**: Startup ($149) or Business ($299), depending on whether you need global geotargeting and 100 concurrent threads.
- **Multi-agent systems or cron pipelines running constantly**: Scaling and above, where Pay-As-You-Go with a spending cap gives you a predictable overflow option.

Two things to check before committing to anything beyond Hobby: confirm your target domains with the Domain Cost Estimator so you know their real credit price, and count your agent's peak parallel requests against the concurrency column above. If those two numbers fit a tier, that's your plan. 👉 [Check current plan pricing and start your free trial](https://www.scraperapi.com/pricing?fp_ref=coupons)
