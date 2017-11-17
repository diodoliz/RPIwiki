<!-- TITLE: Content Scripts -->
<!-- SUBTITLE: A quick summary of Content Scripts -->

# Usefull code snippets

```powershell
Get-Service -Name "imagenow*6.7*" | foreach {
    $svcname = $_.Name 
    $svcname
	sc.exe delete `"$svcname`"
}
```
