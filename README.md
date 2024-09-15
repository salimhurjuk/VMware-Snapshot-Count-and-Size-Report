# VMware Snapshot Count and Size Report
VMware Snapshot Count and Size Report on Email including HTML and CSV Format

# Credential Encryption for VMware vCenter and SMTP

## Saving vCenter Credentials
$vcCred = Get-Credential -Message "Enter vCenter Server credentials"
$vcCred | Export-Clixml -Path "C:\vcCred.xml"

## Saving SMTP Credentials
$smtpCred = Get-Credential -Message "Enter SMTP credentials (email and password)"
$smtpCred | Export-Clixml -Path "C:\smtpCred.xml"

# Email Report Sample

![image](https://github.com/user-attachments/assets/39e7deeb-2d47-44b9-b394-3a643073ddb5)
