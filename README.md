# IIS-Server-SSL-Guide
Getting and adding an SSL certificate to a site on Windows Server 2012 with Let's Encrypt using ACMEbot.

# Step 1: Install Powershell Gallery (?)
set-executionpolicy unrestricted

Install-Module -Name ACMESharp
Install-Module -Name ACMESharp.Providers.IIS
Enable-ACMEExtensionModule -ModuleName ACMESharp.Providers.IIS

### check to see if worked:
 Get-ACMEExtensionModule | Select-Object -Expand Name

Import-Module ACMESharp 

# Step 2: Use ACMESharp to request a certificate

Initialize-ACMEVault

New-ACMERegistration -Contacts mailto:<email@email.com> -AcceptTos

### Make sure the IIS Challenge Handler is available
Get-ACMEChallengeHandlerProfile -ListChallengeHandlers 

### You should see this:
manual
iis

## But you probably won't! So do it manually!

New-ACMEIdentifier -Dns <www.site.com> -Alias <Alias>

Print manual HTTP Instructions
(Complete-ACMEChallenge dns1 -ChallengeType http-01 -Handler manual).Challenge

### ...or...

(Update-ACMEIdentifier leeds -ChallengeType http-01).Challenges | Where-Object {$_.Type -eq "http-01"}


```
HandlerHandleMessage   : == Manual Challenge Handler - HTTP ==
                           * Handle Time:      [21/03/2018 09:43:03]
                           * Challenge Token:  [<REDACTED>]

                         To complete this Challenge please create a new file
                         under the server that is responding to the hostname
                         and path given with the following characteristics:
                           * HTTP URL:     [<REDACTED>]
                           * File Path:    [.well-known/acme-challenge/<REDACTED>]
                           * File Content:
                         [<REDACTED>]
                           * MIME Type:    [text/plain]
                         ------------------------------------
```



To add .well-known dir you need to add a dot at start & end when creating directory

To allow IIS to serve . folders, add a web.config to .well-known/acme-challenge with this information:

<?xml version="1.0" encoding="UTF-8"?>
 <configuration>
     <system.webServer>
         <staticContent>
             <mimeMap fileExtension="." mimeType="text/plain" />
         </staticContent>
     </system.webServer>
 </configuration>




THEN CHECK IF IT WORKS IN BROWSER



#After doing the manual stuff:
Submit-ACMEChallenge leeds -ChallengeType http-01

#check to see if complete: (Status:valid)
(Update-ACMEIdentifier dns1 -ChallengeType http-01).Challenges | Where-Object {$_.Type -eq "http-01"}


### Request & Issue Certificate
New-ACMECertificate <Alias> -Generate -Alias <newAlias>
Submit-ACMECertificate <newAlias>

### Now wait for the cerificate to be resolved. If there are missing bits of text, then it isn't resolved! Check on it with this command:
Update-ACMECertificate <newAlias>

### Generate Certificate
Get-ACMECertificate <newAlias> -ExportPkcs12 "C:\certificate.pfx" -CertificatePassword 'certPass'


### Install via IIS by going to servername > server certificates > import

### Bind the SSL Certificate by right-clicking on the webpage > edit bindings. Edit HTTPS and select the new SSL certificate.

Test that everything is working!

To do: Redirect from http to https
