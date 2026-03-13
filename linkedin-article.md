# I Fine-Tuned a 7B Model on My MacBook. Here's What I Learned About Governed AI Deployment.

This project started because I wanted to refresh myself on tuning a language model from scratch. What I ended up with was a working proof of concept for a governed, private AI workflow, and a clearer understanding of what ethical AI deployment requires in practice, not just in policy documents.

---

## Who I Am and Why That Context Matters

I've spent 25 years in enterprise cybersecurity workforce identity programs at Fortune 32 scale, Zero Trust architecture, IAM across 60,000-user environments. I hold a CRISC certification and have contributed to international standards work in AI/ML Governance and Cyber-Security for ICT.

My work is at the intersection of technical implementation and governance frameworks. This project was a deliberate attempt to close the gap between the two to build something small, observable, and governed, and then reason about what that process teaches.

---

## The Driver: Learning by Building

I wanted to understand the fine-tuning lineage of a model as it progressed from planning through production. The only way to do that is to run it yourself, hit the failure points, and work through them. I call this the .doEffect.

My secondary goal was to produce an output that demonstrates the outcome in terms anyone can evaluate. Human evaluation is accomplished through evaluating legitimate text. If a model learns to respond with brevity and directness, you can read that. You can use your choice of QA scoring to evaluate whether the tuning worked.

The tertiary goal and the one that matters most for sharing this publicly was to reason through what managing an ethical AI initiative actually looks like when you're the one building it, not just auditing it.

---

## What I Built

On my laptop, I tuned a model using a low-impact approach. The model was trained on real examples of my communication style, which is direct and concise. This is the standard I wanted the model to reproduce.

Team members use this model to generate first-draft memos and documents directly from my executive notes. Because the model was fine-tuned on my communication style, outputs require significantly less editing. Outputs are draft-ready and aligned to my voice before final human review and approval.

**The governance architecture is explicit:**

- **Privacy**: The model runs locally and nothing leaves the machine during inference. This approach reduces the likelihood of data loss and minimizes model risk.
- **Auditability**: The training data is 57 examples I selected and controlled. The model's behavior is traceable to a known, versioned dataset.
- **Human-in-the-loop**: As a quality assurance checkpoint, every output passes through a human-gated review stage before promotion. It mirrors CI/CD pipeline discipline where no artifact ships without passing a defined quality gate. Removing that gate is where I observe AI deployment failures, socially and technically.

---

## The Ethical AI Dimension: Where NIST AI RMF Meets Practice

The NIST AI Risk Management Framework (AI RMF, AI 100-1) provides a voluntary framework for organizations to manage risks associated with AI systems across the full model lifecycle. Its core functions, Govern, Map, Measure, and Manage, are directly applicable to what this project demonstrates. The governance decisions in this build map explicitly onto the AI RMF's principles for trustworthy AI deployment.

NIST AI RMF aligns with IEEE Ethics principles on three dimensions this project directly addresses:

**1. Accountability and traceability of AI behavior.** The AI RMF and IEEE Ethics both require that automated systems be traceable and that their behavior can be explained and audited. A fine-tuned model trained on a known, versioned dataset satisfies this requirement in a way that a prompted model does not. System prompts are invisible to auditors. Training data is fixed and reviewable.

**2. Human oversight in the model development lifecycle as a non-negotiable design requirement.** The AI RMF's Govern function, consistent with IEEE Ethics principles, treats human oversight of automated decision-making as a core control. Without that control, model accuracy and accountability cannot be assured.

**3. Data minimization and privacy by design.** The AI RMF incorporates Privacy by Design (PbD) principles consistent with GDPR, HIPAA, and LGPD. Local inference where no data leaves the machine is the strongest possible implementation of data minimization for an AI workflow. Organizations that handle sensitive data can use capable AI without creating third-party data processor exposure they haven't evaluated or disclosed.

---

## What the Outcome Demonstrates

Validation loss dropped from 2.97 to 0.57 over 100 training iterations. The chart below shows the full training run.

![Validation Loss Chart](assets/validation_loss.png)

The same question was asked to each model. The comparison below shows the difference in output between the fine-tuned model and the base Mistral model.

![Model Comparison](assets/model_comparison.png)

The output is legible. You can read a response and evaluate whether it meets the standard. That legibility is itself a governance property. It makes the model's behavior reviewable by non-technical stakeholders, which is a prerequisite for responsible deployment at any scale.

---

## Who This Is For

This architecture is relevant to any organization that:
- Handles sensitive communications that shouldn't leave internal infrastructure
- Operates under GDPR, HIPAA, LGPD, or similar data privacy frameworks
- Wants AI-assisted workflows with a defensible human oversight model

The barrier to this kind of deployment is lower than most practitioners think. This model tuning process works on consumer hardware, and the governance model is straightforward to implement and audit.

---

## What's Next

This is version one. The next iteration expands the training dataset and explores how this architecture applies to higher-stakes workflows. I'm also interested in how the traceability properties of fine-tuned local models map to emerging AI audit requirements under frameworks like the EU AI Act.

The full technical walkthrough is on GitHub. If you're working through similar questions on governed AI deployment, I'd like to hear from you.

---

**Full technical walkthrough and training pipeline: https://github.com/osperry/mistral-lora-finetune**

---

*Oric Perry is a CRISC-certified cybersecurity executive and Principal Consultant at OSP Global Solutions, specializing in agentic AI security, workforce identity, and enterprise AI governance. Contributing member of BSIGEL 9/6AIML (AI/ML Governance for Cyber-Security). 25+ years in enterprise cybersecurity.*

*Connect: [linkedin.com/in/oricperrycybergrc](https://linkedin.com/in/oricperrycybergrc)*
