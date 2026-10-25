# pnpm - Guía de instalación y actualización en Windows

## Problema común
Ejecutar `pnpm self-update` en Windows puede crear archivos o carpetas en ubicaciones inesperadas (como la raíz de `D:`) y romper el PATH, impidiendo que Angular u otras herramientas encuentren `ng` o `pnpm`.

**No uses `pnpm self-update` en Windows.**

> `pnpm self-update` está diseñado principalmente para **Linux y macOS**. En Windows no es confiable. Además, si usas **Corepack**, `pnpm self-update` no debe usarse en ningún sistema operativo.

## Verificar la instalación
```powershell
pnpm --version
where.exe pnpm
pnpm doctor
[Environment]::GetEnvironmentVariable("PNPM_HOME", "User")
Get-Command pnpm | Format-List Source, Path
```

## Desinstalar pnpm completamente (limpieza previa)
Ejecuta esto solo si ya tienes pnpm instalado y quieres empezar limpio:

```powershell
# Si lo instalaste con npm
npm uninstall -g pnpm

# Elimina la carpeta de pnpm
Remove-Item -Recurse -Force "$env:LOCALAPPDATA\pnpm" -ErrorAction SilentlyContinue

# Confirma que quedó limpio (no debería devolver nada)
where pnpm
```

## Instalar o actualizar pnpm (elige UN solo método)

| Método | Comando para INSTALAR | Comando para ACTUALIZAR |
|--------|----------------------|------------------------|
| **Corepack** (recomendado si tienes Node.js) | `corepack enable`<br>`corepack prepare pnpm@latest --activate` | `corepack prepare pnpm@latest --activate` |
| **Script oficial de pnpm** | `iwr https://get.pnpm.org/install.ps1 -useb \| iex` | `iwr https://get.pnpm.org/install.ps1 -useb \| iex` |
| **Winget** | `winget install pnpm` | `winget upgrade pnpm` |

## Verificar después de instalar
```powershell
where pnpm
pnpm --version
pnpm doctor
```

## Cómo actualizar pnpm en el futuro
Nunca uses `pnpm self-update`. Usa el mismo método con el que instalaste (consulta la tabla anterior).

## Usar Angular con pnpm
Dentro de tu proyecto Angular:

```powershell
# Instalar dependencias
pnpm install

# Ejecutar Angular CLI (resuelto localmente en el proyecto)
pnpm dlx ng serve

# O instalar Angular CLI como dependencia de desarrollo
pnpm add -D @angular/cli
pnpm ng serve
```

## Variable de entorno importante
- `PNPM_HOME`: normalmente `C:\Users\<tu-usuario>\AppData\Local\pnpm`
- Asegúrate de que `%PNPM_HOME%\bin` esté en tu `PATH`.

## Solución de problemas comunes

### 1. Store directory sin permisos de escritura (`D:\.pnpm-store`)
Si `pnpm doctor` muestra algo como:

```
✗ Store directory: no write access to D:\.pnpm-store\v11
```

Significa que pnpm intenta usar una carpeta en `D:` donde no tiene permisos de escritura. Esto rompe `pnpm install` y, por ende, `ng serve` en proyectos Angular.

**Solución:** Forzar el store a una ruta dentro de tu perfil de usuario:

```powershell
pnpm config set store-dir "$env:LOCALAPPDATA\pnpm-store"
```

Luego verifica con `pnpm config list` que `storeDir` apunte a `C:\Users\<tu-usuario>\AppData\Local\pnpm-store`.

### 2. `where pnpm` no devuelve nada pero `pnpm --version` sí funciona
Es normal cuando usas **Corepack**: los shims de pnpm pueden no ser visibles para `where.exe`, pero el comando funciona igual. No es un error.

### 3. Error EPERM al activar Corepack en `C:\Program Files\nodejs`
Si ves algo como:

```
EPERM: operation not permitted, open 'C:\Program Files\nodejs\pnpm'
```

Es porque Corepack intenta crear shims en la carpeta de Node.js y requiere permisos de administrador. Puedes:
- Ejecutar PowerShell como **Administrador**, o
- Usar el **script oficial** en su lugar: `iwr https://get.pnpm.org/install.ps1 -useb | iex`

### 4. Nunca uses `pnpm self-update`
En Windows, `pnpm self-update` puede crear carpetas o archivos en ubicaciones inesperadas (como la raíz de `D:`) y romper el PATH. Usa siempre:

```powershell
# Si usas Corepack
corepack prepare pnpm@latest --activate

# O si usas Winget
winget upgrade pnpm
```
