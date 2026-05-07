# 🛡️ Project 1 — Investigating Windows Event Logs for Security Incidents

A hands-on project demonstrating how to access, filter, parse, and correlate Windows Event Logs using native Windows tooling — **Event Viewer**, **PowerShell**, and **Log Parser** — to detect and investigate potential security incidents.

---

## 🎯 Project Overview

Windows Event Logs are one of the most important data sources available to a SOC analyst. They record everything from successful and failed authentication attempts to privilege escalation and system errors — and being able to interrogate them quickly with native tools is a fundamental skill.

This project walks through five practical exercises focused on credential-based attack detection (Event IDs **4624** and **4625**), demonstrating the full investigation workflow: **access → filter → export → parse → correlate**.

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
2. Run the installer with administrative privileges and follow the on-screen prompts.
3. By default, Log Parser installs to `C:\Program Files (x86)\Log Parser 2.2\`.

> 💡 **Note:** Log Parser is officially unsupported (last updated in 2005) but still works reliably on modern Windows. PowerShell's `Get-WinEvent` is its modern equivalent.

---

## 📋 Pre-requisites

- Basic understanding of the Windows operating system
- Familiarity with the command-line interface
- Administrative privileges on the Windows machine

---

## 🔬 Exercises

### Exercise 1 — Accessing Windows Event Logs using Event Viewer

**🎯 Objective:** Learn how to access and navigate Windows Event Logs using Event Viewer.

**Steps:**

1. **Open Event Viewer** — Press `Win + R`, type `eventvwr.msc`, and press `Enter`.
2. **Navigate the log categories** — Expand **Windows Logs** to reveal the five primary categories: **Application**, **Security**, **Setup**, **System**, and **Forwarded Events**.
3. **Explore the Security log** — Click **Security** under **Windows Logs** and review the events for authentication and access activity.

<p align="center">
  <img src="REPLACE_WITH_GITHUB_URL_FOR_event_viewer_security_log.png" alt="Event Viewer showing the Security log" width="700"><br>
  <em>Figure 1: Event Viewer displaying the Security log under Windows Logs</em>
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
  <img src="REPLACE_WITH_GITHUB_URL_FOR_filter_current_log.png" alt="Filter Current Log dialog with Event ID 4625" width="700"><br>
  <em>Figure 2: Filtering the Security log for Event ID 4625 (failed logins)</em>
</p>

3. **Export the filtered logs:**
   - Right-click the filtered view and select **Save Filtered Log File As...**
   - Save the file as `FailedLogins.evtx`.

<p align="center">
  <img src="REPLACE_WITH_GITHUB_URL_FOR_failed_logins_export.png" alt="Saving the filtered log as FailedLogins.evtx" width="700"><br>
  <em>Figure 3: Exporting filtered events to FailedLogins.evtx</em>
</p>

**Expected Output:** A `FailedLogins.evtx` file containing only Event ID 4625 entries, ready for further analysis.

> 💡 **Tip for Exercise 5:** While you're here, repeat steps 2–3 with **Event ID 4624** and save as `SuccessfulLogins.evtx` — you'll need this file for the correlation exercise later.

---

### Exercise 3 — Parsing Event Logs using Log Parser

**🎯 Objective:** Use Log Parser to extract and analyse structured data from event logs.

**Steps:**

1. **Open Command Prompt** and navigate to the Log Parser installation directory:
   ```cmd
   cd "C:\Program Files (x86)\Log Parser 2.2"
   ```

2. **Run the following query** to extract key fields from `FailedLogins.evtx` into a CSV file:
   ```sh
   LogParser.exe "SELECT TimeGenerated, EventID, EventTypeName, Message INTO FailedLogins.csv FROM FailedLogins.evtx" -i:EVT -o:CSV
   ```

3. **Open the resulting `FailedLogins.csv` file** in Excel, Notepad, or any CSV viewer to review the extracted data.

<p align="center">
  <img src="REPLACE_WITH_GITHUB_URL_FOR_logparser_output.png" alt="Log Parser command output and resulting CSV" width="700"><br>
  <em>Figure 4: Log Parser extracting failed login data into CSV format</em>
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
  <img src="REPLACE_WITH_GITHUB_URL_FOR_powershell_failed_logins.png" alt="PowerShell output showing failed login attempts" width="700"><br>
  <em>Figure 5: PowerShell listing failed login attempts via Get-WinEvent</em>
</p>

**Expected Output:** A formatted list of failed login attempts in the PowerShell console, plus a saved `FailedLogins.txt` file containing the same output.

---

### Exercise 5 — Correlating Events for Comprehensive Analysis

**🎯 Objective:** Correlate failed and successful login events to identify potential credential-based attacks.

> ⚠️ **Pre-requisite:** Before running the correlation query, you need a second exported file. Repeat Exercise 2 but filter for **Event ID 4624** (successful logins) and save the result as `SuccessfulLogins.evtx` in the same folder as `FailedLogins.evtx`.

**Steps:**

1. **Open Event Viewer** and select the **Security** log.
2. **Identify the two relevant Event IDs:**
   - **Event ID 4624** — *An account was successfully logged on*
   - **Event ID 4625** — *An account failed to log on*
3. **Run the following Log Parser query** to JOIN the two log files on the username:
   ```sh
   LogParser.exe "SELECT a.TimeGenerated AS FailedLoginTime, b.TimeGenerated AS SuccessfulLoginTime, a.Message AS FailedLoginMessage, b.Message AS SuccessfulLoginMessage INTO CorrelatedLogins.csv FROM FailedLogins.evtx a JOIN SuccessfulLogins.evtx b ON a.User=b.User WHERE a.EventID=4625 AND b.EventID=4624" -i:EVT -o:CSV
   ```
4. **Review `CorrelatedLogins.csv`** for patterns — particularly successful logins that follow multiple failed attempts from the same user (a classic indicator of brute-force or password-spray activity).

<p align="center">
  <img src="REPLACE_WITH_GITHUB_URL_FOR_correlated_logins.png" alt="Correlated logins CSV showing failed and successful login pairs" width="700"><br>
  <em>Figure 6: Correlated failed and successful login events in CorrelatedLogins.csv</em>
</p>

**Expected Output:** A CSV file pairing failed and successful login attempts by user — providing visibility into potential credential attacks where an attacker eventually gained access.

---

## 🎓 Lessons Learned

### Technical insights
- **Native tooling has serious power.** Before reaching for a SIEM, an analyst can do meaningful investigation with just Event Viewer, PowerShell, and Log Parser. This builds intuition for *what* a SIEM is doing under the hood.
- **Log Parser's SQL syntax is intuitive.** Knowing SQL transferred almost directly — `SELECT`, `FROM`, `WHERE`, and `JOIN` work exactly as you'd expect, just against `.evtx` files.
- **PowerShell's `Get-WinEvent` is the modern path.** While Log Parser is fast for one-off queries, PowerShell scales better for repeatable, scriptable analysis and integrates cleanly with the rest of the Windows ecosystem.

### SOC analyst mindset
- **Event ID literacy is foundational.** Knowing 4624 (successful login) and 4625 (failed login) is the bedrock of credential-attack detection. Recognising these IDs at a glance is the difference between fast triage and slow guesswork.
- **Correlation is where value lives.** A failed login is noise; a failed login *followed by* a successful one from the same user is a potential incident. The entire point of Exercise 5 is the mental shift from looking at single events to looking at *patterns*.
- **Native tools sharpen SIEM skills.** After analysing logs in raw form, my Splunk queries became more confident — I knew exactly what fields I was looking for and why.

### Challenges & how I overcame them
- **Log Parser felt dated.** It hadn't been updated since 2005, and the documentation is scattered. I worked around this by leaning on PowerShell's better-documented `Get-WinEvent` for anything Log Parser felt awkward at.
- **Exercise 5 needed an extra step.** The brief assumed `SuccessfulLogins.evtx` already existed — it didn't. I had to repeat Exercise 2's filter/export workflow with Event ID 4624 first. (I've added a clear pre-requisite note to this README to spare the next reader the same friction.)

### What I would do differently next time
- **Build a `.ps1` script for Exercise 4.** Re-typing the `Get-WinEvent` command repeatedly was slow. A small reusable script with parameters (Event ID, time range, output path) would have saved time and shown scripting initiative.
- **Generate test data deliberately.** I worked with whatever events happened to be on the system. Next time I'd intentionally trigger known-bad activity (e.g., several failed RDP attempts) so the correlation queries return more meaningful results.

---

## 🔗 Related Projects

- 🛡️ [Splunk SIEM Project](https://github.com/NickB-26/SPLUNK-Project) — analysing the same Windows Event Logs through a SIEM. Together these projects show end-to-end log analysis from native tools to enterprise SIEM.

---

## 📚 References

- [Microsoft — Log Parser 2.2 Download](https://www.microsoft.com/en-us/download/details.aspx?id=24659)
- [Microsoft Learn — Get-WinEvent cmdlet](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Microsoft Learn — Audit Logon Events (Event IDs 4624, 4625)](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)
- [Ultimate Windows Security — Event ID Encyclopedia](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/Default.aspx)

