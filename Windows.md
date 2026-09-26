# Windows: comandos y soluciones

## Obtener información del sistema
```cmd
wmic bios get serialnumber
```
```cmd
wmic csproduct get name, identifyingnumber
```
```cmd
systeminfo
```

- PowerShell
```powershell
Get-WmiObject -Class Win32_ComputerSystemProduct
```

# Limpieza de Windows
## WinUtil
- [GitHub](https://github.com/ChrisTitusTech/winutil?utm_source=chatgpt.com)
- [Página web](https://christitus.com/windows-tool/?utm_source=chatgpt.com)

## Instalar Windows 10/11
- Descargar `MediaCreationTool.exe` desde Microsoft.
- Crear un USB de arranque.
- Validar el tipo de disco: SSD o HDD.
```powershell
Get-PhysicalDisk | Format-Table FriendlyName, MediaType, BusType, Size
```
- Si se quiere instalar en un SSD, probablemente el instalador de Windows requiera controladores.
- Crear la carpeta `RST` en la unidad `E:` (`USB-BOOT`) y copiar los controladores `.inf`.
```powershell
pnputil /export-driver * E:\RST
```
- Instalar Windows y, si no aparece el controlador, cargarlo desde `RST`.
