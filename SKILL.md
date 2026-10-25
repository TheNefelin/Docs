# SKILL.md — Documentación mejorada en Markdown

## Objetivo

Establecer pautas para crear y mejorar archivos `.md` en este repositorio, asegurando claridad, estructura consistente y buena presentación visual.

## Estructura recomendada para cada `.md`

1. **Título principal** (`#`) descriptivo y corto.
2. **Breve introducción** (1-2 oraciones) explicando de qué trata el documento.
3. **Tabla de contenido** opcional para documentos largos (>500 palabras).
4. **Secciones** con jerarquía lógica (`##`, `###`).
5. **Bloques de código** con lenguaje especificado (ej. ````pwsh`, ````bash`, ````sql`, ````yaml`).
6. **Tablas** cuando se comparen opciones, versiones o pasos.
7. **Advertencias o notas** usando bloques `> **Nota:**` o `> **Advertencia:**`.
8. **Enlaces relativos** a otros archivos del repo cuando sea pertinente.

## Reglas de estilo

- Idioma: español neutro, sin voseo.
- Términos técnicos en inglés cuando sea el estándar (ej. `commit`, `pull request`, `pipeline`, `store-dir`).
- No usar negritas excesivas; reservar para conceptos clave.
- Mantener oraciones cortas y directas.
- Verificar que todos los enlaces internos funcionen antes de commitear.

## Ejemplo de bloque de comando PowerShell

```pwsh
pnpm config set store-dir "$env:LOCALAPPDATA\pnpm-store"
```

## Checklist antes de commitear una documentación

- [ ] Título claro y descriptivo
- [ ] Jerarquía de encabezados consistente (`#` → `##` → `###`)
- [ ] Bloques de código con lenguaje especificado
- [ ] Enlaces internos verificados
- [ ] Idioma español neutro revisado
- [ ] No hay información sensible (contraseñas, tokens, rutas privadas reales)