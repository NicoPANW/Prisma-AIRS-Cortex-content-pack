# Prisma AIRS (AI Runtime Security) Integration

> **⚠️ DISCLAIMER:** This integration is provided strictly for research and Proof of Concept (POC) purposes. It is not officially supported or intended for use in production environments.

This integration connects to the Prisma AIRS API to fetch live AI security violations (Prompt Injections, Toxic Content, Malicious URLs, Data Leaks) and automatically ingest them as alerts (Issues) into Cortex XSIAM.

<p align="center">
  <img width="1844" height="839" alt="Screenshot 2026-05-06 at 13 36 50" src="https://github.com/user-attachments/assets/6f0ef166-85b2-4022-900f-b971c81dc18b" />
  <br><br>
  <img width="1847" height="839" alt="Screenshot 2026-05-06 at 14 31 56" src="https://github.com/user-attachments/assets/dc99305f-0606-477c-8dde-ce9fd5b12c52" />
  <br><br>
  <img width="1850" height="847" alt="Screenshot 2026-05-13 at 16 08 27" src="https://github.com/user-attachments/assets/8c8b5466-5781-4e47-a7d0-3803c3c49c08" />
</p>


---

## 1. Prerequisites and Setup Requirements

Because Cortex XSIAM treats fetched alerts as "Issues" and enforces a strict data schema, this integration requires specific Custom Fields, a Classifier, an Incoming Mapper, and a Custom Layout to function properly. If these are not configured, XSIAM will either drop the fetched alerts silently or render the data unreadable.

### A. Required Custom Fields
Instead of building the custom fields manually, you can automatically generate them by importing the provided JSON configuration file. 
* Navigate to `Settings > Configurations > Object Setup > Issues > Fields`.
* Click the **Import** button and upload `Prisma_AIRS_customfields.json` file. 
* This will instantly create all required fields with their correct data types.

### B. Required Classifier and Mapper
Instead of building the mapping logic from scratch, you can import the provided configuration to automatically route your data.
* Navigate to `Settings > Configurations > Object Setup > Issues > Classification & Mapping`.
* Import the provided Classifier `classifier-Prisma_AIRS_-_Classifier.json` file and Mapper `classifier-Prisma_AIRS_Mapper.json` file. 
* This will automatically set up the rules to route fetched alerts into the `Prisma AIRS Violation` issue type, and perfectly map the JSON payload keys (e.g., `airsprompt`, `airsthreatsnippets`) directly to your newly imported Custom Fields.

### C. Custom Layout and Layout Rules
To ensure the large JSON payloads and specific AIRS fields are easily readable by analysts without squishing or collapsing, you need to import the dedicated AIRS layout.
* **Layouts:** Go to `Settings > Configurations > Object Setup > Issues > Layouts`. Click **Import** and upload `layoutscontainer-Prisma_AIRS_Investigation_Layout.json` file. This layout is pre-configured with full-width sections to give massive fields like `AIRS Prompt` and `AIRS Threat Snippets` the room they need to expand.
* **Layout Rules:** Go to `Settings > Configurations > Object Setup > Issues > Layout Rules` and define a quick rule so XSIAM knows when to apply your new view: *(e.g., If Alert Name contains "AIRS Violation", use the imported layout)*.

### D. AIRS API Credentials
Create AIRS API credentials in SCM. Then in XSIAM, create a new credential in `Settings > Configurations > Integrations > Credentials` and fill in a name, username, and password accordingly.

### E. Importing the integration
* Navigate to `Settings > Data Sources or Integrations` 
* At top right click `Add new > Create Integration > + Import File` and select `Prisma_AIRS.yml` file
* This will import the integration

---

## 2. Fields to Populate (Instance Configuration)

When setting up the integration instance, fill out the following parameters:

* **TSG ID (Tenant Service Group ID):** Provide the corresponding TSG ID.
* **Credentials (Client ID, Client Secret):** Select the credentials profile you created in the prerequisites.
* **Classifier:** Import the classifier (`Prisma AIRS - Classifier`) imported earlier.
* **Mapper (incoming):** Import the mapper (`Prisma AIRS Mapper`) imported earlier.
* **Minimum Severity:** Select what the minimum severity threshold is to import an alert.

---

## 3. Playground Testing Commands

The integration includes several custom commands designed to help you verify data without waiting for the background fetching job. You can execute these in the XSIAM CLI/Playground:

* `!airs-reset-fetch`
  * **Use Case:** Clears the integration's deduplication memory. Use this if you want to force XSIAM to re-ingest old violations during testing.
* `!airs-get-fetch-sample`
  * **Use Case:** Fetches the data and prints the exact JSON payload directly to the screen *without* pushing it into the XSIAM incidents table. Highly useful for debugging schema errors or mapper misconfigurations.
* `!fetch-incidents`
  * **Use Case:** Forces an immediate ingestion run. The data will be pulled and sent to your Incidents queue silently. *Note: This command is a reserved XSOAR command and intentionally does not include the `airs-` prefix.*

---

## 4. API Investigation Commands

You can also pull raw data straight from the Prisma AIRS API for manual investigation:

* `!airs-get-violations time_interval=1 time_unit=hour`  
  Retrieves a list of violations.
* `!airs-get-threat-snippets scan_id="<id>"`  
  Retrieves the specific snippet that triggered the alert.
* `!airs-get-scan-content scan_id="<id>"`  
  Retrieves the entire raw user prompt.
* `!airs-get-session-transaction session_id="<id>" app_id="<id>" app_name="<name>" scan_id="<id>"`  
  Retrieves overarching metadata regarding the AI interaction.

---

## 5. Known issues / troublesooting


* In order to get new issues, you need to create them after you set up the integration. Note often, the very first issue takes time (about 30 minutes) to show up, while the subsequent ones show-up within a minute 
