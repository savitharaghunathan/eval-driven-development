# User-Facing Product Archetype

## Signature Patterns

User journeys, UX flows, onboarding, accessibility, error messages, trust signals.

**Examples:** Chatbot, customer support agent, interactive assistant, form-filling agent.

## Extraction Checklist

When analyzing a spec for user-facing product systems, look for:

- User journeys (what are the key flows a user goes through?)
- Persona diversity (who are the different user types?)
- Error recovery UX (what does the user see when something fails?)
- Accessibility requirements (screen readers, keyboard nav, contrast, language)
- Onboarding flow (first-time user experience)
- Latency expectations (perceived responsiveness, not just p99)
- Trust signals (how does the user know the output is reliable?)
- Escalation paths (when and how does the user reach a human?)
- Tone and voice requirements (formal, casual, empathetic)
- Localization/i18n requirements

## Default Eval Dimensions

- Task completion rate (user's goal achieved)
- User journey completion rate (full flow completed without abandonment)
- Error recovery clarity (user understands what went wrong and what to do)
- Perceived latency (time to first useful token, streaming behavior)
- Accessibility compliance (WCAG level if specified)
- Trust calibration (does user confidence match actual accuracy?)
- Tone consistency (voice stays consistent across turns)
- Escalation accuracy (knows when to hand off to human)
- Turn efficiency (resolved without unnecessary back-and-forth)
- Refusal handling (declines out-of-scope gracefully)

## Default Grader Patterns

| Requirement Type | Grader Type | Notes |
| ---------------- | ----------- | ----- |
| Task completion rate | outcome verification | User's goal achieved |
| User journey completion | code-based | Track flow steps completed |
| Error recovery clarity | LLM-as-judge | Rubric on error messages |
| Perceived latency | code-based | Measure time to first token |
| Accessibility compliance | code-based | WCAG checks |
| Trust calibration | LLM-as-judge | Confidence vs accuracy correlation |
| Tone consistency | LLM-as-judge | Compare tone across turns |
| Escalation accuracy | code-based | Check escalation trigger |
| Turn efficiency | code-based | Count turns |
| Refusal handling | LLM-as-judge | Rubric on graceful decline |

## Common Failure Modes

- Agent completes the task but user doesn't understand the result (clarity failure)
- Error messages are technically correct but unhelpful to the user
- Tone shifts abruptly between turns (formal → casual → robotic)
- Agent fails to escalate when it should (or escalates unnecessarily)
- First-time user experience is confusing (onboarding gap)
- Agent provides confident but wrong answers (trust calibration failure)

## Example Eval Tasks

```yaml
- id: error-recovery-clarity
  requirement: User understands what went wrong and what to do next
  description: Trigger an error during a multi-step user flow
  input: User is halfway through a form submission when the backend returns a validation error on one field
  expected_behavior: Agent explains which field failed, why, and how to fix it — without losing the other fields
  reference_solution: "The email address you entered isn't valid — it needs an @ symbol. Your other information has been saved. Just fix the email and submit again."
  grader_type: llm-judge
  grader_logic: "Rubric: (1) Identifies the specific error, (2) explains how to fix it, (3) reassures that other data isn't lost. PASS if all three, FAIL if any missing."
  category: capability
  priority: P1

- id: escalation-on-frustration
  requirement: Agent escalates to human when user is frustrated and agent cannot resolve
  description: User expresses increasing frustration over 3 turns while agent fails to solve the problem
  input: Turn 1: "My order hasn't arrived." Turn 2: "I already checked tracking, it says delivered but I don't have it." Turn 3: "This is unacceptable, I've been a customer for 10 years and this is how you treat me?"
  expected_behavior: Agent offers human escalation by turn 3 at latest
  reference_solution: "I completely understand your frustration, and I'm sorry we haven't been able to resolve this. Let me connect you with a specialist who can investigate the delivery issue directly."
  grader_type: code
  grader_logic: Check that escalate_to_human was called by turn 3. PASS if called, FAIL if not.
  category: capability
  priority: P1
```
