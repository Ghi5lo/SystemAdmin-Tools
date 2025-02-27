# Automated Backup and Restore for Windows Server

## **Project Overview**
This PowerShell script automates the backup process for critical files and system configurations on a Windows Server. It includes compression, remote storage, and an alert system for failed backups.

## **Technologies & Tools**
- **PowerShell** (for scripting)
- **Robocopy** (for incremental file backups)
- **Task Scheduler** (for automation)
- **Windows Server Backup** (for full system backup if needed)
- **SMTP (Send-MailMessage)** (for email notifications)

---

## **1️⃣ Backup Script**
Save the following script as `backup.ps1`:

```powershell
# Define variables
$BackupFolder = "C:\Backup"
$BackupDate = Get-Date -Format "yyyy-MM-dd_HH-mm"
$Destination = "\\192.168.1.100\BackupServer"  # Remote server path example
$LogFile = "$BackupFolder\backup_log.txt"

$EmailTo = "admin@example.com"
$EmailFrom = "my@gmail.com" # Lets use a gmail email for this istance
$SMTPServer = "smtp.gmail.com"
$SMTPPort = 587
$Password = "gmail_password"
$SecurePassword = ConvertTo-SecureString $Password -AsPlainText -Force
$Creds = New-Object System.Management.Automation.PSCredential ($EmailFrom, $SecurePassword)

# Create backup directory if it does not exist
If (!(Test-Path $BackupFolder)) { New-Item -ItemType Directory -Path $BackupFolder }

# Backup files and folders using Robocopy
$SourceFolders = @("C:\inetpub\wwwroot", "C:\ProgramData", "C:\Users\Administrator\Documents") # Those are the directories I wanna backup
foreach ($Folder in $SourceFolders) {
    $BackupPath = "$BackupFolder\$(Split-Path $Folder -Leaf)_$BackupDate"
    Robocopy $Folder $BackupPath /E /COPY:DAT /LOG:$LogFile /NP
}

# Compress backup folder
$ZipFile = "$BackupFolder\Backup_$BackupDate.zip"
Compress-Archive -Path "$BackupFolder\*" -DestinationPath $ZipFile

# Copy to remote server
try {
    Copy-Item -Path $ZipFile -Destination $Destination -Force
} catch {
    $ErrorMessage = "Backup upload failed: $_"
    Send-MailMessage -To $EmailTo -From $EmailFrom -Subject "Backup Upload Failed" -Body $ErrorMessage -SmtpServer $SMTPServer -Credential $Creds -Port $SMTPPort -UseSsl
}

# Cleanup old backups (keep only last 7 days)
$RetentionDays = 7
Get-ChildItem -Path $BackupFolder -Filter "*.zip" | Where-Object { $_.CreationTime -lt (Get-Date).AddDays(-$RetentionDays) } | Remove-Item -Force

# Log completion
Add-Content -Path $LogFile -Value "Backup completed on $(Get-Date) - File: $ZipFile"
```

---

## **2️⃣ Schedule the Backup with Task Scheduler**
To automate the script execution:
1. **Open Task Scheduler** (`taskschd.msc` from Run).
2. **Create a new task:**
   - Run as an Administrator account.
   - Run even if the user is not logged in.
   - Set a trigger (e.g., every night at 2:00 AM).
3. **Add an action:**
   - **Program:** `powershell.exe`
   - **Arguments:** `-ExecutionPolicy Bypass -File C:\Scripts\backup.ps1`

---

## **3️⃣ Restore Data**
To restore files, use the following script:

```powershell
# Extract backup
Expand-Archive -Path "C:\Backup\Backup_xxx.zip" -DestinationPath "C:\Restore"

# Restore files
Robocopy "C:\Restore\inetpub" "C:\inetpub\wwwroot" /E /COPY:DAT
Robocopy "C:\Restore\Documents" "C:\Users\Administrator\Documents" /E /COPY:DAT
```

---
## **Conclusion**
This script provides a **fully automated and secure backup solution** for Windows Servers, ensuring reliable recovery in case of system failure. 🚀 If you need modifications, feel free to customize it!