<!-- TITLE: Content Scripts -->
<!-- SUBTITLE: A quick summary of Content Scripts -->

# Usefull code snippets

### Delete old ImageNow services
```powershell
Get-Service -Name "imagenow*6.7*" | foreach {
    $svcname = $_.Name 
    $svcname
	sc.exe delete `"$svcname`"
}
```

### Uptade services to run with service accounts
```powershell
#Run as amdinistrator
if (!([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) { Start-Process powershell.exe "-NoProfile -ExecutionPolicy Bypass -File `"$PSCommandPath`"" -Verb RunAs; exit }

#NOTE: YOU HAVE TO MANUALLY GRANT "LOG ON AS A SERVICE" ROLE TO THE ACCOUNT. Local Secutiry Policy -> Local Policies -> User Rights Assignment -> Log On as a service

#Change Username and password
$username = "<DOMAIN>\<USER>"
$password = "<PASSWORD>"

#Add user as local admin
net localgroup administrators $username /add

#Set service to run as this user
Get-Service -Name "imagenow*7.2*" | foreach {
    $svcname = $_.Name 
    $svcname
    sc.exe config `"$svcname`" obj= `"$username`" password= `"$password`"
}

```


### Update JAVA_HOME
```batchfile
setx -m JAVA_HOME "C:\Program Files\Java\jre1.8.0_152"
setx -m PATH "%PATH%;%JAVA_HOME%\bin";
```

