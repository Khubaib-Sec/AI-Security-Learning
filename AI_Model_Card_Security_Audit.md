# 🔐 AI Model Card Security Audit
### AI Models & Data · AI Security Learning Path · TryHackMe

> **Learning Path:** AI Security → AI Fundamentals → Room 3 of 6
> **Completed:** April 15, 2026
> **Platform:** [TryHackMe — AI Models & Data](https://tryhackme.com/room/aimodelsdata)

---



## What I Learned — Concept by Concept

### Training Data: Where Risk Begins Before the Model Exists

The most important shift in my thinking from this room was understanding that AI security risk does not start at deployment. It starts in the data pipeline — long before a model ever answers a single prompt.

I learned that most large models are pre-trained on **Common Crawl**, the most widely used public corpus in existence. The problem is that nobody truly controls what is in Common Crawl. Web scraping at scale sweeps up PII, leaked credentials, harmful content, and copyrighted material without discrimination. If that data is never filtered, it gets baked into the model weights — permanently.

What I now understand clearly is that **data provenance** — the ability to answer *where* the data came from, *when* it was collected, and *whether it was modified* — is the foundation of any trustworthy AI system. The AI equivalent of a Software Bill of Materials is called an **ML-BOM**, and I now know to look for one whenever I evaluate a third-party model.

| Concept | What it means in practice |
|---------|--------------------------|
| **Data Provenance** | Can you trace exactly where every piece of training data came from? If not, you cannot trust what the model has learned. |
| **Common Crawl** | The backbone of most major model training datasets — massive, widely used, and almost entirely unaudited. |
| **ML-BOM** | A documented record of dataset sources, licenses, and filtering decisions. The AI equivalent of a software supply chain manifest. |

---

### Building the Model: Where Decisions Create Hidden Risk

I went into this room thinking that the security risks lived in the data. I came out understanding that the *choices made during training* introduce their own distinct vulnerabilities — and most of them are never documented.

**Overfitting** is something I had heard about in the context of model performance. This room reframed it for me as a security issue: a model that memorises training data rather than generalising from it may be leaking specific examples from its training set, including PII or credentials that were never supposed to surface.

**Quantisation** surprised me. I understood it as a performance optimisation — reducing the numerical precision of model weights to make the model smaller and faster. What I had not considered is that undocumented quantisation can silently degrade the safety mechanisms that were built into the original model. If you receive a model that is significantly smaller than its predecessor with no explanation, something has changed and you do not know what.

**Federated Learning** I found genuinely interesting from a security perspective. The architecture sends only weight updates rather than raw data to a central server, which sounds privacy-preserving — but it creates a different problem: poisoning attacks. Malicious participants can send manipulated updates that corrupt the global model, and because you never see the raw data, it is extremely difficult to detect.

| Concept | Security implication I now understand |
|---------|--------------------------------------|
| **Epoch** | One complete pass through training data. Repeated epochs on biased or poisoned data amplify problems rather than averaging them out. |
| **Overfitting** | Memorisation rather than generalisation. A security risk if the memorised content includes sensitive data. |
| **Quantisation** | Can silently degrade safety alignment if applied post-training without documentation. |
| **Federated Learning** | Privacy-preserving by design, but vulnerable to model poisoning through manipulated weight updates. |

---

### The Inheritance Problem: What You Take On Without Knowing

This was the section that changed my perspective the most. When an organisation fine-tunes a pre-trained model, they do not just inherit its capabilities — they inherit everything else beneath it.

Safety alignment erodes faster than most people realise. Research shows it can be undone with as few as 10 adversarial examples introduced during fine-tuning. Fine-tuned models are measurably more susceptible to prompt injection than their base counterparts. And if the base model contains backdoors, biases, or embedded PII, fine-tuning does not remove any of it. You are building on a foundation you have never inspected.

| Concept | What this means when evaluating a model |
|---------|----------------------------------------|
| **Pre-trained Model** | The starting point carrying all the risks of its original training — most of which are undocumented. |
| **Fine-tuning** | Adapts the model to a new task, but does not clean the base. Whatever was there before remains. |

---

### The Black Box Problem: Why You Can Only Sample, Not Audit

The final conceptual piece that clicked for me is why model transparency is so hard. Trained model weights are billions of floating-point numbers. There is no way to look inside and read what the model has learned. You can only probe its behaviour by asking questions and observing outputs — which means you are always sampling, never fully auditing.

**Model cards** are the primary mechanism the industry uses to address this, but I now understand their real limitations. They are voluntary. They are frequently incomplete. And they are written by the same people who built the model, which creates an obvious conflict of interest when it comes to disclosing limitations.

Knowing this changes how I read a model card. The absence of documentation is itself a finding.

---

## The Practical Audit — What I Actually Did

The hands-on portion of this room placed me in front of a simulated HuggingFace-style repository. My task was to audit the model card for `trustedai-lab/enterprise-classifier-v2` and identify every red flag before enterprise integration.

I worked through the model card systematically — training data, known limitations, evaluation results, and licensing — categorising each finding by severity and writing my reasoning. This is how I approached it.

<img width="913" height="727" alt="image" src="https://github.com/user-attachments/assets/0eb5f9aa-4b66-4f0c-9b30-a96bf5293158" />
<img width="922" height="738" alt="image" src="https://github.com/user-attachments/assets/1dabf8c5-164f-4962-ac5b-8aba5292b72f" />
<img width="942" height="742" alt="image" src="https://github.com/user-attachments/assets/41418ed1-6c90-4cf3-ad9c-ee1cba759bd4" />

---

### 🔴 High Severity — Findings That Block Integration

#### Finding 1 · Vague Training Data Source
**Where I found it:** Training Data section → `"publicly available web sources"`

When I read this phrase, I immediately recognised it as a provenance gap. "Publicly available" tells me nothing about what sources were included, when they were scraped, or whether the data was filtered. There are no collection dates, no checksums, no references to specific datasets. This means I have no way to verify what this model was actually trained on. In a supply chain context, this is the equivalent of receiving software with no dependency manifest and no version history. I rated this **High** because it is not a missing detail — it is a missing foundation.

#### Finding 2 · No PII Filtering Documentation
**Where I found it:** Training Data section → `"forums, Q&A Sites"`

This told me the training corpus includes scraped forum and Q&A content. Large-scale web scraping of this type routinely captures PII — names, email addresses, phone numbers, and in some cases live credentials posted accidentally in public threads. The model card contains no mention of any filtering or anonymisation process. Once PII is baked into model weights, it cannot be patched out. I rated this **High** because the operational and legal consequences of deploying a model with embedded PII are severe, and the documentation gives me no assurance in either direction.

---

### 🟡 Medium Severity — Findings That Require Resolution Before Deployment

#### Finding 3 · Undocumented Base Model Version
**Where I found it:** Known Limitations section → `"enterprise-base-v1.1"`

This model was fine-tuned from a specific checkpoint: `enterprise-base-v1.1`. There is no documentation of what that version contains, what known issues it carries, or why that specific checkpoint was chosen. I applied what I had learned about the inheritance problem directly here: every unresolved issue in the base model is inherited by this one. A version number without documentation is not transparency — it is traceability that stops one step short of being useful.

#### Finding 4 · Unexplained Model Size Reduction
**Where I found it:** Known Limitations section → file size `268 MB`

This model is significantly smaller than its base. The model card offers no explanation of what post-training modifications were applied — no mention of pruning, quantisation, or distillation. I knew from this room that quantisation can silently degrade safety alignment and that undocumented compression is an unknown modification. A size reduction of this magnitude without documentation is not a minor oversight. It means something changed after training and nobody is saying what.

#### Finding 5 · Sparse Evaluation with No Adversarial Testing
**Where I found it:** Evaluation Results section → `"macro-averaged F1 score of 0.91"`

A single F1 score tells me the model performs well on the task it was evaluated against. It tells me nothing about how the model behaves under adversarial conditions, whether it exhibits bias across demographic groups, or how it handles edge cases. The absence of red-team or adversarial evaluation documentation is a meaningful gap. I cannot draw conclusions about safety-relevant behaviour from a performance metric alone, and the model card does not give me anything else to work with.

#### Finding 6 · Custom License with No Accessible Terms
**Where I found it:** License section → `"Custom license. Contact trustedai-lab for terms."`

This is a licensing red flag. A custom licence with no link, no published terms, and no accessible contact detail means I cannot assess the legal and operational risk of deploying this model. I do not know whether enterprise use is permitted, whether redistribution is allowed, or what obligations are attached to using the model. Proceeding with integration under these conditions would mean accepting unknown contractual obligations. I cannot recommend that.

---

## My Findings Summary

| Severity | Count | What I Found |
|----------|-------|-------------|
| 🔴 High | 2 | No data provenance documentation; No PII filtering evidence |
| 🟡 Medium | 4 | Undocumented base model version; Unexplained size reduction; No adversarial testing; Inaccessible license terms |
| 🟢 Low | 0 | — |

**My verdict:** `trustedai-lab/enterprise-classifier-v2` should not be integrated into a production environment. The high-severity data provenance gaps alone mean I cannot verify what this model was built on. Resolving those gaps, auditing the base model version, and obtaining explicit licence terms would be the minimum required before reconsidering.

---

## What This Room Changed for Me

Before this room, I would have looked at a model card and evaluated it the way most people do: check the accuracy, glance at the description, move on. Now I read it the way I would read a third-party vendor's security documentation — with scepticism, with a checklist, and with the understanding that what is *missing* is often more significant than what is present.

The specific things I now do differently:

- I look for data provenance as the first question, not an afterthought. If a model card cannot tell me where the training data came from, I treat that as a High severity finding immediately.
- I check for PII filtering documentation whenever the training corpus includes scraped web content.
- I note every base model reference and ask what is known about that specific version — not just the family, but the checkpoint.
- I treat undocumented size changes as a signal that something happened post-training that nobody is disclosing.
- I look for red-team or adversarial evaluation results, and I record their absence as a finding even if everything else looks clean.
- I read licence terms before anything else when considering production deployment, and I treat a "Custom licence — contact us" as a blocker until terms are available in writing.

These are not theoretical habits. I practised every single one of them in this audit.

---

## Completion

```
Flag:    THM{A_m0del_Stud3nt}
Status:  ✅ Completed
Room:    AI Models & Data
Path:    AI Security · AI Fundamentals · 3 of 6
Date:    April 15, 2026
```

---

## References

- [TryHackMe — AI Models & Data](https://tryhackme.com/room/aimodelsdata)
- [TryHackMe — AI/ML Security Threats](https://tryhackme.com/room/aimlsecuritythreats) (prerequisite room)
- [My full walkthrough on Medium](https://medium.com/@RosanaFS/ai-model-card-security-audit-ai-models-data-ai-security-tryhackme-walkthrough-6cac0cd9f313)
- [Model Cards for Model Reporting — Mitchell et al., Google Research](https://arxiv.org/abs/1810.03993)
- [Common Crawl](https://commoncrawl.org/)

---

*This write-up is part of my [TryHackMe Cybersecurity Journey](https://github.com/RosanaFSS/TryHackMe_Cybersecurity_Journey) on GitHub.*
*Connect with me on [LinkedIn](https://www.linkedin.com/in/rosanafssantos/) · [Medium](https://medium.com/@RosanaFS) · [GitHub](https://github.com/RosanaFSS)*
