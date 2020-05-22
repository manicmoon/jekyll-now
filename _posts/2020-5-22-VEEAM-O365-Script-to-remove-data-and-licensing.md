I created a simple script to perform the following on a list of e-mail addresses:

* Remove Veeam O365 backup data for a user (Exchange, Sharepoint, OneDrive)
* Remove Veeam licensing from said user

```
###### !!!! WARNING !!!!
###### Running this script will remove backup data and licensing for a specific user 
###### Only run when you don't want a user's account backed up!
###### Removes onedrive, exchange and sharepoint data

Import-Module "C:\Program Files\Veeam\Backup365\Veeam.Archiver.PowerShell\Veeam.Archiver.PowerShell.psd1"

# Get the Organization
$org = Get-VBOOrganization -Name "organisationname.onmicrosoft.com"

# Get repository
$repository = Get-VBORepository -name "Exchange Repository"


# Import e-mail addresses using CSV
#
$emailaddresses = get-content C:\temp\emailaddresses.csv


## Run loop on email addresses to remove data and then remove licensing

foreach ($emailaddress in $emailaddresses)
{
$user = Get-VBOEntityData -Type User -Repository $repository -Name $emailaddress

if ($user -eq $null){write-host "The following e-mail address doesn't exist on VEEAM: $emailaddress"}

else {
write-host "Preparing to remove data for" $user.DisplayName
Remove-VBOEntityData -Repository $repository -User $user -Mailbox -ArchiveMailbox -OneDrive -Sites -Confirm:$false
write-host "$user.DisplayName data has been removed"

$licensedUser = Get-VBOLicensedUser -Organization $org -Name "$emailaddress"
write-host "Preparing to remove license for" $licenseduser.UserName
Remove-VBOLicensedUser -User $licensedUser
Write-host "License removal complete for" $licensedUser.UserName
}
} 

```
