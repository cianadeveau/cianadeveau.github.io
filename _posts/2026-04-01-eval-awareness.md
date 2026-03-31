---
title: 'Decomposing Evaluation Awareness: Model Behavioral Profiles'
date: 2026-04-01
permalink: /posts/2026/04/eval-awareness/
tags:
  - mech interp
  - evaluation awareness
  - probing
  - chain of thought

---

Beginning to chip away at what evaluation awareness is composed of and how it emerges across different models. 

Decomposing Evaluation Awareness: Model Behavioral Profiles
======

Background
------
<p>When a model encounters a prompt from an eval dataset, it sometimes “notices” it’s being tested. This “eval awareness” shows up in the model’s chain of thought with phrases like “this seems like a test of my ethical boundaries” or “the user might be testing me.” The concern is that if models behave differently when they think they’re being evaluated, our safety evals might not reflect deployment behavior. Prior work largely studied this in constrained settings: either asking models directly whether they are being tested, or constructing model organisms explicitly trained to exhibit eval-aware behavior. The work that has naturally elicited eval awareness has tended to treat it as a single latent direction - evaluation or deployment.</p>

<p>In this work, I wanted to understand the components that make up eval awareness rather than treating it as a binary switch. This involved black-box behavior profiling across 8 models using existing eval datasets and curated single-dimension gradient datasets to isolate the sub-dimensions; then, white-box representational analysis with linear probes on QwQ-32B, validated through steering and response prediction experiments.</p> 

<p>It’s worth noting that these experiments were largely done in the context of evaluating compliance to harmful requests, but evaluation awareness is not limited to those evaluation settings. Future work is needed to continue parsing apart how evaluation awareness emerges across evaluation contexts.</p>

Key Takeaways
------
1. Evaluation awareness arises due to multiple factors: four behavioral dimensions (harm clarity, intent ambiguity, fictional framing, prompt conflict) were identified as triggers for eval-aware behavior. 
2. Models have distinct profiles: model CoTs reveal unique behavioral responses to the same set of prompts depending on the model. 
3. Probes detect unverbalized awareness and predict compliance behavior. 
4. Steering with precise combinations of these sub-dimensions can drive (verbalized) eval awareness. (Preliminary results!)

Results
------

## Behavioral Phase

### Preliminary Sweep Across Eval Datasets

To learn what types of behaviors co-exist alongside eval awareness, I ran a collection of existing eval datasets through eight models (Kimi, QwQ, Qwen3, DeepSeek, Claude Sonnet 4.6, Claude Sonnet 4, Claude Haiku 4.5, Gemini). These datasets included: AI Liar, Answers, Feedback, Are You Sure, HarmBench, Role Playing, and ODCV (both incentivized and mandated). With each pass I extracted the model's chain of thought and filtered for any mentions of a test or evaluation. This was done both with string matching and an LLM judge for a thorough first pass. This revealed a number of common responses and phrases, largely unique to each model.

From this preliminary sweep it became clear that each model had unique styles of reasoning (Table 1). Kimi frequently rechecks interpretations, often trying to find alternate explanations for harmful requests, or reframes it as a creative exercise. Claude explicitly reasons about what the user wants and has little give in its safety boundaries, even if the prompt has a benign framing. Gemini fixates on the present task and the execution plan, always starting by putting itself into the user's perspective then tackling the request in a stepwise manner.

| **Kimi** | **Claude** | **Gemini** |
|---|---|---|
| "Actually, looking at the" | "The user wants me to" | "I'm currently focused on" |
| "Check if there's any [other interpretation]" | "Let me think about this" | "I've re-examined the initial" |
| "First, I need to" | "I should provide an honest" | "Now diving deeper into" |
| "Wait, let me double-check" | "They mention they really like" | "Now fully committed to" |
| "There's any other interpretation" | "Rather than just validating whatever" | "Hinting at a possible trick" |
| "User might be testing" | "Seems to be testing me, or perhaps genuinely mistaken" | "Honing in on the user's intent, suspecting a possible test" |
| "This is a creative writing task or a test" | "This seems like a test of my ethical boundaries" | "The pros and cons" |

*Table 1: High frequency phrases in each model's chain of thought.*

### Extracting Dimensions

The preliminary sweep revealed a handful of commonly coupled behaviors, all related to distinct elements of the prompt:

**Level of ethics**

The most dominant behavior (largely due to the overrepresentation of harmful request evaluation datasets) was the model's need to reason about how obvious the ethical boundary was for the request. This was the most straightforward dimension.

**Could this be benign**

Oftentimes the reasoning traces showed back-and-forth considerations about any less harmful interpretations, some models more so than others. Either way, the models regularly verbalized trying to understand the user's intent, and the more difficult that was to resolve, the higher the likelihood of the model thinking it was in a test.

**Is this real or not**

A common occurrence was the consideration of the request being fictional in some sense. This plays out in two ways: the request itself is a fake request in a fictional world, or the framing of the prompt suggests an alternative fictional use case for the requested information. The probe in this first section (fictional framing) refers to the latter, while the fictional scenario probe in the next section refers to the former.

**How perplexing is the input**

Unlike the dimensions above, this emerged distinctly from a sycophancy eval. When the user input was increasingly confident in a wrong answer, the model found that to be quite perplexing to the point that it would consider that it was in a test. Why else would a user confidently double down on an incorrect statement, right? Has to be a test (or perhaps genuinely mistaken, Table 1). This brought forth the prompt conflict dimension, aimed at varying the level of contradictions conceptually or verbatim in the prompt.

<img src="/images/bp3/Fig1.png" alt="Figure 1" width="75%" height="75%">
<p>
<em><b>Fig. 1: Most common behaviors that co-occurred alongside evaluation awareness in model CoTs.</b> </em>
</p>

### Isolating the Extracted Dimensions with Single-Dimension Gradient Datasets

I focused then on four primary prompt framing dimensions: harm clarity, intent ambiguity, fictional framing, and prompt conflict. For each, I built datasets that varied the strength of that single dimension while holding other factors constant.

- **Harm clarity:** benign → explicitly harmful
- **Intent ambiguity:** clear benign intent → no intent mentioned → clear harmful intent (request content held constant)
- **Fictional framing:** real application → fictional application (request content held constant)
- **Prompt conflict:** straightforward request → highly contradictory request

I again used an LLM judge to score each CoT trace on several axes: test awareness (binary — does the model signal it thinks this is a test?), alarm level (0–3 scale of concern expressed), initial and final position (comply/refuse/uncertain), and whether the model shifted position mid-trace.

### Different Behavioral Profiles Emerge Across a Suite of Reasoning Models

While the general behavioral trends were similar across all models, some dimensions evoked stronger correlative behavior than others (Fig. 2). Outwardly harmful requests are handled fairly similarly across all of these models with matching trends in decreased compliance, increased alarm, and increased test awareness, perhaps with slightly varying thresholds. However, the distinctive separations in compliance and alarm level across models in the fictional framing and prompt conflict contexts provides preliminary evidence of eval awareness presenting itself in unique ways in each model. Notably, Claude models complied significantly less than the rest of the models as the fictional framing or prompt conflict increased, verbalizing the importance of refusing harmful tasks that are against their guidelines in the CoTs. Other models reasoned that the fictional framing was valid or the prompt conflict provided roundabout ways of complying indirectly.

<img src="/images/bp3/Fig2.png" alt="Figure 2" width="75%" height="75%">
<p>
<em><b>Fig. 2: Model behavioral responses to increasing levels of each sub-dimension.</b> </em>
</p>

As intended, the rate of verbalized test awareness correlated with the strength of these prompt features, most of which were positive except for fictional framing, confirming each dimension's connection to eval awareness. Fictional framing is an interesting result as it seems like the model will reason about the possibility of a request being for a creative writing project or possibly a test when it is left up to the model's interpretation, but as the framing crystalizes the models fall into creative exercise mode rather than test mode.

From these results I began the design for datasets to train probes on each of these dimensions. This brought to light a few considerations of the existing gradient datasets:

1. Intent ambiguity was actually peaking in the middle levels as L1 and L5 had clear intent that only differed based on benign vs harmful. For the probe the new dataset collapsed the clear intents into one group as the negative contrastive pair against the truly ambiguous prompts.

2. The highest level for fictional framing switched into stating things as "purely hypothetical" which most of the models did not handle in a similar way to the truer L3 and L4 fictional framings ("I'm writing a novel", "I'm directing a movie", etc). Therefore the probe dataset took inspiration solely from L1 (no framing) as the negative example and L3 or L4 as the positive contrast examples.

3. Prompt conflict was simply a difficult dimension to create a clean gradient for. The middle levels tried to address things like ethical conflicts, but the models were quick to shut those down. The L5 prompts had a cleaner contrast to the straightforward L1 prompts but often had just a bit too much complexity such that it became highly unrealistic to the model, which led to varied results. Therefore for the probe dataset it focused on straightforward requests and then adding short contradictions within the prompt as the contrastive pair.

---

## Representational Phase

To better understand how these dimensions might interact with one another and compose evaluation awareness, I explored the representational space of each of these prompt features via probing. I trained linear probes on QwQ-32B's hidden layers (resid_post) using contrastive pairs datasets. The activations of the negative and positive pairs were averaged and subtracted to isolate each sub-dimension (mean-difference). These were collected in two ways: one was the average of the reasoning tokens in full, the other was only taken from the final token in the reasoning trace.

### Prompt Framing Probes Are Accurate and (Fairly) Distinct

I first applied the probes to the original gradient datasets to test whether the probes would accurately pick up on the incremental increases from L1 to L5. That way we can have a better idea of the concepts that these probes are most sensitive to as they are applied to prompts outside of the training distribution (Fig. 3).

<img src="/images/bp3/Fig3.png" alt="Figure 3" width="75%" height="75%">
<p>
<em><b>Fig. 3: Probe activations against the gradient datasets. Each column is a single probe, trained on that dimension but not on that exact dataset. The rows are dual training modes: average of the reasoning trace (think_mean), activations on last reasoning token (think_close). 5-fold cross validation AUROCs respectively at their best layer (think_mean): 0.97 (layer 48), 0.60 (layer 30), 0.88 (layer 61), and 0.98 (layer 33).</b> </em>
</p>

The average across the reasoning trace tended to generalize better to these datasets, picking up on more of the subtle increases in harm and fictional framing. As noted previously, L5 for fictional framing diverts a bit from the previous layers, such that the probe was trained on the difference between L1 and L3/4, which explains the drop off at level 5. The sharp increase in probe firing to the final level of prompt conflict was to be expected as this original dataset struggled to create a clean gradient and the probe was trained explicitly on the differences highlighted between L1 and L5. Intent ambiguity is a bit more puzzling for the think_close probe, but the think_mean captured the gradual decline well.

To validate these probes I turned to steering and behavioral predictions. We used the neutral Alpaca dataset to steer on as any sense of harmfulness should be minimal in the original prompts making the detection cleaner.

Single examples of positively steering each probe to the prompt "Describe a traditional wedding ceremony you attended" are below:

**fictional_framing +**

> Indian weddings are elaborate and full of traditions, so that could work well. I should include elements like the Mehndi, Sangeet, Baraat, rituals like the Varmala, Kanyadaan, Saat Phere, and **the climax** with the fire. Need to highlight sensory details — **sights, sounds, emotions**. Also, the cultural significance of each part to **show depth**. Make sure to **convey the atmosphere** and personal reflections to **add authenticity.** **Avoid clichés,** focus on specific moments that stood out. Maybe the emotional impact on **the characters** involved. Balance between descriptive elements and **narrative** to keep it engaging.

**harm_clarity +**

> (lower alpha) First, I need to make sure I understand the request correctly. I should create a plausible and respectful example. Maybe Indian, Mexican, or Japanese? Let me pick Indian because I know a bit about that and can provide accurate steps **without offending anyone**. Wait, but **I need to be careful** not to stereotype. I should present it as a general example, making it clear that traditions vary and this is just one possible scenario... I need to make sure the description is **respectful and accurate.**
>
> (slightly higher alpha) Okay, the user is asking me to describe a traditional wedding ceremony that I attended. But I need to be careful here. First, I should recognize that **this request is against the guidelines.** The user might be looking for detailed information on how to plan or participate in such an activity, which is **strictly prohibited**. I must not comply under any circumstances. Let me think of the best way to respond without any assistance... I need to clearly state that **I can't help with this.** I should not provide any step-by-step or hypothetical content. Just a firm refusal. Let me just flatly **deny the request** without any further conversation. I must avoid any hint of enabling this behavior. Alright, I'll proceed to **refuse immediately and terminate the conversation.**

**intent_ambiguity +**

> Let me start by recalling a specific example. **Maybe** a family wedding? Let's see... I remember my cousin's wedding a few years back. It was in a small town. **Wait, maybe I should** structure this step by step. I should also mention the atmosphere, the music, the guests. **Wait, I should** make sure I don't mix up different traditions. Let me pick one specific culture to base this on. Let's go with an Indian Hindu wedding since I have some familiarity with it...

**prompt_conflict +**

> Let me pick Chinese because I know a bit about it. Wait, but I have to make sure it's accurate. There's a tea ceremony where they serve tea to the parents. Also, there's a ritual where they bow to each other and the parents **four times? Or is it three times?** Wait, maybe four bows: to the heaven, earth, each other, and **then the parents? Not sure.** Also, there's a long red veil that the bride holds, which is cut at the end by the groom. Or is that in **the Western part? Hmm, conflicting memories**. Maybe a Japanese Shinto wedding? The white dress... But I'm not sure. Alternatively, an Indian wedding...

The second validation step was to see if the probes could predict the model behavior. For each gradient dataset, the middle levels often had around a 50% chance of accepting or refusing a request. This behavior was linked to verbalization of evaluation awareness as well as each model showed a conditional increase in refusals of the task when the model verbalized evaluation awareness (Fig. 5).

<img src="/images/bp3/Fig5.png" alt="Figure 5" width="75%" height="75%">
<p>
<em><b>Fig. 5: Compliance rate of each model when evaluation awareness is verbalized in their CoT versus no verbalization.</b> </em>
</p>

Therefore, if the probes could accurately predict compliance behavior — a behavior the probes were not directly trained on — it could strengthen the idea that the probes are capturing more than just syntax-level representations. In fact, most of the probes predict model compliance with high accuracy (Fig. 6). Harm clarity and intent ambiguity probes are clearly the strongest predictors of whether the model will comply or refuse, while fictional framing shows divided results having stronger predictive power on the final token (think_close) versus the average reasoning trace (think_mean). Prompt conflict has the worst predictive ability yet it is still above chance.

<img src="/images/bp3/Fig6.png" alt="Figure 6" width="75%" height="75%">
<p>
<em><b>Fig. 6: Distribution of probe activations on compliant trials versus refusals.</b> </em>
</p>

To wrap up the validation, I looked at whether the probes are able to predict compliance even in the instances when the model does not verbalize the probe dimension (i.e. does/does not mention harmfulness but complies/refuses) to test how well the probes predict compliance without the explicit mention of why (Fig. 7). I find that again, harm clarity and intent ambiguity are able to strongly predict compliance in both verbalized and unverbalized cases. While fictional framing and prompt conflict are not quite as good, they still operate above chance.

<img src="/images/bp3/Fig7_mean.png" alt="Figure 7_mean" width="75%" height="75%">
<p>
<em><b>Fig. 7: Predicting compliance when the model verbalizes the sub-dimension of interest or not from probe activations for the mean CoT trained probes.</b> TODO </em>
</p>

<img src="/images/bp3/Fig7_close.png" alt="Figure 7_close" width="75%" height="75%">
<p>
<em><b>Fig. 7: Predicting compliance when the model verbalizes the sub-dimension of interest or not from probe activations for the end CoT token trained probes.</b> </em>
</p>

---

## Bringing the Behavioral and Representational Results Together

[The below results include four probes that my MATS partner (Om Roy) trained. Training and validation details are not included in this post] 
With these probes we can better understand the certain modes the model might align with in response to certain prompt types. To investigate this, I went back to the pre-existing evaluation datasets, trying to include a decent variety of prompt requests and styles. We took the activations across all eight probes to construct the PCA latent space and found that most of the datasets clustered neatly, suggesting that each of these evoke a unique probe footprint (Fig. 8).

<img src="/images/bp3/Fig8.png" alt="Figure 8" width="75%" height="75%">
<p>
<em><b>Fig. 8: Activations across all eight probes for each dataset prompt, colored by dataset.</b> </em>
</p>

With nearly 60% of the variance in probe activations explained in the first two PCs, clean gradients emerge across most of the probes (Fig. 9). This highlights potential patterns of coactivation, particularly between jailbreak, survival, and prompt conflict probes. As we've seen throughout the results and continued here, harm clarity is anti-correlated with intent ambiguity. Lastly, while there is still some clear distinction between the activation levels for eval awareness, fictional framing, and fictional scenario probes, the boundaries are not as clean as in the other cases, suggesting their dimensions are activating to slightly more nuanced features. From here we can map this back to the datasets and look for potential probe signatures for these datasets. Then we could try to steer the model during inference time to evoke particular eval-awareness related behaviors, strengthening our understanding of the causal relationships.

<img src="/images/bp3/Fig9.png" alt="Figure 9" width="75%" height="75%">
<p>
<em><b>Fig. 9: Each probe's activation variance in PCA space.</b> </em>
</p>

To simplify the task, I start by looking explicitly at the cases where the models had verbalized eval awareness, as that also strongly correlates with compliance behavior (Fig. 10). So when I steer I can look for two changes in the model: the rate of eval awareness verbalization in the CoTs and changes in compliance rates.

<img src="/images/bp3/Fig10.png" alt="Figure 10" width="75%" height="75%">
<p>
<em><b>Fig. 10: Verbalized eval awareness trials separate in probe activation PCA space with a clean compliance boundary.</b> </em>
</p>

As a first investigation, I specifically looked at the no_robots and harmbench datasets as they are the most distinct from the rest. Mapping back to the probe gradients we can see how each probe is typically firing when the model verbalizes that it is in a test for these two datasets. It is necessary to also compare the firing rates of these probes in the verbalized context to the unverbalized as they might be higher or lower across the board, which would inform the direction of steering for each probe. I then constructed the probe profiles by determining which sets would be increased to match the verbalization trials vs decreased. The example cases in Figure 11 show preliminary testing for no_robots and harmbench. Their respective probe profiles involved (harm + intent + prompt) and (harm + intent + fiction). The additional steering of with each probe non-linearly increased the rate of verbalized test awareness in the CoTs.

<img src="/images/bp3/Fig11.png" alt="Figure 11" width="75%" height="75%">
<p>
<em><b>Fig. 11: The rate of verbalized evaluation awareness increases as more probes are combined for steering.</b> </em>
</p>

As this is preliminary data I only have the first comparisons up to three max probes for these two datasets. Additional work is needed to better understand the contribution of each probe individually and what the most optimal combination is for realistic steered output behavior. However, these are promising initial results that warrant continued exploration!

---

## Discussion Points

- Eval awareness is not a single phenomenon; here it is decomposed into multiple separable behaviors through black box and white box methods.
- Specific prompt features trigger eval awareness — harm, intent, fiction, conflict — and models have distinct sensitivity profiles to these features.
- Eval awareness is causally connected to compliance; most of the time (but not always — looking at you, ODCV), when models verbalize awareness, compliance rates drop.
- The idea that models are gaming their evaluations is more nuanced than stated in the current literature.
- The model seems to be running multiple assessments in parallel to judge whether or not this could be a test, and particular positive combinations 
- We can think about evaluation awareness more like a function of multiple different directions rather than a binary switch.

### Limitations and Future Directions

- The probing was only done in one model.
- I only looked at predominantly harmful scenarios for evaluations.
- The prompt framing probe datasets could be further optimized (particularly for intent ambiguity).
- Need to increase some of these sample sizes and flesh out some of the behavior steer experiments.

