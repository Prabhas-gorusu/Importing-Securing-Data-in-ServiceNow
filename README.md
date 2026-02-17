<div align="center">

#  Importing & Securing Data in ServiceNow

**A production-ready demonstration of enterprise data management, secure access control, and dynamic reporting on the ServiceNow platform.**

[![ServiceNow](https://img.shields.io/badge/Platform-ServiceNow-009BDE?style=for-the-badge&logo=servicenow&logoColor=white)](https://www.servicenow.com/)
[![ACL](https://img.shields.io/badge/Security-ACL%20Protected-success?style=for-the-badge&logo=shieldsdotio&logoColor=white)]()
[![Status](https://img.shields.io/badge/Project%20Status-Completed-brightgreen?style=for-the-badge)]()
[![Team](https://img.shields.io/badge/Team-LTVIP2026TMIDS76629-blueviolet?style=for-the-badge)]()

> *"Turning raw data into a secure, role-aware, enterprise-grade system — one transform map at a time."*

</div>

---

## 📖 Table of Contents

1. [Project Overview](#-project-overview)
2. [Key Features](#-key-features)
3. [Tech Stack](#️-tech-stack)
4. [Architecture & Workflow](#-architecture--workflow)
5. [Step-by-Step Implementation](#-step-by-step-implementation)
6. [Security Model](#-security-model)
7. [Testing & Validation](#-testing--validation)
8. [Risk Management](#️-risk-management)
9. [Team](#-team)
10. [Conclusion](#-conclusion)

---

## 🎯 Project Overview

This project tackles a real-world enterprise challenge: how do you **import employee training data** into ServiceNow, **link it to existing employee records**, and ensure the right people can see and edit only what they're supposed to?

The solution goes beyond a basic import — it demonstrates a full end-to-end pipeline using **Import Sets**, **Transform Maps**, **dot-walking** for dynamic field population, and a robust **ACL + role-based security** model. The result is a scalable, secure system fit for any enterprise HR or IT training workflow.

```
External XLSX Data  ──►  Import Set  ──►  Transform Map  ──►  Employee Training Records Table
                                                                         │
                                                              Dot-Walking (pulls Department)
                                                                         │
                                                              ACL Rules (Read / Write)
                                                                         │
                                                           HR Manager Role ◄──► sys_user
```

---

## ✨ Key Features

**Data Import & Mapping** — Structured employee training data is loaded from external XLSX files using ServiceNow's Import Sets and mapped to target table fields via Transform Maps. The entire import pipeline is reusable and scalable.

**Dot-Walking** — Rather than storing redundant data, the system uses ServiceNow's dot-walking capability on the `Employee` reference field to automatically pull the employee's `Department` directly from the `sys_user` table, keeping data clean and always in sync.

**Access Control Lists (ACLs)** — Two dedicated ACL rules govern the `u_employee_training_records` table: one for **Read** access and one for **Write** access. Both require the `HR Manager` role, ensuring unauthorized users are completely blocked.

**Role-Based Access Management** — A custom `HR Manager` role is created and assigned to relevant users, modules, and application menus, giving precise, auditable control over who interacts with training data.

**Custom Table & Fields** — The `Employee Training Records` table is purpose-built with `Training Name`, `Completion Date`, `Status` (Choice: *Completed / In Progress*), and `Employee` (Reference: `sys_user`).

**Impersonation Testing** — Security is verified by impersonating both an `HR Manager` user (full access confirmed) and a standard user (access correctly blocked).

---

## 🛠️ Tech Stack

| Category | Tools & Technologies |
|---|---|
| **Platform** | ServiceNow (Personal Developer Instance) |
| **Core Modules** | Import Sets, Transform Maps, ACL Editor, Roles & Permissions, System Security |
| **Developer Tools** | ServiceNow Studio, Dictionary Editor, Form Layout Designer |
| **Data Format** | Microsoft Excel (`.xlsx`) for source data |
| **Security Layer** | `security_admin` role elevation + ACL configuration |

---

## 🏗️ Architecture & Workflow

```
Phase 1: Ideation
    └── Define the problem: import, link, and secure employee training records

Phase 2: Requirement Analysis
    └── Identify needs: custom table, import sets, ACLs, HR Manager role

Phase 3: Design
    └── Architect the u_employee_training_records table and field schema

Phase 4: Development
    ├── Import data via Import Sets & Transform Maps
    ├── Configure dot-walking for Employee → Department
    ├── Create ACL rules (Read + Write)
    └── Define and assign the HR Manager role

Phase 5: Testing
    ├── Impersonate HR Manager → verify full access ✅
    └── Impersonate standard user → verify access blocked ✅

Phase 6: Conclusion
    └── Evaluate success and readiness for deployment
```

---

## 📋 Step-by-Step Implementation

### 1️⃣ Create the Custom Table

Navigate to **All → Tables → New** and create the table:

- **Label:** `employee training records`
- **Name:** `u_employee_training_records`

Then add these fields via the Dictionary Entries tab:

| Field Label | Type | Notes |
|---|---|---|
| Training Name | String | Free text name of the training |
| Completion Date | Date | When the training was completed |
| Status | Choice | `Completed` (c), `In Progress` (ip) |
| Employee | Reference | References the `sys_user` table |

> 💡 **Tip:** To add choices to Status — right-click the column → *Configure Dictionary* → add entries under the *Choices* related list.

---

### 2️⃣ Prepare Source Data

Create an XLSX spreadsheet with these four columns:

```
| Training Name | Completion Date | Status      | Employee       |
|---------------|-----------------|-------------|----------------|
| Python        | 12/10/2004      | completed   | manogna        |
| Java          | 25/10/2024      | in progress | manasa         |
| MERN          | 4/11/2023       | completed   | poojitha       |
| HTML          | 14/06/2022      | in progress | poojitha valli |
| DSA           | 13/20/2023      | completed   | Aswini         |
```

---

### 3️⃣ Import Data via Import Sets

Navigate to **All → System Import Sets → Load Data**, then:

1. Select **Create table**, set Label to `Employee Training`, Name to `u_employee_training`.
2. Upload your XLSX file as the source.
3. Click **Submit** — the ImportProcessor will stage your data.
4. Once complete, click **Create Transform Map**.

---

### 4️⃣ Configure the Transform Map

Set the **Target Table** to `u_employee_training_records`. ServiceNow will auto-map fields:

```
Source Field          →  Target Field
─────────────────────────────────────
u_training_name       →  u_training_name
u_status              →  u_status
u_employee            →  u_employee
u_completion_date     →  u_completion_date
```

Click **Submit**, then **Run Transform** → **Transform**.

---

### 5️⃣ Enable Dot-Walking for Department

Open the training records form → **Form Context Menu → Configure → Form Layout**. Move **Employee → Department** from Available to Selected, then **Save**. The Department field now auto-populates from the linked `sys_user` record — no extra data entry needed.

---

### 6️⃣ Create the HR Manager Role

Navigate to **All → Roles → New**, name it `Hr Manager`, then assign it to:

- The **sys_user (Users) module** visibility under System Security.
- The **Employee Training Records application menu**.

---

### 7️⃣ Configure ACLs

First, **elevate your role** to `security_admin` via the profile menu. Then go to **All → ACL → Create New ACL** and create two rules:

**ACL Rule 1 — Read Access**
```
Type:          record
Operation:     read
Name:          Employee Training Records [u_employee_training_records]
Decision:      Allow If
Requires role: Hr Manager
```

**ACL Rule 2 — Write Access**
```
Type:          record
Operation:     write
Name:          Employee Training Records [u_employee_training_records]
Decision:      Allow If
Requires role: Hr Manager
```

---

## 🔒 Security Model

The architecture follows the **principle of least privilege** — access is denied by default and only opened through explicit role assignment.

```
┌──────────────────────────────────────────────────┐
│              Security Decision Tree              │
│                                                  │
│    User attempts to access training records      │
│                       │                          │
│          Has Hr Manager role?                    │
│          /                       \               │
│        YES                        NO             │
│         │                          │             │
│   Allow Read +               Access Denied       │
│   Allow Write                (Table Hidden)      │
│                                                  │
│   Admin users bypass via "Admin overrides" flag  │
└──────────────────────────────────────────────────┘
```

---

## 🧪 Testing & Validation

| Test Case | Action | Expected Result | Status |
|---|---|---|---|
| HR Manager Access | Impersonate user with `Hr Manager` role | Records visible and editable | ✅ Pass |
| Unauthorized Access | Impersonate standard user without the role | Table not found, no results | ✅ Pass |
| Dot-Walking Accuracy | Open training record with linked employee | Department auto-populates | ✅ Pass |
| Transform Map Execution | Run transform on staged import data | 5 records inserted, 0 errors | ✅ Pass |
| ACL Enforcement | Verify read/write ACL conditions | Only role-bearing users can access | ✅ Pass |

---

## ⚠️ Risk Management

| Risk | Probability | Impact | Mitigation Strategy |
|---|---|---|---|
| Data import fails due to format mismatch | Medium | High | Validate XLSX structure before upload |
| ACLs accidentally restrict legitimate access | Low | Medium | Test all role permutations before deployment |
| Dot-walking fails to auto-populate | Medium | High | Verify Employee reference field maps to `sys_user` |

---

## 👩‍💻 Team

| Name | Role |
|---|---|
| **Gorusu Prabhas**               | Team Leader |
| **Venkata Sri Dharani Saladi**   | Developer |
| **Sagarapu Anitha**              | Developer |
| **Priyanka Sharmila Nallamilli** | Developer |

**Team ID:** `LTVIP2026TMIDS76629`

---

## 🏁 Conclusion

This project successfully delivers a secure, structured, and scalable solution for managing employee training data within ServiceNow. By combining **Import Sets** for automation, **dot-walking** for dynamic relational data retrieval, and a layered **ACL + role-based** security model, the system is ready for real-world enterprise deployment — all without a single line of custom server-side code.

---

<div align="center">

**Built with 💙 on the ServiceNow Platform**

*If you found this project helpful, consider giving it a ⭐ on GitHub!*

</div>
