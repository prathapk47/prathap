# prathap
azure 
function exportoutput
{
$From = ""
$To = ""
$Cc = "",""
$Attachment = $outpath
$date = Get-Date -DisplayHint Date
$Subject = "Deleted Resources in last 24 hours ( $date )"
$Body = "<h4>Hi Team, Please find the attached CSV file consisting of the deleted resources from last 24 hours </h4><br><br>"
$Body += “This is an automated email using script ” 
$SMTP = "smtp.outlook.com"
Send-MailMessage -From $From -to $To -Cc $Cc -Subject $Subject -Body $Body -BodyAsHtml -SmtpServer $SMTP -UseSsl -Credential (Get-Credential) -Attachments $Attachment
}
$startvalue = 0
$inpath = Read-Host " enter the file path with list of subscriptions"
$outpath = Read-Host " enter the path location where CSV file need to be stored "
foreach($line in [System.IO.File]::ReadLines("$inpath"))
{
Select-AzSubscription -subscription $line
$al= Get-AzActivityLog -StartTime (Get-Date).AddDays(-1) | Select-Object OperationID,subscriptionID,ResourceID,caller -ExpandProperty operationname | Where-Object{$_.Localizedvalue -match "Delete" -or $_.Localizedvalue -match "Remove" }
$tot = ($al).Count
if($tot -eq 0)
{ write-host " There are no resources deleted in the last 24 hours SOP "
}
else
{
$al | Export-Csv -Append -path "$outpath"
}
$startvalue = $startvalue + $tot


}
if($startvalue -gt 0)
{
exportoutput
}
