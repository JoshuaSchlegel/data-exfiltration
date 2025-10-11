## **Data Exfiltration** 
<img width="1011" height="590" alt="image" src="https://github.com/user-attachments/assets/45af778f-c503-4195-9fe3-939e7d919622" />



# 🎯 **Use Case**   

## 📚 **Scenario:**  
An employee, working in a sensitive department, was recently placed on a performance improvement plan (PIP). After displaying concerning behavior, management suspects the employee may be planning to steal proprietary information and leave the company. The investigation involves analyzing activities on the employee's corporate device (`windows-target-1`) using Microsoft Defender for Endpoint (MDE).  

---

## 📊 **Incident Summary and Findings**  

### **Timeline Overview**  
1. **🔍 Archiving Activity:**  
   - **Observed Behavior:** Frequent creation of `.zip` files in a folder labeled "backup."  
   - **Detection Query (KQL):**  
     ```kql
     DeviceFileEvents
     | top 20 by Timestamp desc
     ```
     ```kql
     DeviceNetworkEvents
     | top 20 by Timestamp desc
     ```
     ```kql
     DeviceProcessEvents
     | top 20 by Timestamp desc
     ```
     ```kql
     DeviceFileEvents
     | where DeviceName == "windows-target-1"
     | where FileName endswith ".zip"
     | order by Timestamp desc
     ```
     
     <img width="1511" height="847" alt="image" src="https://github.com/user-attachments/assets/b3af07f6-8cd6-45f8-80e1-d6643a1dc719" />


2. **⚙️ Process Analysis:**  
   - **Observed Behavior:** I took one of the instances of a zip file being created, took the timestamp and searched under DeviceProcessEvents for anything happening 2 minutes before the archive was created and 2 mintutes after. I discoverd around the same time that a powershell script had silently installed 7zip, and then used 7zip to zip up employee data into an archive.
   - **Detection Query (KQL):**  

     ```kql
     let VMName = "windows-target-1";
     let specificTime = datetime(2025-09-28T04:48:56.0852178Z);
     DeviceProcessEvents
     | where Timestamp between ((specificTime - 2m) .. (specificTime + 2m))
     | where DeviceName == VMName
     | order by Timestamp desc
     | project Timestamp, DeviceName, ActionType, FileName, ProcessCommandLine
     ```
<img width="1537" height="871" alt="image" src="https://github.com/user-attachments/assets/b49f688d-bb15-4048-9a08-438efe02ac96" />



   3. **🌐 Network Exfiltration Check:**  
   - **Observed Behavior:** No evidence of data exfiltration via network logs during the time frame.  

   - **Detection Query (KQL):**  

     ```kql
     let VMName = "windows-target-1";
     let specificTime = datetime(2025-09-28T04:48:56.0852178Z);
     DeviceProcessEvents
     | where Timestamp between ((specificTime - 2m) .. (specificTime + 2m))
     | where DeviceName == VMName
     | order by Timestamp desc
     ```  

4. **📝 Response:**  
   - Shared findings with the manager, highlighting automated archive creation and no immediate signs of exfiltration. The device was isolated, awaiting further instructions.

---

---

## 🛡️ **MITRE ATT&CK Framework TTPs**  

| **Tactic**           | **Technique**                                                                                     | **ID**            | **Description**                                                                                                                                                 |  
|-----------------------|---------------------------------------------------------------------------------------------------|-------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|  
| 🛠️ **Execution**      | PowerShell                                                                                       | T1059.001         | PowerShell scripts were used to silently install 7-Zip and execute file compression commands.                                                                   |  
| 📦 **Collection**      | Archive Collected Data                                                                           | T1560.001         | Employee data was compressed into `.zip` files using 7-Zip, possibly for easier handling or exfiltration.                                                       |  
| 📂 **Exfiltration**    | Exfiltration Over Alternative Protocol                                                           | T1048             | Although no network exfiltration was detected, the technique aligns with the potential misuse of alternate protocols for stealthy data transfer.                |  
| 🔍 **Discovery**       | Process Discovery                                                                                | T1057             | Processes were reviewed to identify activities surrounding the installation and use of 7-Zip for archiving.                                                     |  

---

### 🧑‍💻 **Next Steps**  
1. Monitor John’s account activity for unusual access or privilege escalation.  
2. Implement DLP (Data Loss Prevention) measures to alert on potential data exfiltration.  
3. Escalate findings to management and recommend a follow-up review of John's device for additional forensic artifacts.  

---

## Steps to Reproduce:
1. Provision a virtual machine with a public IP address
2. Ensure the device is actively communicating or available on the internet. (Test ping, etc.)
3. Onboard the device to Microsoft Defender for Endpoint
4. Verify the relevant logs (e.g., network traffic logs, exposure alerts) are being collected in MDE.
5. Execute the KQL query in the MDE advanced hunting to confirm detection.

---

## Created By:
- **Author Name**: Joshua Schlegel
- **Author Contact**: https://www.linkedin.com/in/joshuaschlegel/
- **Date**: October 2025

## Validated By:
- **Reviewer Name**: Josh Madakor
- **Reviewer Contact**: https://www.linkedin.com/in/joshmadakor/
- **Validation Date**: October 2025

---

## Revision History:
| **Version** | **Changes**                   | **Date**         | **Modified By**   |
|-------------|-------------------------------|------------------|-------------------|
| 1.0         | Initial draft                  | `October 09, 2025`  | `Trevino Parker`   
