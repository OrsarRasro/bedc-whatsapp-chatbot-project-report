# BEDC WhatsApp Customer-Service Chatbot

## Project implementation and readiness report

**Prepared:** 14 October 2026
**Implementer:** Digitank Technologies
**Service context:** Benin Electricity Distribution Company (BEDC)
**Status:** Operating AWS implementation with a newly deployed, authenticated
internal reporting dashboard; further production-readiness validation remains.

## Executive summary

The project began as a WhatsApp support chatbot and grew into an AWS-hosted
customer-service workflow combining approved BEDC knowledge, account-aware
tools, voice-note handling, operational notifications, and an internal
aggregate dashboard.

The current implementation runs in a single AWS Lambda function. It uses
Amazon Bedrock for conversational responses and knowledge-base retrieval,
alongside AWS services for messaging integration, data, document storage,
voice processing, notifications, and customer-service workflows. The chatbot
has in-code meter, finance, and general-support profiles, but these are routing
profiles inside the Lambda—not separate deployed agents or staffed teams.

The dashboard update was deployed and checked on 14 October 2026. Its page
loads, protected metrics require sign-in, and the authenticated API returns
valid seven- and 30-day summaries. The 30-day view included 21 historical
conversation totals; the seven-day view was zero because no summaries existed
in that date range. Earlier summaries do not contain topic labels, channels,
or sentiment, so those dimensions cannot be reconstructed from totals alone.

This report distinguishes implemented features from tested outcomes. The
project has not been certified as enterprise-ready, and the available checks
do not prove that every customer message is answered, that every escalation
reaches a staffed team, or that all voice-note experiences work consistently.

## Project history

The supplied implementation guide is version 3.1, dated July 2026 and marked
**Confidential — For Internal Use Only**. It records the progression from
initial WhatsApp/knowledge-base setup through AWS deployment, voice features,
and subsequent operational work. This public report uses it as historical
context; the confidential guide itself is not included.

The implementation work documented across the project includes:

1. WhatsApp Business messaging and webhook integration.
2. AWS Lambda and API Gateway deployment, with conversational responses
   grounded in BEDC knowledge-base material.
3. Voice-note transcription and generated audio replies.
4. Language and sentiment analysis where supported by the configured services.
5. Account-related tools protected by a verification flow.
6. Billing, payment, token, meter, fault-reporting, and invoice-related
   workflows.
7. An internal dashboard for aggregate enquiry trends and categories.

Some historical setup notes describe intended architecture or earlier
configuration. They should not be treated as a current infrastructure
inventory without checking the AWS account.

## Current architecture

At a high level, the request flow is:

1. A customer sends a WhatsApp text message or voice note.
2. Meta's WhatsApp webhook delivers the event through API Gateway to the
   Node.js 22.x Lambda application.
3. The application retrieves approved BEDC information from Amazon Bedrock
   Knowledge Bases and uses Amazon Nova Micro with tool configuration for
   supported actions.
4. Private account and billing operations use application tools and require
   the configured customer-verification flow before private information is
   returned.
5. Voice messages can be transcribed with Amazon Transcribe; generated audio
   responses use Amazon Polly. Amazon Comprehend provides language and
   sentiment analysis when applicable.
6. The application uses DynamoDB for conversation/session and aggregate
   records, S3 for supported document/audio storage, an RDS-compatible MySQL
   integration for account workflows, and SNS for configured notifications.
7. The Lambda serves the internal dashboard through the existing API Gateway
   integration; no separate dashboard frontend service is required.

The deployment separates compute and AI services in the Europe (Ireland)
region from configured data services in Africa (Cape Town). Exact resource
identifiers and account configuration are intentionally omitted from this
public report.

## Delivered capabilities

### Customer support and knowledge

- WhatsApp text interaction through the deployed webhook.
- Retrieval from an approved BEDC knowledge base for supported public
  information.
- Tool-assisted account, billing, payment, token, meter, and bill workflows.
- A cautious response path for unsupported knowledge-base questions, rather
  than presenting an unverified answer as fact.
- Meter, finance, and general-support routing profiles within the single
  Lambda application.

### Verification and customer data

- A customer-verification flow is required for private account information.
- Dashboard metrics are aggregate counts and do not return customer names,
  account numbers, phone numbers, or message contents.
- The dashboard uses a password stored in the existing AWS secret and
  time-limited secure session cookies.

These controls are implementation features, not a substitute for an
independent security, privacy, or regulatory assessment.

### Voice, documents, and notifications

- Voice-note transcription and generated audio responses are implemented.
- Invoice generation and short-lived bill-download links are implemented.
- The Lambda can send configured notifications for supported escalation or
  resolution workflows.

The availability and quality of each of these flows should be confirmed with
current end-to-end tests using non-production test accounts.

### Internal dashboard

- BEDC-branded, staff-facing aggregate report.
- Seven-day and 30-day reporting periods, with daily totals.
- New inbound enquiries are assigned rule-based topic, routing-profile,
  language, and channel dimensions; sentiment is recorded when available.
- Historical conversation totals are included where backfilled, but historical
  records without dimension data remain unclassified.
- Dashboard counters are written with an expiry attribute. Automatic deletion
  depends on DynamoDB TTL being enabled and should be verified against the
  account's current table configuration.

The dashboard categories are operational signals, not validated research
labels. The dashboard's routing profiles do not represent actual human teams.

## Deployment and verification evidence

The dashboard update was explicitly approved and deployed on 14 October 2026.
Checks performed after deployment confirmed:

- The Lambda update completed successfully and the deployed package hash
  matched the reviewed local deployment bundle.
- The dashboard page returned HTTP 200 and included the updated public-safe
  dashboard content.
- An unauthenticated request to the metrics endpoint returned HTTP 401.
- In an authenticated session, the seven-day metrics response was valid and
  totalled zero; the 30-day response was valid and totalled 21.
- The 30-day daily totals reconciled to 21: 1 on 15 September, 11 on
  21 September, and 9 on 28 September 2026.
- The dashboard rendered those daily points and reported that historical
  category labels were unavailable.
- Local JavaScript syntax and editor diagnostics reported no errors.

These checks verify the dashboard deployment and its metrics response. They
are not a fresh end-to-end WhatsApp test of all chatbot capabilities.

## Known limitations and open risks

1. **Message-delivery confidence:** There have been user reports of messages
   receiving no response. A post-update multi-user WhatsApp delivery test has
   not been completed, so resolution is not established by this report.
2. **Voice-note consistency:** Voice transcription and audio replies are
   implemented, but reported differences between users need repeatable
   end-to-end tests.
3. **Historical analytics:** Older records lack category, channel, and
   sentiment dimensions. Those values cannot be inferred reliably from
   aggregate totals.
4. **Escalation ownership:** An application notification is not proof that a
   staffed team received, acknowledged, and resolved a customer issue. Define
   an owner, response target, and fallback for every escalation path.
5. **Data retention:** Analytics records include an expiry attribute, but
   automatic cleanup requires TTL configuration. Verify TTL and agree on
   retention periods for all customer and voice data.
6. **New or ambiguous cases:** The bot does not autonomously learn verified
   answers from arbitrary new questions. Knowledge-base changes need an
   accountable review and publishing process.
7. **Production assurance:** No independent penetration test, load test,
   disaster-recovery exercise, comprehensive accessibility review, or
   end-to-end acceptance test across the full customer journey is represented
   here.
8. **Operational dashboard:** QuickSight is not part of the current dashboard
   deployment. Consider it only if BEDC needs managed BI capabilities,
   broader reporting, or established staff access governance.

## Recommended readiness plan

### 1. Verify customer journeys

Run an approved test matrix with multiple test users. Include greetings,
misspellings, follow-up context, public service questions, verified and
unverified account access, incorrect/expired/reused verification codes,
meter-error questions, bill and token workflows, and unknown questions.
Record whether each message is received, answered, safely declined, or
escalated.

### 2. Verify voice and document flows

Test voice notes from each affected user, varied audio quality and supported
languages, and both expected reply formats. Generate and download a test
invoice, then verify link expiry and that one customer's document cannot be
retrieved by another.

### 3. Make escalation operational

Name the staff owner and monitored channel for each escalation. Test delivery,
acknowledgement, customer follow-up, and failure handling. Distinguish a
ticket being created from a ticket being resolved.

### 4. Close security and privacy checks

Review least-privilege IAM, secret rotation, dashboard access and rate
limiting, sensitive-data logging, encryption, backup/recovery, and DynamoDB
TTL. Confirm data retention and deletion behavior for conversation,
analytics, invoice, and voice records with the relevant BEDC stakeholders.

### 5. Calibrate reporting

Use representative, consented test messages to validate the rule-based
categories. Show “not available” where a dimension was not recorded; do not
backfill invented labels. Agree on which staff decisions the report should
support before expanding it or moving to a BI platform.

### 6. Approve a controlled demo

Use synthetic customer records and a written acceptance checklist. Monitor
CloudWatch errors and messaging delivery during the demo. Keep a human support
contact available, and do not describe the system as fully autonomous or
enterprise-certified.

## Public-release and privacy note

This report intentionally excludes the confidential implementation guide,
source code, AWS account identifiers, resource names and ARNs, phone numbers,
API endpoints, secret references or values, customer records, and SQL test
data. It is a high-level project summary, not an operational deployment
manual. Share configuration details only through approved private channels.
