# Entra ID to SAP User Synchronization Tool

An ABAP utility designed to bridge the gap between Microsoft Entra ID (formerly Azure AD) and SAP SU01 User Master Records.

This tool allows SAP administrators to:

- Identify orphaned SAP user accounts
- Detect users disabled in Entra ID
- Lock or terminate inactive SAP users
- Synchronize user master data directly from Microsoft Graph
- Perform mass updates using standard SAP BAPIs

---

## Features

### Clean Up Tool

Compares active SAP dialog users against their Entra ID account status.

The tool can:

- Detect disabled Entra ID accounts
- Identify inactive SAP users based on last logon date
- Propose user locking
- Propose user termination
- Support administrator review before execution

### Data Update Tool

Retrieves current user information from Microsoft Entra ID and synchronizes the following SAP user attributes:

- First Name
- Last Name
- Initials
- Email Address
- Job Title
- Department
- Mobile Phone Number

### Mass Processing

Uses SAP standard APIs:

- `BAPI_USER_CHANGE`
- `BAPI_USER_LOCK`
- `BAPI_TRANSACTION_COMMIT`

No direct database updates are performed.

### Administrator Overrides

The ALV grid allows administrators to:

- Review proposed actions
- Select multiple users
- Override recommendations
- Execute approved changes in a single run

---

# 📋 Prerequisites

## SAP Requirements

The following SAP components and transactions must be available:

- S/4HANA or SAP NetWeaver ABAP Stack
- SE38
- SE41
- SM59
- STRUST
- SOAUTH2 or OA2C_CONFIG

## Microsoft Requirements

Administrative access to:

- Microsoft Entra ID
- Microsoft Graph API
- Application Registration Management

---

# ⚙️ Step 1: Microsoft Entra ID Configuration

To allow SAP to securely access Microsoft Graph, create an application registration.

## Create Application Registration

1. Open the Microsoft Entra Admin Center.
2. Navigate to:

   ```text
   App Registrations
   ```

3. Select:

   ```text
   New Registration
   ```

4. Create an application such as:

   ```text
   SAP_User_Sync
   ```

## Configure API Permissions

Navigate to:

```text
API Permissions
```

Add:

```text
Microsoft Graph
```

Select:

```text
Application Permissions
```

Grant:

```text
User.Read.All
```

Then click:

```text
Grant Admin Consent
```

## Create Client Secret

Navigate to:

```text
Certificates & Secrets
```

Create a new client secret.

Record the following values:

- Client ID
- Tenant ID
- Client Secret

They will be required for SAP OAuth configuration.

---

# 🔐 Step 2: SAP OAuth Configuration (SOAUTH2)

The application uses SAP's built-in OAuth framework.

Client secrets are **never stored in ABAP code or variants**.

---

## Create OAuth Profile

Open transaction:

```text
SOAUTH2
```

or

```text
OA2C_CONFIG
```

Create a new OAuth 2.0 Client Profile.

### Suggested Values

| Setting | Value |
|----------|----------|
| Profile Name | ENTRA_GRAPH_TOKEN |
| Grant Type | Client Credentials |
| Client ID | Entra Application ID |
| Client Secret | Entra Client Secret |
| Token Endpoint | https://login.microsoftonline.com/<TENANT-ID>/oauth2/v2.0/token |

---

## Configure Scope

Add the scope:

```text
https://graph.microsoft.com/.default
```

---

## Test OAuth Connectivity

Use the OAuth administration test functionality to request a token.

A successful test should return:

```text
access_token
```

without authentication errors.

---

## Client Secret Rotation

Whenever the Entra ID client secret expires:

1. Generate a new secret in Entra ID.
2. Open SOAUTH2.
3. Update the Client Secret.
4. Save.
5. Reactivate the profile.
6. Re-test token retrieval.

No ABAP code changes are required.

---

# 🌐 Step 3: SAP SM59 Destination Configuration

The report uses a dedicated Microsoft Graph destination.

---

## Create HTTP Destination

Transaction:

```text
SM59
```

Create a new destination:

| Setting | Value |
|----------|----------|
| Type | G |
| Name | ENTRA_GRAPH_API |
| Host | graph.microsoft.com |
| Service No. | 443 |
| Path Prefix | / |

---

## Logon & Security Tab

Configure:

| Setting | Value |
|----------|----------|
| SSL | Active |
| SSL Client Certificate | ANONYM SSL Client |

---

## SSL Certificates

Ensure Microsoft root and intermediate certificates are trusted in SAP.

Transaction:

```text
STRUST
```

Import certificates required for:

- graph.microsoft.com
- login.microsoftonline.com

Test the destination successfully before proceeding.

---

# 💻 Step 4: ABAP Installation

## Create Report

Open:

```text
SE38
```

Create:

```text
Z_ACTIVE_USER_CHECK
```

Paste the supplied ABAP source code.

---

## Update Tenant ID

Modify the tenant constant:

```abap
CONSTANTS:
  c_tenant TYPE string VALUE 'YOUR-TENANT-ID'.
```

Replace with your own Microsoft Entra Tenant ID.

---

## Verify Destination Configuration

The report assumes:

```abap
p_dstgrp = 'ENTRA_GRAPH_API'
```

Where:

- `ENTRA_GRAPH_API` = HTTP destination in SM59

---

## Activate Program

Activate all dependent objects.

Resolve all syntax issues before transport.

---

# 🎨 Step 5: Create ALV GUI Status (SE41)

The report requires a custom PF-STATUS.

Open:

```text
SE41
```

Program:

```text
Z_ACTIVE_USER_CHECK
```

Status:

```text
PROCESSUSER
```

---

## Application Toolbar Buttons

Create the following function codes:

| Function Code | Icon Text | Purpose |
|----------|----------|----------|
| PROCESS | Process Users | Execute selected actions |
| SEL_ACT | Select Actionable | Select proposed rows |
| DSEL_ALL | Deselect All | Clear selections |
| SET_LOCK | Set to Lock | Force lock action |
| SET_TERM | Set to Terminate | Force terminate action |
| SET_UPDT | Set to Update | Force data update |
| SET_NONE | Set to None | Ignore selected rows |
| &XXL | Export | Export to Spreadsheet |

Save and activate the status.

---

# 🚀 Usage Guide

## Start the Report

Run:

```text
Z_ACTIVE_USER_CHECK
```

---

## Tool Modes

### Clean Up Tool

Used for lifecycle management.

Checks:

- Entra ID account status
- SAP validity dates
- SAP lock status
- Last SAP logon date

Possible recommendations:

- Lock User
- Terminate User
- No Action

---

### Data Update Tool

Used for user master synchronization.

Updates:

- First Name
- Last Name
- Initials
- Phone Number
- Email
- Job Title
- Department

Updates are performed using standard SAP APIs.

---

## Processing Workflow

1. Execute report.
2. Review proposals.
3. Select actionable users.
4. Adjust recommendations if necessary.
5. Press **Process Users**.
6. Review results.

---

## 🔒 Security Best Practices

* Never store Microsoft Entra ID client secrets inside ABAP source code.
* Never store client secrets in SAP variants.
* Store all OAuth credentials in SAP's OAuth 2.0 framework (`SOAUTH2` / `OA2C_CONFIG`).
* Restrict maintenance authorizations for OAuth profiles to a small group of SAP Basis administrators.
* Periodically rotate Microsoft Entra ID client secrets according to corporate security policies.