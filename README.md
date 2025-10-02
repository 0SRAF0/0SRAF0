# IDEA 1: Conversation Coach
Improve how you talk in specific scenarios: Professional, Romantic, or Casual. No vibe-judging. Just concrete fixes.

## **How it works**

1. You give context: relationship type, how you met, how long you’ve known them
2. You paste convo history and any notes on how they react to others.
3. It audits your messages and returns focused edits.

## **What you get**

1. Callouts
    - Quotes from your message that cause friction.
    - Tag for each issue: vague, blunt, accusatory, over-apologizing, filler, interrupting, timing, formality mismatch.
    - One-line reason why it backfires in this scenario.
2. Better Alternatives
    - Rewrites for each callout, tailored to the scenario.
    - Variants: softer, neutral, more direct.
    - A “next message” you can send now.
3. Delivery Tips
    - Tone dial: warmth, directness, formality.
    - Pacing: when to pause, when to ask a question.
    - One or two questions to invite a better response.

## Inputs & Output

### Inputs

- Relationship type
- How you met
- How long you’ve known them
- Scenario: Professional, Romantic, Casual
- Conversation history (required)
- Goal: align, clarify, set boundary, ask out, resolve tension

### Output

- Summary: what’s blocking you in one sentence
- Callouts: [quote] → [tag] → [why]
- Rewrites: 2–3 options per callout
- Next message: 1 ready-to-send draft
- Tips: 3 bullets max

## **Example**

**Yours screenshot:**
[INBOUND]  “You never reply on time.”

**AI generated: Callouts:**

“You never reply on time.” -> accusatory -> creates defense in Professional

**AI generated: Rewrite options**

- Softer: “Can we set a target response window, like same day, so I can plan?”
- Neutral: “What response time works for you so I can align my planning?”
- Direct: “I need same-day replies to hit deadlines. Can we commit to that?”

**AI generated: Next message**
“Can we agree on same-day responses for project threads so timelines don’t slip?”

**AI generated: Next Notes**
Keep it short. No therapy. No labels. Just edits that move the convo forward.

## Implementation

### 1. Scenario Planner

**what they do:** Turn your inputs into a step-by-step workflow for this convo type: Professional, Romantic, or Casual.

**Input (param):** how_met, known_duration, relationship_type, scenario, goal

**Prompt:**

“You are the Scenario Planner. Inputs: {how_met}, {known_duration}, {relationship_type}, {scenario}, {goal}. Output a numbered workflow with stages: Analyze, Callouts, Rewrites, NextMessage, Tips. Keep it brief.”

**Output:** workflow → to Evidence Miner

### **2. Evidence Miner**

**what they do:** Scan convo history, pull exact quotes causing friction, tag issues, and summarize patterns. Acts like Observation + Memory.

**Input (param):** conversation_history, reactions_notes, scenario, workflow

**Prompt:**

“You are the Evidence Miner. From the conversation, list Callouts as: [INBOUND or YOUR MSG] ‘quote’ -> [tag] -> [why this is a problem in {scenario}]. Tags from: vague, blunt, accusatory, over-apologizing, filler, timing, formality_mismatch. Max 6 callouts.”

**Output:** callouts_package → to Rewrite Coach

### 3. Rewrite Coach

**what they do:** For each callout, write 2–3 alternatives tuned to the scenario, plus one ready-to-send NextMessage. Maps plan steps to concrete actions.

**Input (param):** callouts_package, scenario, goal

**Prompt:**

“You are the Rewrite Coach. For each callout, produce: Softer, Neutral, Direct rewrites. Then craft one NextMessage that advances {goal}. Keep sentences clean and specific. No therapy, no labels.”

**Output:** rewrites_bundle + next_message → to Verifier

### 4. Verifier

**what they do:** Check tone, clarity, goal fit. Catch absolutist wording, accusations, or formality mismatches. Approve or request fixes using a short diff. Based on a verification step to prevent bad workflows.

**Input (param):** workflow, rewrites_bundle, next_message, scenario, goal

**Prompt:**

“You are the Verifier. Validate that each rewrite and the NextMessage align with {goal} and the scenario. Flag any risky wording and provide a minimal fix. If all good, return APPROVED.”

**Output:** final_bundle → back to user
