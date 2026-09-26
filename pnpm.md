# pnpm

## Verificar la instalación
```powershell
pnpm --version
where.exe pnpm
pnpm doctor
npm list -g pnpm
```

## Instalar y actualizar
```powershell
npm install -g pnpm
npm install -g pnpm@latest
pnpm setup
pnpm self-update
npm uninstall -g pnpm
```

## Consultar PNPM_HOME
```powershell
echo $env:PNPM_HOME
[Environment]::GetEnvironmentVariable("PNPM_HOME","User")
npm list -g pnpm
Get-Command pnpm | Format-List Source,Path
```
