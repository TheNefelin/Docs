# IAs

## MiniMax Gratis
- [Creat cuenta en OpenRouter](https://openrouter.ai/settings/keys )
- Crear API Key y Copiar
- Abir OpenCode
```sh
/connect
```
- Buscar OpenRouter y pegar la API Key
```sh
/models
```
- Seleccionar modelo (MiniMax M3), (GPT-4o mini), (DeepSeek V3.2), (DeepSeek V4 Flash) u otro modelo (free)

### Limpiar sesiones
```sh
opencode session list
```
```sh
opencode session delete <id>
```

### Rutas importantes
```sh
ls ~/.config/opencode
```
```sh
ls ~/.local/share/opencode
```

- opencode.jsonc
```json
{
  "$schema": "https://opencode.ai/config.json",
  "provider": {
    "openrouter": {
      "models": {
        "minimax/minimax-m3": {
          "limit": {
            "context": 32000,
            "output": 4000
          }
        }
      }
    }
  }
}
```

---