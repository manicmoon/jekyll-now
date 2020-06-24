### Problem:
- Newly created account syncs to AzureAD but is provisioned with the wrong Azure UPN, and an O365 mailbox which does not exist within the on-premises exchange

### Fix:

It's as simple as

```
Enable-MailUser -Identity <email address> –ExternalEmailAddress <onmicrosoft.com email address>
Enable-RemoteMailbox <username>
```

### Powershell Script against multiple OUs

I also needed to check for newly provisioned accounts and ensure the above commands are run before the automated Azure AD sync cycle. The code checks whether proxyAddresses value is empty. If empty, then run the commands.

```
  #Variables

 $OU=@('OU=Staff,DC=my,DC=company','OU=OtherStaff,DC=my,DC=company')
 $WithinLast15Mins = (get-date).AddMinutes(-15)
 $externalDomain = "@mycompany.onmicrosoft.com"

 $OU | foreach {
    # Get list of users that were created within the last 15 minutes # 
    $users = get-aduser -searchbase $_ -filter * -property whenCreated,proxyAddresses,msExchRemoteRecipientType | Where {$_.whenCreated -gt $WithinLast15Mins}
    
    # Check if proxyAddresses attribute or msExchRemoteRecipientType has been set, if not then run #
    foreach($user in $users){
        if (($user.proxyAddresses -like $null) -and ($user.msExchRemoteRecipientType -notlike 1)){
            $ExternalEmailAddress = $user.SamAccountName+$externalDomain
            Add-PSSnapin Microsoft.Exchange.Management.PowerShell.SnapIn; 
                 
            Enable-MailUser -identity "$user.UserPrincipalName" -ExternalEmailAddress "$ExternalEmailAddress"

            Enable-RemoteMailbox "$user.SamAccountName"
        }
    }
}
```

Refer to the following for more details

- [Match Office 365 Mailbox with New On-Premises User in a Hybrid Deployment](http://techgenix.com/match-office-365-mailbox-new-premises-user-hybrid-deployment/)
- [Microsoft community thread regarding the same issue](https://techcommunity.microsoft.com/t5/exchange/i-created-an-exchange-online-mailbox-but-it-doesn-t-appear-in-on/m-p/64542)
