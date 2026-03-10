# Incident-Response-SOC-Investigation-project
Incident Response / SOC Investigation project using tryhackme.
Case 8814 incident response walkthrough
How you investigated the phishing ticket, validated the evidence, and documented the closure.
### OVERVIEW
This deck reconstructs the investigation strictly from the screenshots you provided. It shows the exact
workflow: alert review, IOC extraction, SIEM pivoting, log validation, final classification, escalation
rationale, and remediation.
Rule
Inbound Email Containing Suspicious External Link
### Type
Phishing
Severity
Medium
Disposition
True positive phishing
### OUTCOME
• Confirmed the message was delivered to the user mailbox.
• Found no evidence that the link was clicked or that a host was compromised.
• Recommended blocking the domain, removing the message, notifying the user, and filtering the
sender
### Event ID 8814
Prepared from case screenshots only
Investigation flow
• Triage alert record
• Extract IOCs
• Pivot in SIEM
• Validate log lineage
• Assess impact
• Write report and close

## Bottom line: why the ticket was solved
The investigation separated confirmed phishing delivery from unconfirmed downstream compromise.
### KEY FINDINGS
• The trigger was legitimate: the case report tied the alert to an inbound email with a suspicious external
link and explicitly described a phishing scenario.
• The lure was concrete and actionable: onboarding pretext, account setup language, and a direct
hyperlink to hrconnex.thm.
• Your SIEM pivot showed the message existed in the mail telemetry and also uncovered a related
internal onboarding thread that made the lure believable.
• The final incident note recorded that no proxy, firewall, or DNS evidence showed user access to the
link, so there was no identified compromised endpoint.
• Because delivery was confirmed, the event was still handled as a true positive phishing incident and
not dismissed as noise.
### DECISION LOGIC
1. Email with suspicious external link was confirmed in telemetry.
2. Message was delivered to the intended recipient, so the phishing risk was real.
3. No supporting logs showed link access, which limited the impact to exposure rather than compromise.
4. You closed the ticket with a true-positive phishing disposition plus containment recommendations.
### Practical resolution:
Stop future delivery, remove residual copies, and notify the user while preserving
the finding that no endpoint was affected.



## Step 1 — validate the alert context
I started by grounding the investigation in the case metadata and the raw email details.
### WHAT I DID
• Opened event ID 8814 and captured the alert rule, incident type, severity, and reported detection time.
• Read the rule description to understand the expected workflow: review the suspicious link and
determine whether any endpoint attempted to access it via firewall or proxy evidence.
• Expanded Alert details and recorded the message fields: datasource email, inbound direction,
recipient j.garcia@thetrydaily.thm, sender onboarding@hrconnex.thm, and subject “Action Required:
Finalize Your Onboarding Profile.”
• Extracted the embedded URL from the body: https://hrconnex.thm/onboarding/15400654060/j.garcia,
establishing the first IOC set before pivoting anywhere else.
Matched evidence: case report and expanded alert details.
<img width="3135" height="1518" alt="Screenshot 2026-03-09 001956" src="https://github.com/user-attachments/assets/76c225aa-21a9-4e90-8af2-5886038bb422" />

## Step 2 — pivot in SIEM on hrconnex
The domain search tied the alert to live telemetry and uncovered supporting context.
### WHAT I DID
• Ran a SIEM search for hrconnex and confirmed 3 matching events.
• Matched the inbound email event back to the original case: same sender, same recipient, same
subject, same direction, and same phishing link.
• Captured the host value 10.10.156.209:8989 from the event view, preserving another useful technical
artifact from the telemetry path.
• Reviewed the related internal email from h.harris@thetrydaily.thm to j.carter@thetrydaily.thm with
subject “Unable to Receive Onboarding Email from hrconnex.thm.” That message showed why the lure
was persuasive: the onboarding scenario was already in circulation internally.
#### Interpretation
The domain search did more than confirm the phish. It also explained the social-engineering hook by
showing an internal business process touching the same third-party name.
Matched evidence: SIEM query for hrconnex returned 3 correlated events
<img width="3152" height="1474" alt="Screenshot 2026-03-09 002032" src="https://github.com/user-attachments/assets/4d3fa446-8b26-42fc-9632-413a09bb6d2e" />

## Step 3 — verify source consistency
I checked whether the correlated events came from the same collection path.
## WHAT I DID
• Opened the field statistics for source directly from the SIEM result set.
• Confirmed that all returned events shared the same value: source = eventcollector.
• Used that consistency check to reduce ambiguity; the three events were not coming from mixed or
unrelated log pipelines.
• This strengthened confidence that the correlation was valid and that the same telemetry source was
describing the phishing activity and the related context email.
Matched evidence: source field = eventcollector for 100% of returned events
<img width="746" height="562" alt="image" src="https://github.com/user-attachments/assets/b1af3aca-9d08-41c9-b8a7-8fb359b75b2a" />

## Step 4 — verify sourcetype and field parsing
I repeated the integrity check at the sourcetype level to make the extracted fields trustworthy.
### WHAT I DID
• Opened the field statistics for sourcetype on the same result set.
• Confirmed that all correlated events had sourcetype = _json.
• Used that consistency to support the reliability of the parsed fields you were using in the investigation:
sender, recipient, subject, timestamp, datasource, and direction.
• By validating both source and sourcetype, you documented that the IOC extraction and the
message-to-message comparison were based on a stable parsing path.
Matched evidence: sourcetype = _json for 100% of returned events
<img width="746" height="562" alt="image" src="https://github.com/user-attachments/assets/2c930dcc-5c37-48b4-bad4-c2291a3415b6" />

## Step 5 — classify, escalate, and close
The final write-up captured both the confirmed risk and the absence of downstream compromise.
### WHAT I RECORDED
• Time of activity: 3/9/26 3:53:39.471 AM, aligning the reporting with the SIEM event timeline.
• Incident classification: True positive.
• Affected entities: none identified, because no evidence of compromise was observed and no proxy,
firewall, or DNS logs showed link access by the user.
• Escalation rationale: the email was confirmed as delivered to the user, so the phishing exposure was
real even without a click event.
• Remediation actions: block hrconnex.thm on firewall and proxy, remove the email from other mailbox
locations, notify the targeted user, and add the sender to filters to prevent future delivery.
Matched evidence: incident report with true-positive classification and remediation plan
<img width="958" height="474" alt="image" src="https://github.com/user-attachments/assets/d886b6fa-5eb1-4945-baab-0156fce6b187" />

IOC inventory and response procedure
This is the complete artifact list and the end-to-end method you used to solve the ticket.
### INDICATORS OF COMPROMISE / EVIDENCE
Primary domain: hrconnex.thm
Full URL: https://hrconnex.thm/onboarding/15400654060/j.garcia
Sender: onboarding@hrconnex.thm
Targeted recipient: j.garcia@thetrydaily.thm
Subject: Action Required: Finalize Your Onboarding Profile
Message direction: inbound
Datasource: email
Telemetry host seen in SIEM: 10.10.156.209:8989
Shared source: eventcollector
Shared sourcetype: _json
Related context email: h.harris@thetrydaily.thm → j.carter@thetrydaily.thm
Subject: Unable to Receive Onboarding Email from hrconnex.thm
## STEP-BY-STEP PROCEDURE I FOLLOWED
1. Open the case and record alert ID, rule name, severity, incident type, and timestamps.
2. Expand the alert payload and extract sender, recipient, subject, direction, and full URL.
3. Search the suspicious domain in SIEM to find all correlated events.
4. Match the phishing event back to the case and capture any related business-context messages.
5. Validate source and sourcetype so the event set is coming from a consistent telemetry pipeline.
6. Check for downstream access evidence in proxy, firewall, and DNS logs; here, the final report
recorded none.
7. Classify the alert as true positive because delivery was confirmed.
8. Escalate for containment, recommend blocking and message removal, notify the user, and document
closure rationale.
