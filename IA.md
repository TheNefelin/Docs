# OpenCode

## Usar modelos gratuitos con OpenRouter
- [Crear una cuenta y obtener una API Key en OpenRouter](https://openrouter.ai/settings/keys)
- Copiar la API Key.
- Abrir OpenCode.
```sh
/connect
```
- Buscar OpenRouter y pegar la API Key.
```sh
/models
```
- Seleccionar un modelo gratuito, como MiniMax M3, GPT-4o mini, DeepSeek V3.2 o DeepSeek V4 Flash.

## Sesiones
```sh
opencode session list
opencode resume <id>
opencode --continue
opencode -c
opencode --session <id>
opencode -s <id>
opencode session delete <id>
```

## Rutas importantes
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

## NVIDIA Build
[NVIDIA Build](https://build.nvidia.com)
- Obtener una API Key.
