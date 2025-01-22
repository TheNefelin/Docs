# Windows Comand And Fix

### Get Sistem data
* CMD
```
wmic bios get serialnumber
wmic csproduct get name, identifyingnumber
systeminfo
```
* PowerShell
```
Get-WmiObject -Class Win32_ComputerSystemProduct
```
