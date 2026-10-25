# Herramientas de IA gratuitas

## Agentes

### OpenCode
- **URL**: https://opencode.ai
- **Login**: NO (solo requiere API key de un proveedor)
- **Instalación**
```sh
pnpm add -g opencode-ai
```
- **Sesiones**
```sh
opencode session list       # Listar sesiones
opencode resume <id>        # Abrir sesión específica
opencode session delete <id> # Eliminar sesión específica
```

### Cline
- **URL**: https://cline.bot
- **Login**: SI (requiere `cline auth`)
- **Instalación**
```sh
pnpm add -g cline
```
- **Sesiones**
```sh
cline history               # Listar sesiones
cline --id <session-id>     # Abrir sesión específica
```
> Nota: Cline no expone un comando CLI documentado para eliminar sesiones. Los datos se almacenan en `~/.cline/data/sessions/`.

### Kilo
- **URL**: https://kilo.ai
- **Login**: NO (usa `/connect` dentro de la TUI para proveedores)
- **Instalación**
```sh
pnpm add -g @kilocode/cli
```
- **Sesiones**
```sh
kilo session list           # Listar sesiones
kilo --continue             # Abrir última sesión
kilo session delete <id>    # Eliminar sesión específica
```

## Proveedores gratuitos

### NVIDIA
- **URL**: https://build.nvidia.com
- Modelos gratuitos con límites: Nemotron 3 Ultra, Nemotron 3.5 Super VL, Nemotron 3 Nano Omni.

### OpenRouter
- **URL**: https://openrouter.ai
- Modelos gratuitos: Laguna S 2.1, Flint, Xing4.0 29B A4B, Nemotron 3 Ultra, entre otros.
- [Colección de modelos gratuitos](https://openrouter.ai/collections/free-models)
