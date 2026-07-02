# 💾 SaaS CRM Pipeline Analysis - Data Modeling & Database Logic

To support the automation rule described in AC 4 (Closed-Lost Rule) and visualized in the process flowchart, a relational database structure has been designed. The system tracks the relationship between Sales Representatives (`Users`), Potential Customers (`Leads`), and Logged Activity History (`Interactions`).

---

## 📊 Entity Relationship (ER) Table Structure

### 1. `Users` Table (Sales Team)
This table stores information about the internal sales team members who manage the incoming leads.

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| **User_ID (PK)** | Integer | Unique identifier for each sales representative |
| Name_Surname | Varchar | First and last name of the representative |
| Email | Varchar | Corporate email address |
| Role | Varchar | System role (e.g., Sales Agent, Team Lead, Admin) |

### 2. `Leads` Table (Potential Customers)
This table tracks all potential customers entering the sales pipeline. It includes a Foreign Key (`User_ID`) to assign each lead to a specific sales representative.

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| **Lead_ID (PK)** | Integer | Unique identifier for each potential customer |
| Name_Surname | Varchar | Customer's name and surname |
| Company | Varchar | Company name |
| Status | Varchar | Current pipeline status (e.g., New, In-Pipeline, Closed-Lost) |
| Last_Contact_Date| Date | The exact timestamp of the last communication/log |
| **User_ID (FK)** | Integer | Linked to `Users.User_ID` (Owner of the lead) |

### 3. `Interactions` Table (Activity History)
This table logs every individual communication attempt (calls, emails, WhatsApp messages) made with a customer. It maintains a Bire-Çok (One-to-Many) relationship with the `Leads` table.

| Column Name | Data Type | Description |
| :--- | :--- | :--- |
| **Interaction_ID (PK)**| Integer | Unique identifier for each interaction log |
| **Lead_ID (FK)** | Integer | Linked to `Leads.Lead_ID` (Which customer?) |
| Interaction_Date | Date | Timestamp of when the activity occurred |
| Interaction_Type | Varchar | Communication channel used (Call, WhatsApp, Email) |
| Notes | Text | Detailed summary entered by the representative |

---

## 🔍 Technical Automation Logic (Sample SQL Query)

The backend automated background system (cron job) executes the query below every morning to scan the database, detect leads with no activity for 3 days, and trigger systemic actions:

```sql
SELECT 
    Lead_ID, 
    Name_Surname, 
    User_ID 
FROM 
    Leads 
WHERE 
    Status = 'In-Pipeline' 
    AND Last_Contact_Date <= CURRENT_DATE - INTERVAL '3 days';

'''

---

## 🌐 6. API & Integration Analysis

When the automated background system updates a lead's status to 'Closed-Lost' (as defined in AC 4), it triggers an internal API integration to update related microservices and notify the sales ecosystem.

### 🔹 API Contract: Lead Status Internal Update
* **Endpoint:** `/api/v1/leads/{lead_id}/status`
* **HTTP Method:** `PATCH` (Used for partial updates)
* **Headers:** `Content-Type: application/json`

### 📤 Request Payload (JSON)
The automated system sends the following data structure to execute the status update:
```json
{
  "status": "Closed-Lost",
  "updated_by": "System_Automation_Cron",
  "reason": "No interaction logged for 3 consecutive days"
}
