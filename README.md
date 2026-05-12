# 🛡️ Investigating Windows Event Logs for Security Incidents

A hands-on project demonstrating how to access, filter, parse, and correlate Windows Event Logs using native Windows tooling — **Event Viewer**, **PowerShell**, and **Log Parser** — to detect and investigate potential security incidents.

---

## 🎯 Project Overview

Windows Event Logs were the first thing I dug into when I started learning SOC fundamentals — and for good reason. They record everything from login attempts to privilege escalation, and being able to query them quickly with native tools felt like a skill worth having before touching a SIEM.

This project covers five exercises built around credential-based attack detection using Event IDs **4624** and **4625**, following the full workflow: **access → filter → export → parse → correlate**.

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
<br>
2. Click through the wizard and click **Finish** when complete. <p align="left">

 <p align="center">
  <img src="https://github.com/user-attachments/assets/81d95b06-876d-478e-b24f-5f9e5f35eb15" alt="Log Parser 2.2 Setup Wizard completion screen" width="700"><br>
  <em>Figure 3: Log Parser 2.2 installation complete</em>
</p>

<br>

3. By default, Log Parser installs to `C:\Program Files (x86)\Log Parser 2.2\`.

<p align="center">
  <img src="https://github.com/user-attachments/assets/f5b7c80b-ae45-4c1b-a078-ecd72fc72cfe" alt="Log Parser 2.2 folder visible in Program Files (x86)" width="700"><br>
  <em>Figure 4: Log Parser 2.2 installation directory in Program Files (x86)</em>
</p>

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
  <em>Figure 8: Log Parser extracted failed login data into a CSV file and resulting file</em>
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

1. **Extract failed logins to CSV** using Log Parser in PowerShell:
   ```powershell
   & "C:\Program Files (x86)\Log Parser 2.2\LogParser.exe" "SELECT TimeGenerated, EXTRACT_TOKEN(EXTRACT_TOKEN(Message, 1, 'Account Name:'), 0, 'Account Domain:') AS Username INTO temp_failed.csv FROM FailedLogins.evtx WHERE EventID=4625" -i:EVT -o:CSV
   ```

2. **Extract successful logins to CSV** using Log Parser in PowerShell:
   ```powershell
   & "C:\Program Files (x86)\Log Parser 2.2\LogParser.exe" "SELECT TimeGenerated, EXTRACT_TOKEN(EXTRACT_TOKEN(Message, 1, 'Account Name:'), 0, 'Account Domain:') AS Username INTO temp_success.csv FROM SuccessfulLogins.evtx WHERE EventID=4624" -i:EVT -o:CSV
   ```

3. **Correlate the two CSV files** using PowerShell to produce `CorrelatedLogins.csv`:
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

4. **Review `CorrelatedLogins.csv`** for patterns — particularly successful logins that follow multiple failed attempts from the same user, which is a classic indicator of brute-force or password-spray activity.

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
- **I underestimated native tooling.** Before this project I assumed you needed a SIEM to do anything meaningful with logs. Turns out Event Viewer, PowerShell, and Log Parser can get you surprisingly far — and working at this level gave me a much better understanding of what a SIEM is actually doing behind the scenes.
- **Log Parser's SQL syntax hits a wall fast.** The basic `SELECT`/`FROM`/`WHERE` stuff works fine against `.evtx` files, but the moment I tried a `JOIN` between two event log files it fell apart completely. Not supported for that input format. I had to export both files to CSV first and do the join in PowerShell — which actually turned out to be more flexible anyway.
- **`Get-WinEvent` is where I'd start next time.** Log Parser is handy for quick one-off queries but PowerShell handles the more complex stuff much better, scales to scripting, and doesn't need a separate install.
- **PowerShell quoting will catch you out.** Calling `LogParser.exe` from PowerShell looks straightforward until the path has spaces in it and the query has nested quotes. The `&` call operator is required, and getting the quoting right inside the SQL string took more trial and error than I expected.

### SOC analyst mindset
- **Knowing your Event IDs matters more than I thought.** Once I found the ID's for successful and failed logins (4624/4625) I've filtered the logs for what I was interested in instead of fishing around. It's a small thing but it adds up fast during triage.
- **A single event tells you almost nothing.** The interesting stuff only appeared when I started looking at sequences — a failed login followed by a success from the same account is a very different story from just a failed login on its own.
- **Machine accounts are easy to overlook.** I nearly ignored `WIN11$` in the output because it didn't look like a user account. The trailing `$` means it's a computer account, and computer accounts don't normally produce interactive login failures — so when they do, it's worth paying attention to.
- **Doing this manually made Splunk click.** After wrangling raw `.evtx` files by hand, the first time I ran a similar query in Splunk I actually understood what it was doing. That context made a real difference.

### Challenges & how I overcame them
- **Log Parser JOIN syntax does not work on `.evtx` files.** The exercise brief implied a direct JOIN between two `.evtx` files was possible. Testing revealed that Log Parser 2.2 does not support `JOIN` or table aliases for the EVT input format — only for CSV and other text-based formats. I ended up extracting each file to CSV first, then doing the correlation join in PowerShell.
- **Usernames are buried inside the Message field.** They're not exposed as a top-level column in `.evtx` logs — they're embedded in the message text. I had to use Log Parser's `EXTRACT_TOKEN` function with the `Account Name:` and `Account Domain:` delimiters to pull them out.
- **PowerShell echoes commands instead of executing them** when the `&` call operator is omitted and the executable path contains spaces. Adding `&` before the quoted path resolved this immediately.
- **PIN authentication doesn't generate failed login events.** Because the machine was configured to log in using a Windows PIN rather than a password, there were no Event ID 4625 entries in the Security log to work with. I created a separate local account with `net user TestUser Password123! /add` and deliberately logged in with the wrong password a few times to generate the events I needed.

### What I would do differently next time
- **Generate test data deliberately from the start.** I wasted time early on because I was relying on whatever events happened to already exist on the system. Intentionally triggering known activity produces cleaner results and makes it much easier to validate that your queries are working correctly.
- **Add a time-delta column to the correlation output.** Right now the output shows all combinations of failed and successful logins for a given user but doesn't calculate the gap between them. A column showing how much time elapsed between failure and success would make brute-force patterns obvious at a glance.

---

## 🔗 Related Projects

- 🛡️ [Splunk SIEM Project](https://github.com/NickB-26/SPLUNK-Project) — analysing the same Windows Event Logs through a SIEM. Together these projects show end-to-end log analysis from native tools to enterprise SIEM.

---

## 📚 References

- [Microsoft — Log Parser 2.2 Download](https://www.microsoft.com/en-us/download/details.aspx?id=24659)
- [Microsoft Learn — Get-WinEvent cmdlet](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent)
- [Microsoft Learn — Audit Logon Events (Event IDs 4624, 4625)](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)
- [Ultimate Windows Security — Event ID Encyclopedia](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/Default.aspx)
