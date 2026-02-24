# CasePilot-AI-Service-Copilot-for-Salesforce-Service-Cloud

# CasePilot — AI Service Copilot for Salesforce Service Cloud

CasePilot is a Salesforce Service Cloud prototype that adds AI-powered case summarization and priority classification to improve triage, routing, and SLA operations. It demonstrates how to integrate an LLM with Salesforce using Apex + REST, automate workflows with Flow/Assignment Rules, and provide operational visibility with dashboards.

## What it does
- **AI Case Summary:** Generates a concise summary for each Case and stores it in `AI_Summary__c`.
- **AI Priority Prediction:** Classifies or scores urgency and stores it in `Suggested_Priority__c` / `Priority_Score__c`.
- **Auto Routing:** Uses Flow + Assignment Rules to route cases based on predicted priority and business rules.
- **SLA Monitoring:** Provides reports/dashboards to track volume, SLA risk, and resolution performance.

## Architecture
1. Case created/updated in Salesforce  
2. **Flow** triggers and invokes **Apex** (Invocable/Queueable)  
3. Apex makes a **REST callout** to an AI endpoint (OpenAI or a proxy service)  
4. AI returns `{summary, priority}`  
5. Salesforce updates Case fields and triggers **Assignment Rules**  
6. Dashboards/Reports show SLA + operational metrics

> If direct callouts to OpenAI are not allowed, use a small proxy API (Node/Flask) that forwards requests securely.

## Data Model (Custom Fields on Case)
- `AI_Summary__c` (Long Text Area) — AI-generated summary
- `Priority_Score__c` (Number) — AI urgency score (e.g., 0–100)
- `Suggested_Priority__c` (Picklist) — {Low, Medium, High, Critical}
- (Optional) `AI_Confidence__c` (Percent)

## Setup (Salesforce Dev Org)
1. Create a free Salesforce Developer Org.
2. Enable **Service Cloud** (if available in your org features).
3. Create the Case custom fields listed above.
4. Add fields to the **Case Lightning Record Page**.
5. Create:
   - A **Flow** that runs on Case create/update
   - Assignment Rules for routing by `Suggested_Priority__c`
6. Deploy Apex classes from `/force-app/main/default/classes`.

## AI Integration
### Option A: Direct REST callout (Apex → AI API)
- Configure a **Named Credential** and Remote Site Setting.
- Apex sends Case `Subject` + `Description` and receives JSON:
```json
{ "summary": "…", "priority": "High", "score": 87 }
