---
layout: post
title: "Custom investment themes. Part I: YOLO LLM tagging"
date: 2026-02-15
---

## Themes ≠ standard classifications

I ran into a problem today trying to implement an aggressive momentum strategy to trade thematic stocks.

The default Standard Industrial Classification (SIC) or Global Industry Classification Standard (GICS) are too broad. They don't allow you to narrow down to a specific set of companies that produce a specific product, enabling queries in our platform such as "*Give me top 50 robotics companies ranked by momentum score*".

The classifications use traditional industry verticals, which are mutually exclusive and very outdated. 

For example:
| Ticker | SEC Industry | GICS | My themes |
|--------|--------------|------|-----------|
| TSLA | Automobiles | Consumer Discretionary / Automobiles & Components / Automobile Manufacturers | robotics, autonomous-vehicles |
| ISRG | Health Care Equipment | Health Care / Health Care Equipment & Supplies / Surgical Equipment | robotics, medical-devices |

Both TSLA and ISRG have key operations in the robotics space, but they are classified in completely different industries, typically the **main current revenue driver** for the company. 

Unlike GICS or SIC, our investment themes are flexible and constantly changing depending on our investment focus.
We may want to track the following themes:
- **AI native**: encompasses companies from different industries that have built their products and services around AI from the ground up, thus having an advantage over incumbents.
- **defense**, **space-technology**: due to geopolitical tension we want to track companies that are involved in BOTH defense and space technology sector.
- **cybersecurity**: companies that provide products and services to protect against exponential threat of AI powered cyber attacks.


## Batch tagging, the YOLO way

Our goal is to tag 3,000+ companies that match a theme once a month, or upon ingesting our universe of stocks in the platform.

Not being in the mood to manually tag a training set of 100+ companies every time a new theme is added — good luck finding 100 space technology companies — I decided this was a perfect use case for what I call **YOLO LLM**: submit the raw data with a basic prompt and let the AI serf do the work. We pass the business section of each SEC filing and rely on the LLM's ability to pick up semantic meaning and tag companies accordingly. Exactly what an LLM should be good at.

## Workflow vs. cost engineering

For non-core features, the goal is the quickest path to acceptable results without overengineering code I'll have to maintain. This is especially true with any AI powered workflow. Time and time again we've seen in-house solutions engineered to workaround the LLM limitations be later absorbed by the LLM API and rendered completely obsolete: prompt chaining, output parsing, RAG stitching, tool calls.

Unintuitively, extracting meaning from a ~8,000 word SEC business section for 3,000 companies seems like it should be straightforward with a Batch LLM API, but in reality it isn't due to how pricing is designed. The problem is not so much about the complexity of the task, but how you manage input tokens to achieve acceptable costs. 

The moment you use an LLM, you're not just building a workflow — you're negotiating a cost structure, and every token is a line item. While per-token pricing should decrease over time with improved hardware and increased competition, this is by no means a guarantee. LLM pricing is artificially subsidised by VC capital in a land-grab phase.

Same logic applies to open-source models — the per-token bill just shifts to hardware or cloud compute.

## Labelling Prompt

Here is the prompt refined over several runs, producing very satisfying clean predictions.

```
# Company Theme Classification Task

You are an expert analyst tasked with classifying companies into investment themes based on their business descriptions from SEC 10-K filings.

## Important Classification Principles

1. **Multiple themes allowed**: A company can belong to 0, 1, or multiple themes
2. **Evidence-based**: Only tag if there's clear evidence in the business description
3. **No external knowledge**: Only use information explicitly stated in the provided business description. Do not rely on your training knowledge about the company, its products, or its industry
4. **Conservative approach**: When in doubt, don't tag. Better to miss borderline cases than over-tag
5. **Focus on actual business**: Look at what the company actually does and focuses on, not future plans or aspirational statements
6. **Check exclusions**: For each theme you consider tagging, verify the company doesn't match any exclusion listed under that specific theme
7. **Independent classification**: Classify each company independently. One company's result must not influence another's.
8. **Order by relevance**: List themes in order of strongest fit first, based on evidence strength in the business description

## Theme Definitions

{THEMES_CONFIG}

## Output Format

{OUTPUT_EXAMPLE}
```


To diagnose any misclassifications, we add a diagnostic section in the response model:

```
if include_reasoning:
    example = ThemesResultWithReasoning(
        themes=["theme-id-1", "theme-id-2"],
        reasoning={
            "theme-id-1": "Brief explanation of why this theme applies",
            "theme-id-2": "Brief explanation of why this theme applies",
        },
        considered_but_rejected={
            "theme-id-3": "Brief explanation of why this theme does NOT apply"
        },
    )
```

Zero-shot labelling works because each theme ships with its own instructions. Adding a new theme is just a JSON entry.

```
{
        "id": "robotics",
        "display_name": "Robotics",
        "description": "Companies that design, manufacture, and deploy autonomous or semi-autonomous physical systems capable of sensing, deciding, and acting in the real world. These systems perform work that directly alters physical environments through movement, manipulation, or force, replacing or augmenting human labour.",
        "key_indicators": [
          "Builds and deploys physical robotic systems that interact with the real world",
          "Robots can autonomously or semi-autonomously move, manipulate objects, or apply force",
          "Core value comes from real-world task execution, not simulation or decision support",
          "Hardware, control systems, and embodiment are central to the product",
          "Robotics is a primary revenue driver or long-term strategic focus"
        ],
        "keywords": [
          "robotics", "physical automation", "embodied AI", "autonomous machines",
          "robotic manipulation", "robotic mobility", "humanoid robot", "industrial robot",
          "collaborative robot", "cobot", "autonomous mobile robot", "AMR",
          "robotic arm", "warehouse robot", "field robotics"
        ],
        "exclusions": [
          "Pure software or digital automation (RPA, AI agents, optimisation tools)",
          "Simulation, digital twins, or virtual robotics without physical deployment",
          "Generic industrial machinery without autonomy or sensing",
          "Companies that merely integrate or operate robots developed by others"
        ],
        "nuances": [
          "The defining feature is physical agency: the system must change the real world",
          "Includes both fully autonomous and tightly supervised robotic systems",
          "Software qualifies only when inseparable from a physical robotic platform",
          "Embodiment creates high barriers to entry through hardware, safety, and deployment complexity",
          "Economic value scales with task replacement, reliability, and real-world uptime"
        ],
        "example_companies": ["Tesla (Optimus)", "Boston Dynamics", "ABB Robotics", "Fanuc", "KUKA", "Intuitive Surgical"]
    },
```

## Negotiating with the API

To open the negotiation with Mr. Claude, we start with the least intelligent family member that can achieve acceptable precision on raw, unaltered data. From there, we gradually trim the input to drive the cost down.

Sonnet 4 is the first model that picks up on nuances. Haiku and its cheaper cousins lack the ability to correctly tag edge-case themes like "ground-truth-data" or "ai-picks-and-shovels".

![Claude API costs by model](/assets/images/investment-themes/claude-costs.png)
*Costs as of Feb 2026*

Batch processing comes with a 50% discount, so base rate is $1.5/Mtok. At ~20,000 tokens per company (business section + prompt), that's $0.03/company — or $90 worst case for 3,000 companies, before any caching savings (see [Playing cache roulette](#playing-cache-roulette)). Let's see how much further we can squeeze it.

Here's how the negotiation rounds went. Discount indicates improvement from the best attempt so far.

| Round | Cost/company | Cache r/w | F1 | Notes |
|-------|-------------|-----------|-----|-------|
| 1 | $0.057 | - | — | Smoke test, no benchmark |
| 2 | $0.040 | 0.213 | 81.48% | Baseline |
| 3 | $0.022 | 12.000 | 85.71% | Drop extra reasoning trace in response |
| 4 | $0.033 | 0.110 | 86.54% | Cleanse and trim input — cache backfired! |
| 5 | $0.018 | 0.583 | 85.39% | Hack batching 5 companies/request |
| 6 | $0.018 | 0.010 | — | Scale to 1,000 with 10 companies/request|
| 7 | $0.018 | 0.316 | — | Change 5min to 1h cache TTL, no difference! |

**Round 1** — Smoke test with 5 companies

Not off to a great start! That being said with a batch of only 5 the cache savings are probably not kicking in yet.
It's like negotiating to buy fake Labubus from Yiwu, you gotta commit for a large order first.

> **$0.057/company** — smoke test, no benchmark

---

**Round 2** — Baseline test using labelled benchmark of ~100 companies

Nice results out of the box with precision of 81.48%. The model is able to pick up on nuances and correctly classify companies that are borderline cases. The cost is still quite high. Perhaps due to output token costs of reasoning trace in the response.

> **$0.040/company** — <span style="color:#22c55e">↓30%</span> · F1 81.48%

---

**Round 3** — Retry with reasoning stripped out

44% cost reduction with no impact on precision! The extra reasoning trace is nice to have for diagnostics, but not worth the cost for production runs.

> **$0.022/company** — <span style="color:#22c55e">↓44%</span> · F1 85.71%

---

**Round 4** — Reduce input

The SEC 10-K business section includes predictable regulatory topics which don't add any value to the classification such as ESG clauses.
Example of headers that precede the ESG boilerplate:
```
"environmental_social_governance_sustainability": [
    "environmental",
    "social",
    "governance",
    "esg",
    "sustainability",
    "climate",
],
```

Removing these boilerplates and truncating the input to 100K characters results in a combined 29% reduction of total words.
After all this data cleansing, we get a ... INCREASE in cost? Turns out this run had horrendous read/write cache ratio, so the expected savings from input tokens got eaten up by additional write costs.

> **$0.033/company** — <span style="color:#ef4444">↑+50% (cache backfired!)</span> · F1 86.54%

---

**Round 5** — Hack the prompt to pack 5 companies per request

To reduce the number of requests and increase the chances of hitting the cache, we jam 5 companies in the same request. All it takes is adding the following line:
```
You will receive multiple companies to classify. Return results for ALL companies keyed by their ticker symbol.
```
Surprisingly this works well — a further 20% discount, much better cache r/w ratio, and barely any impact on precision.

> **$0.018/company** — <span style="color:#22c55e">↓20%</span> · F1 85.39%

---

**Round 6** — Scale up to 1,000 companies

We now pack 10 companies per request to check if the cost reduction continues to hold. The cost remains stable at $0.018/company, even with a poor cache hit rate.

> **$0.018/company** — flat

---

**Round 7** — Another set of 1,000 companies

Increase the cache TTL from 5 mins to 1h to see if it reduced costs. No such luck.

> **$0.018/company** — flat

...

At this point I'd rather buy my kids an ice cream than keep subsidising Anthropic. Any further gains would be marginal and require more data massaging than it's worth.

$0.018/company isn't cheap at scale, but getting there took less than a day and zero labelled training data. For something that would have taken weeks with a classical classifier, that's a decent trade.


I realise this experiment provided me with the labelled training data I need, so I will do a part II to see how a fined tune open model performs on the same task.

## Playing cache roulette

When submitting requests in the batch API, we dumbly repeat the instructions over and over. Example

Request 1
```
You are an expert in X. Classify the info below <-- static instructions
{info for company ABC} <-- changes every request
```

Request 2
```
You are an expert in X. Classify the info below
{info for company XYZ} <-- only change
```

We can ask to cache the instructions by adding a flag in the request. Subsequent requests are able to read them from cache. A cache read costs only 10% of the input token price.

Straightforward? Not quite.

WRITING to the cache incurs a cost of 1.25x for 5 mins caching, and 2x for 1h caching. Anthropic processes requests across parallel workers, and you have zero control on the number of workers or the time it takes to process each request. Tests shows wild swings in cost, with a read/write ratio as low as 0.01 to as high as 12.0. Enabling the 'cache ttl" flag is akin to entering a lucky draw.































