# System Design Scenario: Reconciliation System (Event-Driven Architecture)

## 📄 Prompt

**Scenario:**  
You’re working at a financial services company. One team comes to you and says:  
“We need a reconciliation system. Sometimes we receive data from partners, and it doesn't match our internal records.  
We need a way to process and flag mismatches.”

---

## ❓ Clarifying Questions & Stakeholder Responses
Questions from me to the AI acting as the client

### Q1: What do you mean by reconciliation? (me)
> “By reconciliation, I mean we get external reports or transactions from partners — like payment processors, logistics services, or banks — and we need to compare that with our internal system of record.

> For example, a partner might say:
> - ‘We paid you $1000 yesterday for invoice #ABC123’
> - But our system only shows $800 received

> So we need to flag mismatches, keep an audit trail, and allow internal teams to review or resolve them later.”

---

### Q2: Does your internal accounting department trigger this, or is it automated? (me)

> “Ideally it should be automated. But the accounting team does want control over the schedule — they might run it daily, weekly, or manually in some cases. The system should flag discrepancies, store them, and make them available for internal resolution. Bonus if it can alert us immediately when something’s obviously wrong.”

---

### Q3: How do you wish to be notified? (me)

> “For critical mismatches — like a high-value invoice that wasn’t paid, or a duplicate payment — we want a real-time alert (Slack or email). For lower-priority discrepancies — like small rounding issues or late payments — we’re fine reviewing those in a dashboard or report. Ideally, your system can: categorize discrepancies, push urgent alerts, and log everything to a system the finance team can access.”

---

## 🖼️ Architecture Diagram

![Reconciliation System Architecture](Accounts_Payable_system.drawio.png)

---

## 🧠 Interview Questions + Candidate Responses

### 1. What data is the Check Payment Lambda comparing?

**Answer:** This data comes from the already existing payment system, which is an internal financial system, where payments are made.  
If there is no clean API, then we would connect to the DB directly. I assume there's an API or shared DB available within AWS.

---

### 2. Why use SQS instead of triggering the Lambda directly from the time-based event or API Gateway?

**Answer:** While this could be direct, SQS provides value via retries and DLQ handling. If it’s a low-frequency task, direct invocation may suffice, but SQS gives us resilience and replayability.

---

### 3. Suppose an external financial system reports a duplicate payment. How do you prevent your system from triggering multiple alerts?

**Answer:** We should track a unique `checkPaymentId` in a DynamoDB table and reject duplicates.  
This lets us deduplicate regardless of event source (EventBridge or manual).

---

### 4. Where is the audit log for mismatches stored?

**Answer:** I’d store audit records in DynamoDB, with status and timestamps.  
This data could be exposed to the dashboard or separated depending on live sync needs.

---

### 5. What if a high-severity alert fails to send (e.g., Lambda or email fails)?

**Answer:** We should attach SQS + DLQ to the Notification Lambda and retry.  
Failed alerts can be stored in DynamoDB or S3 for reprocessing or manual resend.

---

### 6. Could this system handle multiple external sources?

**Answer:** Yes — this version notifies the accountant first.  
A second phase could automate partner-side notifications, but that's out of scope for now.

---

### 7. What if an accountant manually initiates reconciliation for a single client or invoice?

**Answer:** The management console would hit API Gateway → SQS or Lambda depending on setup. The same reconciliation flow can be reused.

---

### 8. Could other services subscribe to reconciliation results?

**Answer:** Yes — I’d use SNS to allow publish/subscribe patterns in the future if other systems needed hooks into reconciliation outcomes.

