# DEVELOPMENT.md — mapa del proyecto Docs

## ¿Qué es este repositorio?

Repositorio de documentación técnica en Markdown. No es un proyecto de aplicación; no tiene `build`, tests de unidad ni dependencias de runtime. Cada archivo `.md` es una guía independiente.

## Stack

- Markdown (documentación)
- Git + GitHub (versionado)
- pnpm (solo mencionado en la guía `pnpm.md`; no se usa para instalar nada en este repo)

## Mapa: área → archivo → cuándo tocarlo

| Área | Archivo | Cuándo editar |
|------|---------|---------------|
| .NET Entity Framework + PostgreSQL | `.NET_EntityPgSql.md` | Cambios en conexión, migraciones, consultas a Postgres |
| .NET: algoritmos | `.NET_AlgorithmsProficiencyTest.md` | Ejercicios de algoritmos en C# |
| .NET: backend | `.NET_BackendCodingChallenge.md` | Retos de backend .NET |
| .NET: C# | `.NET_C#ProficiencyTest.md` | Pruebas de conocimiento C# |
| .NET: orientación a objetos | `.NET_ObjectOrientedProgrammingProficiencyTest.md` | Principios de POO |
| .NET: VB.NET | `.NET_VB.ProficiencyTest.md` | Pruebas de VB.NET |
| .NET Interview | `.NET_Interview.md` | Preguntas de entrevista .NET |
| Clean Architecture | `CleanArchitecture.md` | Diseño de arquitectura limpia |
| CSS | `CSS.md` | Estilos y CSS |
| Docker | `Docker.md` | Contenedores y Docker |
| ERP | `ERP.md` | Documentación de ERP |
| IA | `IA.md` | Inteligencia artificial |
| Guía de entrevista | `Interview.md` | Guía general de entrevista |
| Linode | `Linode.md` | Cloud Linode |
| Linux | `Linux.md` | Comandos y administración Linux |
| Next.js | `NextJS.md` | Framework Next.js |
| Oracle Cloud Free Tier | `ORACLE_CLOUD_FREE_TIER.md` | Oracle Cloud Free Tier |
| Oracle SQL | `OracleSQL.md` | SQL para Oracle |
| pnpm | `pnpm.md` | Guía de instalación y actualización de pnpm en Windows |
| Principios SOLID | `SOLID.md` | Principios SOLID |
| SQL Server | `SqlServer.md` | SQL Server y T-SQL |
| Video | `Video.md` | Multimedia y video |
| Windows | `Windows.md` | Administración y configuración Windows |

## Comandos de verificación

No hay `build` ni `test` de aplicación. Para verificar cambios en documentación:

1. Abre el archivo `.md` afectado y confirma que las rutas, comandos y versiones son correctas.
2. Si se menciona un comando de terminal, ejecútalo en el entorno real antes de afirmarlo.

## Cómo contribuir

1. Crea una rama desde `main`.
2. Edita el archivo `.md` correspondiente según el mapa anterior.
3. Abre un Pull Request describiendo el cambio.
4. Mantén el idioma español neutro y los términos técnicos en inglés cuando sea estándar.