# NextGen Electronics Sales Tracker Documentation

## 1. Overview and Purpose 🚀

| Detail | Description |
| :--- | :--- |
| **App Name** | NextGen Electronics Sales Tracker |
| **Goal** | This application is designed to efficiently track and manage sales **Leads** and **Opportunities** for the NextGen Electronics sales team, enabling efficient follow-up and accurate report generation. |
| **Context** | **NextGen Electronics** is a pseudo company used solely for the context of this application build. |
| **Target Audience** | Sales Representatives, Sales Managers, and System Administrators. |

---

## 2. Getting Started (For End Users) 🧑‍💻

### Accessing the App

1. Click on the **App Launcher** (the $\text{9-dot}$ icon) on the top left corner of the Salesforce screen.
2. Search for `NextGen Electronics`.
3. Click on the **NextGen Electronics** app to open it.

### Key Tabs

| Tab Name | Purpose | Notes |
| :--- | :--- | :--- |
| **CustomLeads** | Contains all active and historical Lead records. | **Note:** This is a custom object to differentiate it from the standard Salesforce Lead object. |
| **CustomOpportunities** | Contains all sales Opportunity records being pursued or closed. | **Note:** This is a custom object to differentiate it from the standard Salesforce Opportunity object. |
| **Reports** | Access to standard and custom reports for sales tracking. | Includes the core **Opportunities With Leads** report. |

---

## 3. Core Functionality & Business Processes ⚙️

### 3.1. Lead Management: Creating a New CustomLead

Use the following steps to log a new potential customer or prospect.

1.  Navigate to the **CustomLeads** tab within the app.
2.  Click the **New** quick action button.
3.  Enter the Name of the lead in the **CustomLead Name** field.
4.  Enter the email address in the **Email** field. (***Required*** - enforced by a Validation Rule)
5.  **Optional Fields and Best Practice:**
    * **Lead Source:** Select the origin of the lead (e.g., Website, Referral, Cold Call).
    * **Phone:** Enter a direct phone number.
    * **Lead Score:** Enter a number between $\text{0}$ and $\text{100}$. This score indicates the likelihood of conversion.
    * **Lead Status:** Set to a value other than `Converted`.
6.  Click **Save** or **Save & New**.

### 3.2. Lead Conversion and Opportunity Creation

An Opportunity can be created in one of two ways:

#### A. Automatic Creation (Recommended)

To automatically create a `CustomOpportunity` from a `CustomLead`:

1.  Open the `CustomLead` record you wish to convert.
2.  Change the value of the **Lead Status** field to **Converted**.
3.  Click **Save**.
4.  A **CustomOpportunity** record will be automatically generated and linked to this Lead via the **Create Opportunity from Converted Lead** flow.

#### B. Manual Creation

1.  Navigate to the **CustomOpportunities** tab.
2.  Click the **New** quick action button.
3.  Enter the name of the Opportunity in the **CustomOpportunity Name** field (**Required**).
4.  **Best Practice:** Select the corresponding Lead record in the **CustomLead** lookup field (if the record exists).
5.  **Optional Fields:**
    * **Stage:** Select the current stage of the sales process.
    * **Close Date:** Enter the expected close date.
6.  Click **Save** or **Save & New**.

### 3.3. Reporting

1.  Navigate to the **Reports** tab.
2.  The standard report is **Opportunities With Leads**.
    * **Purpose:** Tracks all **CustomOpportunity** records that were generated *automatically* from a converted **CustomLead** record.
3.  To create a new report, click the **New Report** button and select the appropriate report type.

---

## 4. Technical Details and Administration 🛠️

This section is intended for system administrators and developers.

### 4.1. Custom Objects

| Label | API Name | Purpose |
| :--- | :--- | :--- |
| CustomLeads | `CustomLead__c` | Stores prospect and initial inquiry records. |
| CustomOpportunities | `CustomOpportunity__c` | Stores qualified sales deals and pipeline records. |

### 4.2. Key Custom Fields

#### CustomLeads Object (`CustomLead__c`)

| Field Label | Data Type | API Name | Notes |
| :--- | :--- | :--- | :--- |
| Lead Source | Picklist | `Lead_Source__c` | |
| Email | Email | `Email__c` | Required field. |
| Phone | Phone | `Phone__c` | |
| Lead Status | Picklist | `Lead_Status__c` | The value `Converted` triggers Flow 1. |
| Lead Score | Number(3,0) | `Lead_Score__c` | Range $\text{0-100}$. |

#### CustomOpportunities Object (`CustomOpportunity__c`)

| Field Label | Data Type | API Name | Notes |
| :--- | :--- | :--- | :--- |
| Stage | Picklist | `Stage__c` | |
| CustomLead | Lookup(CustomLead) | `CustomLead__c` | Relationship to the Lead it originated from. |
| Closed Date | Date | `Closed_Date__c` | Date the Opportunity was won/lost. |

### 4.3. Automation and Logic (Record-Triggered Flows)

| Flow Name | Trigger Object | Functionality |
| :--- | :--- | :--- |
| **Create Opportunity from Converted Lead** | `CustomLead__c` (on Update) | Creates a linked `CustomOpportunity__c` record when the **Lead Status** is set to `Converted`. |
| **Set Closed Date to Today** | `CustomOpportunity__c` (on Update) | Sets the **Closed Date** field to the current date (`TODAY()`) when the **Stage** field is updated to any 'Closed' status (`Closed Won` or `Closed Lost`). |

### 4.4. Validation Rules

#### CustomLeads Object

| Rule Name | Field (API Name) | Functionality |
| :--- | :--- | :--- |
| `Email_Cannot_Be_Blank` | `Email__c` | Prevents saving the record if the **Email** field is left blank. |

#### CustomOpportunities Object

| Rule Name | Field (API Name) | Functionality |
| :--- | :--- | :--- |
| `Closed_Date_Cannot_Be_a_Future_Date` | `Closed_Date__c` | Prevents saving the record if the **Closed Date** field is set to a future date. |

## 4.5. Duplicate Management (Data Quality) 🛡️

To maintain high data quality and prevent redundant outreach, the application includes custom duplicate management rules on the `CustomLead` object.

### Matching Rule: NextGen Lead Match

This rule defines how two `CustomLead` records are determined to be potential duplicates.

| Matching Criteria | Explanation |
| :--- | :--- |
| **Formula** | $\text{(CustomLead: Name ExactMatchBlank = FALSE) AND ((CustomLead: Email ExactMatchBlank = TRUE) OR (CustomLead: Phone Fuzzy: PhoneMatchBlank = FALSE)}$ |
| **Logic** | Records are considered a match if the **Name** is an exact match (and not blank), *AND* if either the **Email** is an exact match (and not blank), *OR* the **Phone** is a fuzzy match (and not blank). |

### Duplicate Rule: Prevent Duplicate Leads

| Rule Name | Action | Description |
| :--- | :--- | :--- |
| **Prevent Duplicate Leads** | **Alert** and **Report** | This rule utilizes the **NextGen Lead Match** criteria. It alerts users when creating or editing a record that potentially matches an existing lead and prevents the user from saving the duplicate record (based on the rule's settings). |
