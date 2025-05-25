# Windows Comand And Fix

### Get Sistem data
* CMD
```cmd
wmic bios get serialnumber
wmic csproduct get name, identifyingnumber
systeminfo
```
* PowerShell
```cmd
Get-WmiObject -Class Win32_ComputerSystemProduct
```
