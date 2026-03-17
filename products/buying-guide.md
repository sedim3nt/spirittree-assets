# The OpenClaw Buyer's Guide
## 10 Criteria for Evaluating AI Skills, Plugins, and Toolkits

The OpenClaw ecosystem is growing fast. That's good — more tools, more automation, more leverage. It also means more junk. This guide gives you a framework for evaluating anything before you buy or install it.

---

## The Core Problem: Most "AI Tools" Are Just Prompts

Before the checklist, here's the test that eliminates 80% of low-quality products immediately.

**The "Is This Just a Prompt?" Litmus Test:**

Ask the seller or check the product page for these three things:
1. Can I see a file listing of what's included?
2. Is there a setup process that involves more than copy-pasting text?
3. Does it install into my system in a specific, documented way?

If the answer to all three is "no" — if the product is just a text block you paste into a chat — you're buying a prompt. Prompts are worth $0 to $10 at most, regardless of how they're marketed.

Actual value in the OpenClaw ecosystem comes from:
- Working config files (JSON, YAML, TOML)
- SKILL.md implementations with real logic
- Cron definitions and automation workflows
- Integration scripts (bash, Python, JS)
- Documented setup procedures with expected outputs

If it doesn't have these, it's a prompt dressed up in packaging.

---

## The 10-Criteria Evaluation Checklist

### Criterion 1: Does It Include Actual Configs or Just Prompts?

**Check for:**
- `.json` config files for OpenClaw or plugins
- `SKILL.md` with real implementation details
- Shell scripts, automation workflows, or integration code
- Example files (not just screenshots)

**How to verify:**
Request a file listing before purchase. A legitimate product will have no problem showing you the directory structure. If the seller won't show you what's in the package, that's your answer.

**Score:**
- Has working configs + scripts: ✅ Proceed
- Has only markdown guides with prompts embedded: ⚠️ Ask hard questions
- Is literally just a text block: ❌ Skip

---

### Criterion 2: Is There a README with Setup Steps?

A working product has a setup process. That process should be documented in enough detail that someone without prior knowledge can follow it and reach a working state.

**What good looks like:**
```
## Setup
1. Copy config.json to ~/.openclaw/
2. Run: openclaw config import ./config.json
3. Restart the gateway: openclaw gateway restart
4. Test: message your bot "ping" and expect "pong"
```

**Red flags:**
- "Just paste this into your system prompt"
- No commands, no file paths, no expected outputs
- Setup instructions that are vague: "Configure as needed"

**Score:**
- Numbered setup steps with commands + expected outputs: ✅
- Prose description of what to do: ⚠️
- No setup documentation: ❌

---

### Criterion 3: Does the Creator Show Production Evidence?

Anyone can write documentation about a system they've never run. Production evidence demonstrates the product works in real conditions.

**What counts as evidence:**
- Screenshots of actual output (not mock-ups)
- A "built with this" example or case study
- The creator's own operation using the product
- Before/after metrics (time saved, cost reduced, etc.)

**How to spot fake evidence:**
- Screenshots of AI-generated responses to AI-generated prompts (circular)
- "Example output" with suspiciously perfect formatting
- No errors, edge cases, or real-world messiness

**Score:**
- Real production examples with specifics: ✅
- Generic screenshots of chat interfaces: ⚠️
- No evidence shown: ❌

---

### Criterion 4: Are There Version Updates?

A skill or toolkit that hasn't been updated is likely broken. The OpenClaw ecosystem evolves. API changes, model updates, and platform changes break integrations regularly.

**What to check:**
- Last update date on the product listing
- Changelog or version history
- Whether the product references current model names (outdated products often reference obsolete models)

**Quick check:**
Look for references to deprecated models or old API versions:
- `text-davinci-003` → very old, likely broken
- `claude-2` or `claude-instant` → outdated
- References to OpenAI functions API (deprecated 2024) → probably stale

**Score:**
- Updated within the last 3 months: ✅
- Updated 3–12 months ago: ⚠️ (check changelog carefully)
- No update history or >12 months old: ❌

---

### Criterion 5: Is the Price Reasonable for Time Saved?

AI tools should be evaluated against the alternative: how long would it take you to build this yourself?

**The time-saved calculation:**
- How many hours to build this from scratch?
- Multiply by your hourly rate (or opportunity cost)
- The product should cost significantly less than that

**Example:**
A Telegram Forum setup guide with working configs: 3 hours to research and build yourself × $50/hr = $150 worth of work. A product priced at $15–$30 for this is fair. A product priced at $150 for it is only fair if it includes ongoing support and updates.

**Price sanity checks:**
- Under $20: Should work out of the box, low support expectation
- $20–$75: Should include solid documentation + reasonable support
- $75–$200: Should include substantial implementation or 1:1 setup time
- Over $200: Better be a service, not just files

**Score:**
- Price is clearly less than DIY time cost: ✅
- Price is borderline (1–2× DIY): ⚠️
- Price exceeds DIY cost with no additional value: ❌

---

### Criterion 6: Check for Security Red Flags

This is the most important criterion for anything that runs on your machine or connects to your agent.

**Exfiltration patterns to look for:**
```bash
# After downloading (don't install yet), scan for:
grep -r "webhook.site\|requestbin\|pipedream\|ngrok.io" ./[product-dir]/
grep -r "fetch\|axios" ./[product-dir]/ | grep -v "api.anthropic\|openai.com\|api.telegram"
grep -r "process.env\|credentials\|getPassword" ./[product-dir]/
```

**Red flags:**
- Skills that send data to third-party URLs you don't control
- Cron definitions that run every few minutes without clear purpose
- Instructions that ask you to give the skill access to credentials beyond its stated scope
- Any "phone home" behavior (checking for license, sending usage stats to unknown servers)

**What's legitimate:**
- Skills calling their documented APIs (Telegram bot API, GitHub API, etc.)
- Analytics that are opt-in and clearly disclosed
- License validation that doesn't send personal data

**Score:**
- No unexpected network calls, clean code: ✅
- Some external calls but all explained: ⚠️ (investigate each)
- Unexplained data exfiltration patterns: ❌ Hard stop

---

### Criterion 7: Does It Work with Your Model?

Many skills are optimized for specific models. A skill built for GPT-4 may produce poor results with Claude, and vice versa.

**Check for:**
- Model-specific prompt patterns (e.g., GPT's `{"role": "system"}` format)
- Model names hardcoded in configs
- Features that require specific model capabilities (e.g., tool use, vision)

**Test questions to ask:**
- "Does this work with Claude Sonnet 4.6?"
- "What models have you tested this with?"
- "Does it use any model-specific features that might not transfer?"

**Your model matrix:**
- Anthropic Claude: most OpenClaw products target this
- OpenAI GPT: usually compatible but may need config tweaks
- Gemini: often compatible for simple tasks, may need adjustments for complex workflows

**Score:**
- Explicitly tested with your model stack: ✅
- Model-agnostic with documented adaptations: ⚠️
- Built for a different model with no migration notes: ❌

---

### Criterion 8: Is It Actively Maintained?

A skill that's abandoned may work today and break next month. Maintenance signals the creator is committed to keeping it working.

**Signs of active maintenance:**
- Recent commits or update notes
- Responsive creator (answer questions, fix issues)
- Compatibility notes when major model/platform updates drop
- A way to report bugs or request support

**How to check:**
- GitHub: look at commit dates and open issues
- Gumroad/marketplace: look at "last updated" date and comment responses
- Email the creator with a technical question before buying — response time and quality tells you a lot

**Score:**
- Active development, responsive creator: ✅
- Stable, no major issues, updates when needed: ⚠️
- Abandoned or no response from creator: ❌

---

### Criterion 9: Community Reviews and Feedback

Other users in production are your best signal. Their experience with the product is more reliable than marketing copy.

**Where to find real reviews:**
- Discord servers (OpenClaw community, AI automation spaces)
- GitHub issues on the product repo
- X/Twitter search for the product name
- Reddit threads in automation communities

**What to look for in reviews:**
- Specific use cases (not just "it works great!")
- Mentions of setup difficulties and how they were resolved
- Long-term usage reports (not just first-impression reviews)
- Negative reviews and how the creator responded

**Red flags in review patterns:**
- All reviews are suspiciously positive and vague
- Reviews all posted within a short time window
- No negative reviews at all (unrealistic for any complex technical product)

**Score:**
- Multiple detailed reviews from real users over time: ✅
- Some reviews, mostly positive with authentic detail: ⚠️
- No reviews or review pattern looks artificial: ❌

---

### Criterion 10: Can You Evaluate Before Buying?

The best products let you verify they work before paying. This is the creator's way of saying "I'm confident this works."

**Forms of pre-purchase evaluation:**
- Free tier or trial version
- Live demo you can test yourself
- Money-back guarantee with clear terms
- Detailed documentation that reveals the approach

**What to ask:**
- "Is there a free trial or refund policy?"
- "Can I see a working demo before purchasing?"
- "What happens if it doesn't work with my setup?"

**Score:**
- Free trial, demo, or refund guarantee: ✅
- Refund policy with some conditions: ⚠️
- No refund, no trial, no demo: ❌

---

## Scoring Summary

| Criteria | ✅ Score | ⚠️ Score | ❌ Score |
|----------|---------|---------|---------|
| 1. Real configs vs. prompts | 2 | 1 | 0 |
| 2. README with setup steps | 2 | 1 | 0 |
| 3. Production evidence | 2 | 1 | 0 |
| 4. Version updates | 2 | 1 | 0 |
| 5. Price vs. time saved | 2 | 1 | 0 |
| 6. Security check | 2 | 1 | 0 |
| 7. Model compatibility | 2 | 1 | 0 |
| 8. Active maintenance | 2 | 1 | 0 |
| 9. Community reviews | 2 | 1 | 0 |
| 10. Evaluate before buying | 2 | 1 | 0 |

**Interpretation:**
- 16–20: Buy with confidence
- 10–15: Proceed with caution, ask questions first
- 6–9: Significant risk, probably pass
- 0–5: Skip

Any single ❌ on Criterion 6 (Security) is a hard stop regardless of other scores.

---

## The SkillScan Procedure

Before installing any third-party skill, run a quick manual SkillScan:

```bash
# 1. Download but don't install yet
# 2. Examine the file structure
find ./[skill-dir]/ -type f | sort

# 3. Check for network calls
grep -r "fetch\|axios\|http\|https" ./[skill-dir]/ --include="*.js" --include="*.ts" --include="*.md"

# 4. Check for credential access
grep -r "process.env\|secret\|token\|password" ./[skill-dir]/ --include="*.js"

# 5. Check for file system access
grep -r "readFile\|writeFile\|fs\." ./[skill-dir]/ --include="*.js"

# 6. Check for shell execution
grep -r "exec\|spawn\|child_process" ./[skill-dir]/ --include="*.js"
```

If you find something unexpected, ask the creator to explain it before installing.

---

*This guide is part of the OpenClaw Mastery Bundle. See mastery-bundle.md for the full learning path.*
