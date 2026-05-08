# 🛡️ Investigating Windows Event Logs for Security Incidents

A hands-on project demonstrating how to access, filter, parse, and correlate Windows Event Logs using native Windows tooling — **Event Viewer**, **PowerShell**, and **Log Parser** — to detect and investigate potential security incidents.

---

## 🎯 Project Overview

Windows Event Logs are one of the most important data sources available to a SOC analyst. They record everything from successful and failed authentication attempts to privilege escalation and system errors — and being able to interrogate them quickly with native tools is a fundamental skill.

This project walks through five practical exercises focused on credential-based attack detection (Event IDs **4624** and **4625**), demonstrating the full investigation workflow: **access → filter → export → parse → correlate**.

---


## 📋 Pre-requisites

- Basic understanding of the Windows operating system
- Familiarity with the command-line interface
- Administrative privileges on the Windows machine
  
---

## 🖥️ Lab Environment

- **Operating System:** Windows 11 (running on a VMware Fusion virtual machine)
- **Privileges:** Local administrator account
- **Tools:** Event Viewer, PowerShell 5.1+, Log Parser 2.2

---

## 🧰 Tools Used

| Tool | Purpose |
|---|---|
| **Event Viewer** | Built-in GUI for browsing and filtering Windows Event Logs |
| **PowerShell** | Command-line shell and scripting language used to automate log analysis |
| **Log Parser 2.2** | Microsoft utility for querying log files using SQL-like syntax |

### Installing Log Parser

1. Download Log Parser 2.2 from the [Microsoft Download Center](https://www.microsoft.com/en-us/download/details.aspx?id=24659).

   <br><br>

<p align="center">
  <img src="https://github.com/user-attachments/assets/82919d4b-7a7a-42ad-a94c-5e138e7d4825" alt="Log Parser 2.2 download page on Microsoft Download Center" width="700"><br>
  <em>Figure 1: Log Parser 2.2 download page on the Microsoft Download Center</em>
  
<p align="center">
  <img src="https://github.com/user-attachments/assets/32461cc8-5e13-4f15-b094-59ef6c4e502a" alt="Log Parser 2.2 download page on Microsoft Download Center" width="700"><br>
  <em>Figure 2: Run the installer with administrative privileges and follow the on-screen prompt</em>
  
<br>
 
<p align="center">
  <img src="https://github.com/user-attachments/assets/81d95b06-876d-478e-b24f-5f9e5f35eb15" alt="Log Parser 2.2 Setup Wizard completion screen" width="700"><br>
  <em>Figure 3: Log Parser 2.2 installation complete</em>
</p>

<br>
2. Click through the wizard and click **Finish** when complete.

<br><br>


<p align="center">
  <img src="https://github.com/user-attachments/assets/f5b7c80b-ae45-4c1b-a078-ecd72fc72cfe" alt="Log Parser 2.2 folder visible in Program Files (x86)" width="700"><br>
  <em>Figure 4: Log Parser 2.2 installation directory in Program Files (x86)</em>
</p>
<br>
4. By default, Log Parser installs to `C:\Program Files (x86)\Log Parser 2.2\`.
<br>

> 💡 **Note:** Log Parser is officially unsupported (last updated in 2005) but still works reliably on modern Windows. PowerShell's `Get-WinEvent` is its modern equivalent.

---

## 🔬 Exercises

### Exercise 1 — Accessing Windows Event Logs using Event Viewer

**🎯 Objective:** Learn how to access and navigate Windows Event Logs using Event Viewer.

**Steps:**

1. **Open Event Viewer** — Press `Win + R`, type `eventvwr.msc`, and press `Enter`.
2. **Navigate the log categories** — Expand **Windows Logs** to reveal the five primary categories: **Application**, **Security**, **Setup**, **System**, and **Forwarded Events**.
3. **Explore the Security log** — Click **Security** under **Windows Logs** and review the events for authentication and access activity.

<p align="center">
  <img src="https://github.com/user-attachments/assets/85a32b35-2313-4bef-bb71-a36995c0634d" alt="Event Viewer showing the Security log" width="700"><br>
  <em>Figure 5: Event Viewer displaying the Security log under Windows Logs</em>
</p>


**Expected Output:** You should be able to navigate the various event logs and identify different event types — particularly authentication-related events in the Security log.

---

### Exercise 2 — Filtering and Exporting Event Logs

**🎯 Objective:** Learn how to filter and export event logs for detailed analysis.

**Steps:**

1. **Open Event Viewer** and select the **Security** log.
2. **Apply a filter for failed logins:**
   - Right-click **Security** and select **Filter Current Log...**
   - In the filter dialog, enter `4625` in the **Event ID** field.
   - Click **OK** to apply the filter.

<p align="center">
  <img src="https://github.com/user-attachments/assets/96d652a8-d3ca-4d82-af78-358010d578c9" alt="Event Viewer filtered to show only Event ID 4625 failed login events" width="700"><br>
  <em>Figure 6: Security log filtered for Event ID 4625 — failed login attempts</em>
</p>

3. **Export the filtered logs:**
   - Right-click the filtered view and select **Save Filtered Log File As...**
   - Save the file as `FailedLogins.evtx`.

<p align="center">
  <img src="https://github.com/user-attachments/assets/dd29bfa9-6210-484e-892f-40080875d078" alt="Save As dialog exporting filtered events as FailedLogins.evtx" width="700"><br>
  <em>Figure 7: Exporting the filtered Security log as FailedLogins.evtx</em>
</p>


**Expected Output:** A `FailedLogins.evtx` file containing only Event ID 4625 entries, ready for further analysis.

> ⚠️ **Troubleshooting — No Event ID 4625 entries?** Windows PIN authentication does not generate Event ID 4625 failed login events, so if your account uses a PIN you may find the Security log has no failed login entries at all. The workaround is to deliberately generate them using a separate local account:
> 1. Create a new local user in Command Prompt (run as Administrator):
>    ```cmd
>    net user TestUser Password123! /add
>    ```
> 2. Attempt to log in as `TestUser` several times using the **wrong** password — each failed attempt will generate an Event ID 4625 entry.
> 3. Log in once with the **correct** password to generate an Event ID 4624 entry.
> 4. Return to Event Viewer and filter/export as described above.

> 💡 **Tip for Exercise 5:** While you're here, repeat steps 2–3 with **Event ID 4624** and save as `SuccessfulLogins.evtx` — you'll need this file for the correlation exercise later.

---

### Exercise 3 — Parsing Event Logs using Log Parser

**🎯 Objective:** Use Log Parser to extract and analyse structured data from event logs.

**Steps:**

1. **Open Command Prompt** as Administrator and navigate to the Log Parser installation directory:
   ```cmd
   cd "C:\Program Files (x86)\Log Parser 2.2"
   ```

2. **Run the following query** to extract key fields from `FailedLogins.evtx` into a CSV file:
   ```cmd
   LogParser.exe "SELECT TimeGenerated, EventID, EventTypeName, Message INTO FailedLogins.csv FROM FailedLogins.evtx" -i:EVT -o:CSV
   ```

3. **Open the resulting `FailedLogins.csv` file** in Excel, Notepad, or any CSV viewer to review the extracted data.

<p align="center">
  <img src="https://github.com/user-attachments/assets/0b4a4774-7b52-47d7-84b7-c968dadfdd91" alt="Log Parser command output in CMD alongside the resulting CSV opened in Excel" width="700"><br>
  <em>Figure 8: Log Parser extracting failed login data into CSV — command output and resulting file</em>
</p>


**Expected Output:** A CSV file containing the timestamp, Event ID, event type name, and full message for each failed login attempt — ready for filtering, sorting, or pivoting in spreadsheet software.

---

### Exercise 4 — Analysing Event Logs using PowerShell

**🎯 Objective:** Use PowerShell to automate the analysis of Windows Event Logs.

**Steps:**

1. **Open PowerShell** as an administrator (right-click → Run as Administrator).

2. **List all failed login attempts** using `Get-WinEvent`:
   ```powershell
   Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} | Format-Table TimeCreated, Id, Message -AutoSize
   ```

3. **Save the output to a text file** for offline analysis:
   ```powershell
   Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4625} | Format-Table TimeCreated, Id, Message -AutoSize | Out-File -FilePath FailedLogins.txt
   ```

<p align="center">
  <img src="https://github.com/user-attachments/assets/70919115-b514-49ec-b651-5652cbc9fb8e" alt="PowerShell console showing Get-WinEvent output listing failed login attempts" width="700"><br>
  <em>Figure 9: PowerShell listing failed login attempts via Get-WinEvent and saving to file</em>
</p>

**Expected Output:** A formatted list of failed login attempts in the PowerShell console, plus a saved `FailedLogins.txt` file containing the same output.

---

### Exercise 5 — Correlating Events for Comprehensive Analysis

**🎯 Objective:** Correlate failed and successful login events to identify potential credential-based attacks.

> ⚠️ **Pre-requisite:** Before running the correlation, you need two exported `.evtx` files in the same folder:
> - `FailedLogins.evtx` — exported in Exercise 2 (Event ID **4625**)
> - `SuccessfulLogins.evtx` — repeat Exercise 2's filter/export steps with Event ID **4624**

**Steps:**

1. **Open Event Viewer** and select the **Security** log.
2. **Identify the two relevant Event IDs:**
   - **Event ID 4624** — *An account was successfully logged on*
   - **Event ID 4625** — *An account failed to log on*

3. **Extract failed logins to CSV** using Log Parser in PowerShell:
   ```powershell
   & "C:\Program Files (x86)\Log Parser 2.2\LogParser.exe" "SELECT TimeGenerated, EXTRACT_TOKEN(EXTRACT_TOKEN(Message, 1, 'Account Name:'), 0, 'Account Domain:') AS Username INTO temp_failed.csv FROM FailedLogins.evtx WHERE EventID=4625" -i:EVT -o:CSV
   ```

4. **Extract successful logins to CSV** using Log Parser in PowerShell:
   ```powershell
   & "C:\Program Files (x86)\Log Parser 2.2\LogParser.exe" "SELECT TimeGenerated, EXTRACT_TOKEN(EXTRACT_TOKEN(Message, 1, 'Account Name:'), 0, 'Account Domain:') AS Username INTO temp_success.csv FROM SuccessfulLogins.evtx WHERE EventID=4624" -i:EVT -o:CSV
   ```

5. **Correlate the two CSV files** using PowerShell to produce `CorrelatedLogins.csv`:
   ```powershell
   $failed = Import-Csv "temp_failed.csv"
   $success = Import-Csv "temp_success.csv"

   $results = foreach ($f in $failed) {
       $matches = $success | Where-Object { $_.Username.Trim() -eq $f.Username.Trim() }
       foreach ($m in $matches) {
           [PSCustomObject]@{
               FailedTime  = $f.TimeGenerated
               SuccessTime = $m.TimeGenerated
               Username    = $f.Username.Trim()
           }
       }
   }

   $results | Export-Csv -Path CorrelatedLogins.csv -NoTypeInformation
   ```

6. **Review `CorrelatedLogins.csv`** for patterns — particularly successful logins that follow multiple failed attempts from the same user, which is a classic indicator of brute-force or password-spray activity.

<p align="center">
  <img src="https://github.com/user-attachments/assets/a5b6fed8-4c8e-4cc5-b858-cbe958529dc7" alt="CorrelatedLogins.csv open in Excel showing FailedTime, SuccessTime and Username columns" width="700"><br>
  <em>Figure 10: CorrelatedLogins.csv pairing failed and successful login events by username</em>
</p>

**Expected Output:** A CSV file pairing failed and successful login attempts by username — providing visibility into potential credential attacks where an attacker eventually gained access after repeated failures.

> 💡 **What to look for in the output:**
> - The same username appearing in both `FailedTime` and `SuccessTime` columns
> - A short time gap between failure and success — classic brute-force pattern
> - Logins occurring at unusual hours (e.g. 01:35am, 02:45am)
> - Machine accounts (ending in `$`) showing interactive login failures — these rarely log in interactively under normal circumstances

> ⚠️ **Log Parser limitation discovered during testing:** Log Parser 2.2 does not support `JOIN` operations or table aliases when querying `.evtx` files directly. The workaround above — extracting to CSV first, then correlating in PowerShell — achieves the same result and is more flexible for real-world use.

---

## 🎓 Lessons Learned

### Technical insights
- **Native tooling has serious power.** Before reaching for a SIEM, an analyst can do meaningful investigation with just Event Viewer, PowerShell, and Log Parser. This builds intuition for *what* a SIEM is doing under the hood.
- **Log Parser's SQL syntax has real limitations.** While the `SELECT`, `FROM`, and `WHERE` clauses work well against `.evtx` files, `JOIN` operations and table aliases are not supported for that input format. Workarounds — such as extracting to CSV first and joining there — are necessary for multi-file correlation.
- **PowerShell's `Get-WinEvent` is the modern path.** While Log Parser is fast for one-off single-file queries, PowerShell scales better for repeatable, scriptable analysis, handles complex logic like multi-file correlation natively, and integrates cleanly with the rest of the Windows ecosystem.
- **PowerShell quoting requires care.** When calling external executables like `LogParser.exe` from PowerShell, the `&` call operator is required, and nested quotes inside SQL strings must be managed carefully. Running complex Log Parser queries from Command Prompt avoids most of these quoting issues.

### SOC analyst mindset
- **Event ID literacy is foundational.** Knowing 4624 (successful login) and 4625 (failed login) is the bedrock of credential-attack detection. Recognising these IDs at a glance is the difference between fast triage and slow guesswork.
- **Correlation is where value lives.** A failed login is noise; a failed login *followed by* a successful one from the same user is a potential incident. The entire point of Exercise 5 is the mental shift from looking at single events to looking at *patterns across time*.
- **Machine accounts deserve attention.** The account `WIN11$` — a computer account identifiable by the trailing `$` — appeared in both failed and successful login events at unusual hours. Machine accounts don't normally produce interactive login failures, making this a meaningful signal worth investigating further.
- **Native tools sharpen SIEM skills.** After analysing logs in raw form, Splunk queries become more confident — you know exactly what fields you're looking for and why, rather than relying on pre-built dashboards.

### Challenges & how I overcame them
- **Log Parser JOIN syntax does not work on `.evtx` files.** The exercise brief implied a direct JOIN between two `.evtx` files was possible. Testing revealed that Log Parser 2.2 does not support `JOIN` or table aliases for the EVT input format — only for CSV and other text-based formats. The solution was a two-stage approach: use Log Parser to extract each `.evtx` file to CSV, then use PowerShell to perform the correlation join on the resulting CSV files.
- **Usernames are not a top-level field in `.evtx` logs.** The username is embedded inside the `Message` field rather than exposed as a dedicated column. Log Parser's `EXTRACT_TOKEN` function was used to parse it out using the `Account Name:` and `Account Domain:` delimiters in the message text.
- **PowerShell echoes commands instead of executing them** when the `&` call operator is omitted and the executable path contains spaces. Adding `&` before the quoted path resolved this immediately.
- **PIN authentication doesn't generate failed login events.** Because the machine was configured to log in using a Windows PIN rather than a password, there were no Event ID 4625 entries in the Security log to work with. PIN-based authentication bypasses the standard credential checking that produces these events. The solution was to create a separate local user account via Command Prompt (`net user TestUser Password123! /add`) and deliberately attempt to log in with the wrong password several times to generate the required 4625 events, followed by a successful login to generate a matching 4624 event.

### What I would do differently next time
- **Build a `.ps1` script for the full Exercise 5 workflow.** The three-step correlation process (extract failed → extract successful → join in PowerShell) is a natural candidate for a single reusable script with parameters for file paths and Event IDs.
- **Generate test data deliberately.** Rather than relying on whatever events happened to exist on the system, intentionally triggering known activity — such as several failed login attempts followed by a successful one — produces cleaner, more meaningful correlation results and makes the exercise easier to validate.
- **Add timestamps to the correlation output.** The current output shows all combinations of failed and successful logins for a given user. Adding a time-delta column (how many seconds/minutes between failure and success) would make brute-force patterns immediately obvious without manual inspection.

---

## 🔗 Related Projects

- 🛡️ [Splunk SIEM Project](https://github.com/NickB-26/SPLUNK-Project) — analysing the same Windows Event Logs through a SIEM. Together these projects show end-to-end log analysis from native tools to enterprise SIEM.

---

## 📚 References

- [Microsoft — Log Parser 2.2 Download](https://www.microsoft.com/en-us/download/details.aspx?id=24659)
- [Microsoft Learn — Get-WinEvent cmdlet](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Microsoft Learn — Audit Logon Events (Event IDs 4624, 4625)](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)
- [Ultimate Windows Security — Event ID Encyclopedia](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/Default.aspx)
