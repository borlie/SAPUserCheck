Here is a complete, professionally structured `README.md` file tailored for your GitHub repository. It covers the Entra ID setup, the SAP SM59 configuration, the ABAP installation, and the SE41 GUI status requirements.

You can copy and paste this directly into your repository!

***

# Entra ID to SAP User Synchronization Tool

An ABAP utility designed to bridge the gap between Microsoft Entra ID (formerly Azure AD) and SAP SU01 User Master Records. This tool allows SAP administrators to easily identify orphaned accounts, lock/terminate inactive users, and synchronize live HR attributes (like Job Title, Department, and Email) directly from the Microsoft Graph API.

## Features
* **Clean Up Tool:** Compares active SAP dialog users against their Entra ID `accountEnabled` status. Proposes locking or terminating users based on Azure status and SAP inactivity (days since last logon).
* **Data Update Tool:** Fetches active users from Entra ID and securely synchronizes their First Name, Last Name, Initials, Phone Number, Email, Job Title, and Department directly into SAP SU01.
* **Mass Processing:** Utilizes `BAPI_USER_CHANGE` and `BAPI_USER_LOCK` for safe, standard-compliant mass updates.
* **Administrator Overrides:** ALV grid includes interactive buttons to override proposed actions before committing changes to the database.

---

## 📋 Prerequisites
1. **SAP NetWeaver** (ABAP stack) with access to transactions `SE38`, `SE41`, `SM59`, and `STRUST`.
2. **Microsoft Entra ID** tenant with administrator access to register an application.

---

## ⚙️ Step 1: Microsoft Entra ID Configuration

To allow SAP to read user data, you must register an application in Entra ID and grant it the appropriate Graph API permissions.

1. Go to the **Microsoft Entra admin center** > **App registrations** > **New registration**.
2. Name it something like `SAP_User_Sync`.
3. Go to **API permissions** > **Add a permission** > **Microsoft Graph** > **Application permissions**.
4. Search for and select **`User.Read.All`**.
5. Click **Grant admin consent for [Your Tenant]**.
6. Go to **Certificates & secrets** > **New client secret**. Copy the **Value** immediately (you will not be able to see it again).
7. Go to the **Overview** page and copy your **Application (client) ID** and **Directory (tenant) ID**.

---

## 🌐 Step 2: SAP SM59 Destinations

The ABAP program relies on two standard HTTP connections to communicate with Microsoft Graph. 

### 1. The Token Destination
This endpoint is used to exchange your Client ID and Secret for an OAuth2 Bearer Token.
* **Transaction:** `SM59`
* **Create New:** Type `G` (HTTP Connection to External Server)
* **Name:** `ENTRA_GRAPH_TOKEN`
* **Host:** `login.microsoftonline.com`
* **Port:** `443`
* **Path Prefix:** `/`
* **Logon & Security Tab:** Set SSL to **Active**. Set SSL Certificate to **ANONYM SSL Client** (or your default).

### 2. The API Destination
This endpoint handles the actual `$batch` JSON requests to query user data.
* **Create New:** Type `G` 
* **Name:** `ENTRA_GRAPH_API`
* **Host:** `graph.microsoft.com`
* **Port:** `443`
* **Path Prefix:** `/`
* **Logon & Security Tab:** Set SSL to **Active**. Set SSL Certificate to **ANONYM SSL Client**.

*(Note: Ensure that the root certificates for `microsoftonline.com` and `graph.microsoft.com` are imported into your SAP system via transaction `STRUST` so the SSL handshake succeeds).*

---

## 💻 Step 3: ABAP Installation

1. Open transaction **`SE38`**.
2. Create a new executable program named **`Z_ACTIVE_USER_CHECK`**.
3. Paste the provided ABAP code into the editor.
4. **Important Code Update:** Locate the `CONSTANTS` block near the top of the code and replace `c_tenant` with your actual Entra ID **Directory (tenant) ID**.
   ```abap
   CONSTANTS:
     c_tenant TYPE string VALUE 'YOUR-TENANT-ID-GOES-HERE'.
   ```
5. Save and Activate.

---

## 🎨 Step 4: Create the ALV GUI Status (SE41)

Because the ALV grid relies on custom buttons for administrator overrides, you must create a custom PF-STATUS.

1. Open transaction **`SE41`**.
2. Enter Program: **`Z_ACTIVE_USER_CHECK`** and Status: **`PROCESSUSER`**. Click Create/Change.
3. Expand the **Application Toolbar** and add the following Function Codes:

| Function Code | Icon | Icon Text | Info. Text |
| :--- | :--- | :--- | :--- |
| `PROCESS` | `ICON_EXECUTE_OBJECT` | Process Users | Execute BAPI Updates |
| `SEL_ACT` | `ICON_SELECT_ALL` | Select Actionable | Select proposed rows |
| `DSEL_ALL` | `ICON_DESELECT_ALL` | Deselect All | Clear all selections |
| `SET_LOCK` | `ICON_LOCKED` | Set to Lock | Override: Lock User |
| `SET_TERM` | `ICON_CANCEL` | Set to Term. | Override: Terminate |
| `SET_UPDT` | `ICON_WRITE_FILE` | Set to Update | Override: Update Data |
| `SET_NONE` | `ICON_TEST` | Set to None | Override: Ignore |
| `&XXL` | `ICON_EXPORT` | Export | Export to Spreadsheet |

4. Save and Activate the PF-STATUS.

---

## 🚀 Usage Guide

1. Run the report (`Z_ACTIVE_USER_CHECK`).
2. **First Run Configuration:** Expand the "API Configuration Parameters" block. Enter your Entra ID **Client ID** and **Client Secret**. Save this screen state as an SAP Variant so you don't have to type it every time.
3. Select your desired tool:
   * **Clean Up Tool:** Evaluates user lock status, Entra ID enablement, and SAP inactivity (`trdat`).
   * **Data Update Tool:** Fetches live Entra ID attributes and prepares `SU01` address updates.
4. Use the `Select Actionable` button to highlight proposed changes.
5. Review the grid, make any manual overrides using the toolbar buttons, and click **Process Users**.

---

### Disclaimer
*This code executes `BAPI_USER_CHANGE` and modifies live user master data. Always thoroughly test in a Development/Quality Assurance SAP environment before executing in Production.*

***

Would you like me to add anything else to this, such as a section on how to create the Selection Screen texts (Text Elements) in SE38?
