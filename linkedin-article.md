# I Fine-Tuned a 7B Model on My MacBook. Here's the Governance Case for Why You Should Care.

Last week I fine-tuned Mistral 7B locally on Apple Silicon and deployed it through Ollama. The whole pipeline ran on my MacBook — training, fusing, converting, serving. I'm publishing the full technical walkthrough on GitHub because most tutorials skip the parts where things break. But the technical details aren't the point of this article.

The point is the governance model behind it.

---

## Who I Am and Why That Context Matters

I've spent 25 years in enterprise cybersecurity — workforce identity programs at Fortune 32 scale, Zero Trust architecture, IAM across 60,000-user environments. I hold a CRISC certification and have contributed to the development of two international standards: IEC 63452 on Cyber-Security for ICT and BSIGEL 9/6AIML on AI/ML Governance for Cyber-Security.

I'm a practitioner who thinks carefully about data exposure, control architecture, and what "responsible AI deployment" actually means in practice — not just in policy documents.

What I built here is small. The principles behind it aren't.

---

## The Problem with How Most Organizations Are Using AI Right Now

Every time an employee pastes internal notes, draft communications, or organizational context into a commercial LLM API, that data is transmitted to a third-party processor. In most organizations, that transmission hasn't been formally evaluated under GDPR, HIPAA, or the organization's own data classification policy.

That's not a hypothetical risk. It's a gap that exists right now in most enterprise AI deployments.

The standard response is "we have a business agreement with the vendor." That's not a control. That's a contract. Controls prevent exposure. Contracts respond to it after the fact.

---

## What I Built and Why

My use case is straightforward: staff generate first-draft memos and documents from my notes. Every output is reviewed and approved by me before it goes anywhere.

The model needed to match a specific communication standard — direct, brief, no filler. Off-the-shelf LLMs default to verbose, over-qualified language. Fine-tuning on curated examples of the actual standard I want is more durable than prompt engineering, which degrades over long context windows and doesn't hold across sessions.

The governance architecture is the point:

**Privacy**: The model runs locally. Nothing leaves the machine during inference. No API call, no data processor, no exposure.

**Auditability**: The training data is 57 real examples I selected and controlled. I know exactly what the model was trained on. That's not true of any foundation model I didn't train myself.

**Quality control**: Human review before every output ships. This is the control that makes the whole system safe to scale. The model produces a draft. I approve the final. That division of labor is explicit and enforced — not aspirational.

---

## The Technical Reality

This ran on a MacBook with Apple Silicon. Training took under 3 minutes. Peak memory was 16.7 GB. Validation loss dropped from 2.97 to 0.57 over 100 iterations.

The stack: MLX-LM for LoRA fine-tuning, llama.cpp for GGUF conversion, Ollama for local serving. All open source. All local.

The barrier to this kind of deployment is lower than most people think. The tooling exists and it works. What's been missing is documentation that covers the full path — including every error and fix — in one place.

That's what I published.

---

## What This Means for Enterprise

I'm not suggesting every organization should fine-tune their own models. I am suggesting that local model deployment is now a viable option that belongs in the evaluation set when organizations make AI deployment decisions — especially for workflows that touch sensitive communications, personnel matters, legal drafts, or anything else with data classification implications.

The questions worth asking:

- What data are your employees sending to commercial AI APIs right now?
- Has that data flow been evaluated under your privacy and security frameworks?
- What would a locally hosted, human-reviewed AI workflow look like for your highest-sensitivity use cases?

These are governance questions. The technology to answer them is already available.

---

## What's Next

This is version one. 57 training examples demonstrates the approach. The next iteration expands the dataset, tightens validation, and documents the evaluation methodology more rigorously. I'm also exploring how this architecture applies to higher-stakes workflows where the privacy and auditability requirements are even more demanding.

The full technical walkthrough — every command, every error, every fix — is on GitHub. If you're building something similar or thinking through the governance implications for your organization, I'd like to hear from you.

---

**Full technical walkthrough and training pipeline: [GitHub link]**

---

*Oric Perry is a CRISC-certified cybersecurity executive and Principal Consultant at OSP Global Solutions, specializing in agentic AI security, workforce identity, and enterprise AI governance. Contributing member of IEC 63452 and BSIGEL 9/6AIML. 25+ years in enterprise cybersecurity.*

*Connect: [linkedin.com/in/oricperrycybergrc](https://linkedin.com/in/oricperrycybergrc)*
