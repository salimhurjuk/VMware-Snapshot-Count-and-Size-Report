<#
    .SYNOPSIS
    VMwareSnapshotReport_v3.ps1

    .DESCRIPTION
    1. Export VMware VMs Snapshot Count and Size in GB.
    2. Keep Minimum Two Snapshots and Would like to know Pending to Delete Details.

    .LINK
    

    .NOTES
    Written by: Salim Hurjuk
    LinkedIn:   linkedin.com/in/salimhurjuk
    Twitter:    https://x.com/salimhurjuk

    .CHANGELOG
    v1.00, 14/09/2024 - Initial version
#>

# Set PowerCLI Configuration to ignore invalid certificates
Set-PowerCLIConfiguration -InvalidCertificateAction Ignore -Confirm:$false

# Variables for vCenter and File Paths
$vCenterServer = "192.168.1.234"
$htmlReportPath = "C:\VMware_Snapshot_Report.html"
$csvReportPath = "C:\VMware_Snapshot_Report.csv"

# SMTP Configuration
$smtpServer = "smtp.gmail.com"
$smtpPort = 587  # Specify the SMTP port here (e.g., 25, 587, 465)
$from = "sender@domain.com"
$to = "receiver@domain.com"
$subject = "VMware Snapshot Report"

# SMTP Credentials
$smtpUser = "sender@domain.com"
$smtpPassword = "Password"

# Connect to vCenter Server
$cred = Get-Credential -Message "Enter vCenter Server credentials"
Connect-VIServer -Server $vCenterServer -Credential $cred

# Get all VMs with snapshot count, size, and PendingDeletion column
$vmReport = Get-VM | Select-Object Name, 
@{Name="VMIPAddress";Expression={ ($_ | Get-VMGuest).IPAddress -join ", " }},
@{Name="SnapshotCount";Expression={ (Get-Snapshot -VM $_).Count }},
@{Name="PendingDeletion";Expression={ (Get-Snapshot -VM $_).Count - 2 }},
@{Name="SnapshotSizeGB";Expression={ "{0:N2}" -f ((Get-Snapshot -VM $_ | Measure-Object -Property SizeGB -Sum).Sum) }}

# Filter out VMs with a SnapshotCount of 0
$vmReport = $vmReport | Where-Object { $_.SnapshotCount -gt 0 }

# Calculate totals for SnapshotCount, PendingDeletion, and SnapshotSizeGB
$totalSnapshotCount = ($vmReport | Measure-Object SnapshotCount -Sum).Sum
$totalPendingDeletion = ($vmReport | Measure-Object PendingDeletion -Sum).Sum
$totalSnapshotSizeGB = ($vmReport | Measure-Object SnapshotSizeGB -Sum).Sum

# Prepare report data for CSV output
$vmReport | Export-Csv -Path $csvReportPath -NoTypeInformation -Encoding UTF8

# Convert the report to HTML with violet color scheme and black border
$htmlContent = @"
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>VMware Snapshot Report</title>
    <style>
        body {
            font-family: Calibri, sans-serif;
            background-color: #f4f4f4;
            color: #333;
        }
        h1 {
            font-size: 22px; /* Font size for title */
        }
        table {
            width: auto;
            border-collapse: collapse;
            margin: 5px auto;
            max-width: auto; /* Ensure table width is responsive */
            table-layout: auto; /* Auto fit table layout */
        }
        table, th, td {
            border: 1px solid black; /* Black border */
        }
        th, td {
            padding: 5px;
            text-align: center; /* Center align text */
            word-wrap: break-word;
        }
        th {
            background-color: #8a2be2; /* Blue Violet */
            color: #fff;
        }
        tr:nth-child(even) {
            background-color: #e6e6fa; /* Lavender */
        }
        tr:hover {
            background-color: #dcdcdc; /* Light Gray */
        }
    </style>
</head>
<body>
    <h1>VMware Snapshot Report</h1>
    <table>
        <thead>
            <tr>
                <th>SrNo</th>
                <th>VMName</th>
                <th>VMIPAddress</th>
                <th>SnapshotCount</th>
                <th>PendingDeletion</th>
                <th>SnapshotSizeGB</th>
            </tr>
        </thead>
        <tbody>
"@

# Add rows for each VM in the report with SrNo
$srNo = 1
foreach ($vm in $vmReport) {
    $htmlContent += "<tr>
        <td>$srNo</td>
        <td>$($vm.Name)</td>
        <td>$($vm.VMIPAddress)</td>
        <td>$($vm.SnapshotCount)</td>
        <td>$($vm.PendingDeletion)</td>
        <td>$($vm.SnapshotSizeGB)</td>
    </tr>"
    $srNo++
}

# Add the totals row with merged columns A, B and C for the "Total" label
$htmlContent += @"
    <tr>
        <td colspan='3' style='font-weight:bold;'>Total</td>
        <td>$totalSnapshotCount</td>
        <td>$totalPendingDeletion</td>
        <td>$totalSnapshotSizeGB</td>
    </tr>
        </tbody>
    </table>
    <br>
    <p>Best regards,</p>
    <p><strong>Server Admin</strong><br>
    ServerAdmin Team<br>
    Phone: +91 123456789<br>
    Email: serveradmin@domain.in<br>
    </p>
</body>
</html>
"@

# Save the HTML content to a file
$htmlContent | Set-Content -Path $htmlReportPath -Encoding UTF8

# Disconnect from vCenter
Disconnect-VIServer -Server $vCenterServer -Confirm:$false

# Create SMTP Credential Object
$smtpCred = New-Object PSCredential ($smtpUser, (ConvertTo-SecureString $smtpPassword -AsPlainText -Force))

# Send Email with HTML content in the body and attach both HTML and CSV reports
Send-MailMessage -SmtpServer $smtpServer -Port $smtpPort -From $from -To $to -Subject $subject -Body $htmlContent -BodyAsHtml -Credential $smtpCred -UseSsl -Attachments $htmlReportPath, $csvReportPath

Write-Host "Snapshot report generated, email sent, and files attached successfully."
