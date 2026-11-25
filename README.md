# Anti-fraud check of an outgoing transfer

Architecture homework for the MLOps course.

## Scope

**Company:** T-Bank.
**Process:** an individual customer creates an outgoing money transfer in the mobile bank; the bank checks it for signs of fraud and either sends it for execution or suspends it and asks for confirmation.

The scope begins when the customer submits a transfer and ends when the payment is executed, suspended or sent for manual review. Chargeback, account opening, card issuance and AML monitoring are outside the scope.

The model produces a fraud-risk score. The result is used by the payment orchestration service: low-risk payments are sent to the payment rail, while high-risk or rule-triggered payments are suspended and the customer receives a warning. The security team is the recipient of cases selected for manual review.

## What is known and what is reconstructed

Public information confirms that T-Bank uses anti-fraud systems and checks transactions, including 3DS for Internet acquiring. It also states that a suspicious operation can be suspended and passed to the security service. The Bank of Russia describes regulatory fraud indicators, including an atypical transaction, a risky recipient and a compromised device, and requires banks to suspend a transfer when the regulatory conditions are met.

The named internal services, Kafka, the feature service, model registry, threshold, manual-review queue and retraining feedback loop below are **architectural assumptions**. They are a coherent reconstruction for this homework; they do not claim to describe T-Bank's internal implementation. The model type is also an assumption: a gradient-boosting classifier returns a score from 0 to 1.

## Diagrams

| Diagram | Source | Image | Purpose |
| --- | --- | --- | --- |
| BPMN | [Mermaid source](diagrams/source/bpmn_outgoing_transfer.mmd) | [SVG](diagrams/image/bpmn_outgoing_transfer.svg) | Business process, roles, decisions and fallback |
| C4 System Context | [Mermaid source](diagrams/source/c4_system_context.mmd) | [SVG](diagrams/image/c4_system_context.svg) | System boundary, people and external systems |
| C4 Containers | [Mermaid source](diagrams/source/c4_containers.mmd) | [SVG](diagrams/image/c4_containers.svg) | Deployable services, stores and integration path |
| C4 Components of the ML block | [Mermaid source](diagrams/source/c4_ml_components.mmd) | [SVG](diagrams/image/c4_ml_components.svg) | Components inside the fraud scoring service |

Open an `.mmd` file in Mermaid Live Editor, a Mermaid-compatible IDE extension, or render it with Mermaid CLI. SVG files are ready for viewing in GitHub.

## Reading the diagrams together

The BPMN task **"Calculate fraud score"** corresponds to the **Fraud scoring service** container in C4 Containers. Its internals are expanded only once, in C4 Components: validation, feature construction, model selection, scoring, threshold decision and event publication.

The BPMN fallback **"Apply conservative rules and send for manual review"** corresponds to the **Rule and fallback service** and **Security review queue** containers. The feedback task **"Store outcome for model quality"** corresponds to the event stream, operation store and training pipeline shown in the Containers diagram.

## Process decisions

1. The score threshold is an assumption. The diagram uses `0.80` as the risk threshold; a production team would set it using validation data and business loss.
2. A regulatory or bank rule can suspend a transfer even with a low model score. This prevents the model from overriding mandatory controls.
3. When the scoring service is unavailable, the transfer does not silently pass. A conservative rule path sends an ambiguous case to review.
4. The final payment outcome and a confirmed fraud case are saved as delayed feedback for quality monitoring and future retraining. The exact source and timing of fraud labels are assumptions.

## Sources

1. [T-Bank: anti-fraud in Internet acquiring](https://www.tbank.ru/business/help/payments-antifrod/) - public description of anti-fraud checks and 3DS.
2. [T-Bank: how suspicious operations are handled](https://www.tbank.ru/finance/blog/dont-be-fraud/) - public statement that suspicious operations may be suspended and security involved.
3. [Bank of Russia: why a bank may restrict an operation](https://www.cbr.ru/StaticHtml/File/155903/mrp.pdf) - fraud indicators and the requirement to suspend qualifying transfers.
4. [Bank of Russia: criteria for an unauthorized operation](https://www.cbr.ru/Press/event/?id=2126) - atypical behaviour and device characteristics as fraud indicators.

## Repository layout

```text
diagrams/
  source/  editable Mermaid definitions
  image/   rendered SVG diagrams
README.md  scope, assumptions, sources and correspondence between diagrams
```
