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


### Update JAVA_HOME
```batchfile
setx -m JAVA_HOME "C:\Program Files\Java\jre1.8.0_152"
setx -m PATH "%PATH%;%JAVA_HOME%\bin";
```

