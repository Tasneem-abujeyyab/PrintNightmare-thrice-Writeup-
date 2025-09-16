# PrintNightmare-thrice-Writeup-
Scenario: After discovering the PrintNightmare attack the security team pushed an emergency patch to all the endpoints. The PrintNightmare exploit used previously no longer works. All is well. Unfortunately, the same 2 employees discovered yet another exploit that can possibly work on a fully patched endpoint to elevate their privileges.

Task: Inspect the artifacts on the endpoint to detect the PrintNightmare exploit used.
_________________________________________________________________________________________________________________________

🔍 Objective
- Detect traces of the PrintNightmare exploit on the endpoint.
- Analyze system artifacts and network traffic.
- Document indicators of compromise (IOCs) and exploitation behavior.

## 🛠️ Tools Used
- **Wireshark** – For packet capture analysis.
- **Process Monitor (Procmon)** – to trace process creation and DLL loading activity
- **Event Viewer** – For service installation and printer spooler activity.

## 📊 Investigation Steps
# 1.What remote address did the employee navigate to?
To begin my investigation, I loaded the provided PCAP file and focused on analyzing SMB2 protocol traffic. This helped me identify the remote destination the employee interacted with during the suspected exploitation.
Using Wireshark, I applied the filter: 

<img width="918" height="178" alt="image" src="https://github.com/user-attachments/assets/3c799e28-0827-41fb-b65b-32865265e8bd" />

By inspecting the packet details — specifically the destination IP address in the SMB2 request frames — I was able to identify the remote system the employee interacted with.

# 2.Per the PCAP, which user returns a STATUS_LOGON_FAILURE error?
As I reviewed the SMB2 traffic in Wireshark, I focused on the Session Setup Response packets to identify any failed authentication attempts. By scrolling through the SMB2 protocol frames and inspecting the headers, I found a response containing the status code: STATUS_LOGON_FAILURE

<img width="695" height="161" alt="image" src="https://github.com/user-attachments/assets/ca3051b1-1d36-4219-bb5e-60ab7bae1b6e" />

The packet details revealed that the user associated with this failure: 

<img width="945" height="322" alt="image" src="https://github.com/user-attachments/assets/5d5aa70a-ecba-472e-b719-1566b8ef0d8f" />

The flag is structured like this DOMAIN\USER.

# 3.Which user successfully connects to an SMB share? 

To identify which user successfully connected to an SMB share, I applied the following display filter: 
<img width="977" height="569" alt="image" src="https://github.com/user-attachments/assets/b9045057-0576-4503-9bb5-faeb5ad5a9ef" />

This filter isolates SMB2 traffic where the NT Status code is 0, indicating a successful operation.
The flag is structured like this DOMAIN/USER.

# 4.What is the first remote SMB share the endpoint connected to? What was the first filename? What was the second? (format: answer,answer,answer)
Using the hint: Wireshark or Brim Brim i will switch to brim, which is a tool that helps you analyze PCAP files by converting raw network traffic into readable logs using Zeek.
I will look for _path=smb_mapping OR _path=dce_rpc | sort ts
This tells Brim to:
1. Show all events related to SMB share mapping (smb_mapping) and RPC (remote procedure calls)  activity (dce_rpc)
2. Sort them by timestamp so you can see what happened first

<img width="1029" height="685" alt="image" src="https://github.com/user-attachments/assets/208bdd34-898c-4787-8935-4e44c6866ac8" />

# 5.From which remote SMB share was malicious DLL obtained? What was the path to the remote folder for the first DLL? How about the second? (format: answer,answer,answer)
For this task I will check the File Activity query

<img width="1056" height="677" alt="image" src="https://github.com/user-attachments/assets/cdffa797-6871-4586-8a76-364a829d137c" />

# 6.What was the first location the malicious DLL was downloaded to on the endpoint? What was the second?
I reviewed the provided endpoint log file to determine where the malicious DLLs were downloaded locally. Using FullEventLogView, I filtered for relevant entries.
At first, make sure to view events of all time, not just last 7 days.

<img width="774" height="372" alt="image" src="https://github.com/user-attachments/assets/b088a158-0496-463a-b0ee-95d0e51edf24" />

<img width="928" height="496" alt="image" src="https://github.com/user-attachments/assets/db59f436-b0c6-4c7e-8870-38c45c330642" />

# 7. What is the folder that has the name of the remote printer server the user connected to? (provide the full folder path)
To identify the remote SMB share used during exploitation, I searched for the term printnightmare.gentilkiwi in the File Explorer.

<img width="939" height="329" alt="image" src="https://github.com/user-attachments/assets/a41af37b-0a03-42a1-8a89-e7f8dc712d73" />

# 8. What is the name of the printer the DLL added? 
Back to the FullEventLogView, I set the provider to Microsoft-Windows-PrintService. 

<img width="768" height="505" alt="image" src="https://github.com/user-attachments/assets/b28efb62-ccb5-4227-906f-1fbe9a9204e5" />

Scrolling down in the events , I found the printer name 

<img width="952" height="416" alt="image" src="https://github.com/user-attachments/assets/555d3b45-d0e3-431e-8b90-267f6b2005ab" />

# 9. What was the process ID for the elevated command prompt? What was its parent process? (format: answer,answer)
I will open process monitor and filter for cmd.exe

<img width="972" height="200" alt="image" src="https://github.com/user-attachments/assets/283ffece-4dae-4b8e-bf12-7aca87d4b77b" />

and filter for parent id process to find its name 

# 10. What command did the user perform to elevate privileges?
Now back to the event viewer and search for event ID which is cmd.exe, I found the the command that executed

<img width="1262" height="747" alt="image" src="https://github.com/user-attachments/assets/fe4fa670-fe2b-4d83-9a06-d7aa2907b7c9" />











