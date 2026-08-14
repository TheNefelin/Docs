# ERP RRHH + Asistencia + Remuneraciones + ERP Financiero

**Documento maestro de desarrollo**

* **Proyecto:** ERP para organizaciones chilenas
* **Objetivo:** Reemplazar progresivamente las funcionalidades actualmente cubiertas por Defontana + Workera mediante una única plataforma.
* **País objetivo inicial:** Chile
* **Arquitectura inicial:** Modular Monolith
* **Backend:** .NET 10 / ASP.NET Core 10
* **Frontend:** Angular 22 + TypeScript
* **Base de datos:** PostgreSQL 18
* **Contenedores:** Docker
* **Cloud:** arquitectura cloud-ready, despliegue inicial simple
* **Estado:** Diseño / Inicio de MVP
* **Última actualización:** 2026-08-12

---

# 1. Visión del proyecto

## 1.1 Objetivo general

Construir una plataforma ERP modular orientada inicialmente a organizaciones chilenas, especialmente fundaciones, corporaciones y organizaciones sin fines de lucro, que permita centralizar:

* Personas
* Recursos humanos
* Contratos
* Asistencia
* Turnos
* Horas extraordinarias
* Vacaciones
* Permisos
* Remuneraciones
* Gratificaciones
* Finiquitos
* Documentos
* Firma electrónica
* Libro de Remuneraciones Electrónico
* Integraciones previsionales
* Contabilidad
* Presupuesto
* Centros de costo
* Compras
* Ventas
* Inventario
* Proveedores
* Clientes
* Reportes
* Auditoría

El producto debe permitir que una organización pueda comenzar utilizando únicamente RRHH + asistencia + remuneraciones y posteriormente incorporar las funcionalidades ERP.

---

# 2. Principios fundamentales

## 2.1 Principio 1 — Modularidad

El sistema debe dividirse en módulos de negocio claramente delimitados.

Los módulos no deben depender directamente de implementaciones internas de otros módulos.

Ejemplo:

```text
Payroll
   |
   +--> People
   +--> Contracts
   +--> Attendance
```

Pero Payroll no debe acceder directamente a tablas internas arbitrarias de Attendance.

Debe existir un contrato de aplicación o dominio.

---

## 2.2 Principio 2 — Modular Monolith primero

No se utilizarán microservicios durante el MVP.

La aplicación será un único deployment inicialmente:

```text
Angular
   |
   v
ASP.NET Core
   |
   +-- Identity
   +-- Organization
   +-- People
   +-- Contracts
   +-- Attendance
   +-- Payroll
   +-- Documents
   +-- Accounting
   +-- Purchasing
   +-- Sales
   +-- Inventory
   |
   v
PostgreSQL
```

Los límites internos deberán permitir separar posteriormente módulos específicos en servicios independientes si existe una necesidad real.

---

## 2.3 Principio 3 — La base de datos es fuente de verdad

PostgreSQL será la fuente de verdad transaccional.

Redis, cache, búsquedas externas y otros mecanismos nunca deberán considerarse fuente primaria de información.

---

## 2.4 Principio 4 — Todo cálculo importante debe ser reproducible

Especialmente remuneraciones.

Una liquidación no debe almacenar únicamente:

```text
neto = 850000
```

Debe ser posible reconstruir por qué se obtuvo ese resultado.

Debe registrarse:

* período
* trabajador
* contrato
* haberes
* descuentos
* reglas utilizadas
* versión de reglas
* parámetros
* topes
* resultados intermedios
* resultado final

---

## 2.5 Principio 5 — Las reglas laborales no deben estar hardcodeadas

No se debe implementar:

```csharp
if (isNonProfit)
{
    gratification = 0;
}
```

como lógica permanente.

Debe existir un motor de reglas.

Ejemplo conceptual:

```text
Payroll Rule
    Code
    Version
    EffectiveFrom
    EffectiveTo
    Priority
    Configuration
    Formula
```

Esto permite mantener diferentes versiones de reglas por período.

---

## 2.6 Principio 6 — Histórico inmutable

Una remuneración cerrada no debe cambiar silenciosamente.

Si se modifica una información histórica:

* debe existir auditoría
* debe quedar registrado quién modificó
* debe registrarse cuándo
* debe registrarse el valor anterior
* debe registrarse el valor nuevo
* cuando corresponda, debe generarse una nueva versión o reliquidación

---

## 2.7 Principio 7 — Multi-tenant desde el diseño

El sistema debe poder soportar múltiples organizaciones.

Todas las entidades que pertenezcan a una organización deberán estar asociadas a un `TenantId` o equivalente.

No se debe construir primero como aplicación de una sola organización y posteriormente intentar convertirla a SaaS.

---

# 3. Stack tecnológico

## 3.1 Backend

```text
.NET 10
ASP.NET Core 10
C# 14
Entity Framework Core 10
OpenAPI
REST
```

.NET 10 será la versión objetivo de producción.

No utilizar versiones preview de .NET para producción.

---

# 4. Frontend

```text
Angular 22
TypeScript
Angular Material
Angular CDK
RxJS
Signals
```

Angular Material será utilizado como base de componentes administrativos.

Se construirá una capa visual propia sobre Material para mantener consistencia.

---

# 5. Base de datos

## 5.1 Motor

```text
PostgreSQL 18
```

## 5.2 Reglas

* No utilizar SQL propietario innecesariamente.
* Mantener compatibilidad con PostgreSQL estándar.
* Usar migraciones EF Core.
* Las migraciones deben versionarse en Git.
* Nunca modificar manualmente producción sin registrar el cambio.
* Utilizar índices explícitos.
* Utilizar constraints de base de datos cuando corresponda.

---

# 6. Infraestructura

## 6.1 Desarrollo

Todo desarrollador debe poder levantar el entorno mediante Docker.

```text
Docker Compose
    |
    +-- PostgreSQL
    +-- API
    +-- Frontend
    +-- Mail testing
```

Redis será incorporado cuando exista una necesidad concreta.

---

## 6.2 Ambientes

Debe existir separación conceptual entre:

```text
Development
Staging
Production
```

Nunca desarrollar directamente sobre producción.

---

# 7. Arquitectura general

```text
                         INTERNET
                             |
                             v
                    +----------------+
                    | Reverse Proxy  |
                    | HTTPS / WAF    |
                    +-------+--------+
                            |
              +-------------+-------------+
              |                           |
              v                           v
        Angular SPA                 ASP.NET Core
                                      API
                                        |
       +--------------------------------+-------------------------------+
       |                |               |               |               |
       v                v               v               v               v
    Identity        People         Attendance        Payroll        Documents
       |                |               |               |               |
       +----------------+---------------+---------------+---------------+
                                        |
                                        v
                                  PostgreSQL
                                        |
                         +--------------+--------------+
                         |                             |
                         v                             v
                    Object Storage                  Jobs
```

---

# 8. Estructura del repositorio

Propuesta:

```text
erp-platform/
│
├── docs/
│   ├── desarrollo.md
│   ├── architecture/
│   ├── adr/
│   ├── business/
│   ├── payroll/
│   └── api/
│
├── src/
│   │
│   ├── Backend/
│   │   ├── Erp.Api/
│   │   │
│   │   ├── Erp.BuildingBlocks/
│   │   │   ├── Domain/
│   │   │   ├── Application/
│   │   │   └── Infrastructure/
│   │   │
│   │   └── Modules/
│   │       ├── Identity/
│   │       ├── Organizations/
│   │       ├── People/
│   │       ├── Contracts/
│   │       ├── Attendance/
│   │       ├── Leave/
│   │       ├── Payroll/
│   │       ├── Documents/
│   │       ├── Accounting/
│   │       ├── Purchasing/
│   │       ├── Sales/
│   │       ├── Inventory/
│   │       └── Reporting/
│   │
│   └── Frontend/
│       └── erp-web/
│
├── tests/
│   ├── Unit/
│   ├── Integration/
│   ├── Architecture/
│   └── E2E/
│
├── infrastructure/
│   ├── docker/
│   ├── compose/
│   └── deployment/
│
├── scripts/
│
├── .github/
│   └── workflows/
│
├── docker-compose.yml
├── .gitignore
├── README.md
└── LICENSE
```

---

# 9. Arquitectura backend

## 9.1 Capas

Cada módulo seguirá aproximadamente:

```text
Module
│
├── Domain
│
├── Application
│
├── Infrastructure
│
└── Presentation
```

---

## 9.2 Domain

Contiene:

* Entities
* Value Objects
* Domain Events
* Domain Services
* Business Rules
* Enumerations

No debe depender de:

* Entity Framework
* ASP.NET
* HTTP
* PostgreSQL
* Angular

---

## 9.3 Application

Contiene:

* Commands
* Queries
* DTOs
* Validators
* Application Services
* Interfaces
* Use Cases

Ejemplo:

```text
Payroll
├── Application
│   ├── Commands
│   │   ├── CreatePayrollRun
│   │   ├── CalculatePayroll
│   │   ├── ClosePayroll
│   │   └── RecalculatePayroll
│   │
│   └── Queries
│       ├── GetPayroll
│       ├── GetPayslip
│       └── GetPayrollSummary
```

---

## 9.4 Infrastructure

Contiene:

* EF Core
* DbContexts
* repositories cuando sean necesarios
* almacenamiento de archivos
* email
* integraciones externas
* servicios de terceros
* background jobs

---

# 10. API

La API será REST.

Ejemplo:

```text
/api/v1/organizations
/api/v1/employees
/api/v1/contracts
/api/v1/attendance
/api/v1/payroll
/api/v1/documents
```

Versionar APIs desde el inicio.

No crear endpoints sin definir:

* autorización
* validación
* respuesta
* errores
* auditoría cuando corresponda

---

# 11. Respuestas API

Formato recomendado:

```json
{
  "data": {},
  "errors": [],
  "meta": {}
}
```

Errores:

```json
{
  "data": null,
  "errors": [
    {
      "code": "PAYROLL_PERIOD_CLOSED",
      "message": "El período de remuneraciones está cerrado."
    }
  ]
}
```

No devolver excepciones internas directamente al cliente.

---

# 12. Identidad y seguridad

## 12.1 Autenticación

Inicialmente:

```text
ASP.NET Core Identity
```

La arquitectura debe permitir posteriormente integrar:

* Microsoft Entra ID
* Google Workspace
* otros proveedores OIDC

---

# 13. Autorización

Utilizar RBAC.

Ejemplo:

```text
Roles

SUPER_ADMIN
TENANT_ADMIN
HR_ADMIN
PAYROLL_ADMIN
ACCOUNTANT
SUPERVISOR
EMPLOYEE
AUDITOR
```

Permisos más específicos:

```text
employee.read
employee.create
employee.update
employee.delete

payroll.read
payroll.calculate
payroll.close
payroll.reopen

attendance.read
attendance.manage

accounting.read
accounting.post
```

La autorización debe basarse en permisos, no únicamente en nombres de roles.

---

# 14. Auditoría

Debe existir un módulo transversal de auditoría.

Entidad conceptual:

```text
AuditLog
---------
Id
TenantId
UserId
Timestamp
Action
EntityType
EntityId
OldValues
NewValues
IpAddress
UserAgent
CorrelationId
```

Ejemplo:

```text
PAYROLL_UPDATED
EMPLOYEE_UPDATED
CONTRACT_CREATED
PAYROLL_CLOSED
PAYROLL_REOPENED
BANK_ACCOUNT_CHANGED
```

Cambios sensibles deben auditarse obligatoriamente.

---

# 15. Multi-tenancy

## 15.1 Modelo inicial

Base de datos compartida:

```text
Tenant A
Tenant B
Tenant C
```

Las tablas multi-tenant tendrán:

```text
TenantId
```

Ejemplo:

```text
Employee
---------
Id
TenantId
PersonId
EmployeeNumber
...
```

---

## 15.2 Regla crítica

Nunca aceptar un `TenantId` arbitrario enviado por el frontend como mecanismo de autorización.

El tenant debe determinarse desde:

* contexto autenticado
* claims
* membership
* autorización del usuario

---

# 16. Módulos funcionales

## Fase MVP

```text
Identity
Organizations
People
Contracts
Attendance
Leave
Payroll
Documents
Audit
```

## Fase V1

```text
Electronic Payroll Book
Finiquitos
Integraciones previsionales
Firma electrónica
Portal trabajador
Reporting
Cost Centers
Budget
```

## Fase ERP

```text
Accounting
Purchasing
Suppliers
Sales
Customers
Inventory
Assets
Banking
Financial Reporting
```

---

# 17. Módulo Organizations

Responsabilidad:

* organizaciones
* entidades legales
* sucursales
* centros de costo
* departamentos
* cargos
* configuraciones laborales

Entidades iniciales:

```text
Tenant
Organization
LegalEntity
Branch
Department
Position
CostCenter
```

---

# 18. Módulo People

Separar:

```text
Person
Employee
User
```

Una persona no necesariamente es un usuario.

Una persona puede tener múltiples relaciones laborales a lo largo del tiempo.

---

## 18.1 Person

```text
Person
------
Id
TenantId
Rut
FirstName
MiddleName
LastName
SecondLastName
BirthDate
Nationality
Email
Phone
Address
...
```

---

## 18.2 Employee

```text
Employee
--------
Id
TenantId
PersonId
EmployeeNumber
Status
HireDate
TerminationDate
DepartmentId
PositionId
CostCenterId
```

---

# 19. Contratos

Entidades:

```text
EmploymentContract
ContractVersion
ContractType
WorkSchedule
SalaryAgreement
```

Un contrato debe ser versionable.

No sobrescribir silenciosamente información contractual histórica.

---

# 20. Asistencia

Responsabilidades:

* marcaciones
* horarios
* turnos
* atrasos
* salidas anticipadas
* ausencias
* horas extraordinarias
* feriados
* jornadas

Entidades:

```text
AttendanceRecord
AttendanceDay
Shift
Schedule
OvertimeRequest
OvertimeApproval
Holiday
```

---

# 21. Vacaciones y permisos

Entidades:

```text
LeaveRequest
LeaveType
LeaveBalance
LeaveApproval
VacationPeriod
```

Estados:

```text
Draft
Pending
Approved
Rejected
Cancelled
```

---

# 22. Remuneraciones

Este es uno de los módulos críticos del sistema.

Debe manejar:

* sueldo base
* haberes
* descuentos
* bonos
* asignaciones
* horas extras
* gratificación
* impuestos
* previsión
* salud
* seguros
* otros descuentos
* anticipos
* préstamos
* retenciones
* finiquitos

---

# 23. Modelo de Payroll

Entidades iniciales:

```text
PayrollPeriod
PayrollRun
PayrollEmployee
PayrollItem
PayrollCalculation
PayrollRule
PayrollRuleVersion
PayrollAdjustment
Payslip
```

---

# 24. PayrollPeriod

Ejemplo:

```text
2026-08
```

Estados:

```text
OPEN
CALCULATING
CALCULATED
APPROVED
CLOSED
```

Regla:

Un período cerrado no puede modificarse directamente.

---

# 25. PayrollItem

Cada concepto de remuneración debe ser un item.

Ejemplos:

```text
BASE_SALARY
OVERTIME
BONUS
ALLOWANCE
GRATIFICATION
TAX
AFP
HEALTH
UNEMPLOYMENT_INSURANCE
LOAN
ADVANCE
OTHER_DEDUCTION
```

Propiedades:

```text
Code
Name
Type
Taxable
Imponible
Tributable
AffectsNet
AccountingAccount
Formula
```

---

# 26. Motor de reglas

El motor de reglas será una pieza estratégica del producto.

Debe permitir:

```text
Rule
RuleVersion
RuleParameter
RuleCondition
RuleResult
```

Cada regla tendrá vigencia.

```text
EffectiveFrom
EffectiveTo
```

---

# 27. Gratificación

La gratificación debe ser configurable.

Tipos:

```text
NOT_APPLICABLE
LEGAL_ART_47
LEGAL_ART_50
CONVENTIONAL
CUSTOM
```

Nunca utilizar un monto ficticio para representar "no aplica".

Ejemplo:

```text
EmployerConfiguration
---------------------
GratificationMode = NOT_APPLICABLE
```

Resultado:

```text
Aplica = false
Monto = 0
```

Esto es diferente conceptualmente de:

```text
Aplica = true
Monto = 0
```

---

# 28. Regla para organizaciones sin fines de lucro

El sistema debe permitir configurar:

```text
OrganizationType
----------------
FOR_PROFIT
NON_PROFIT
COOPERATIVE
OTHER
```

Pero el tipo de organización **no debe determinar automáticamente todas las obligaciones laborales**.

Debe utilizarse como dato de contexto para el motor de reglas.

La configuración efectiva debe poder ser revisada por un administrador autorizado.

---

# 29. Fórmulas

Las fórmulas no deben estar distribuidas por cientos de clases.

Conceptualmente:

```text
Formula
--------
Code
Expression
Version
EffectiveFrom
EffectiveTo
```

Ejemplo conceptual:

```text
OVERTIME_AMOUNT =
    BASE_HOURLY_RATE
    * OVERTIME_FACTOR
    * OVERTIME_HOURS
```

El motor debe controlar:

* tipos
* redondeos
* dependencias
* límites
* errores
* trazabilidad

---

# 30. Precisión monetaria

Nunca utilizar `float` o `double` para dinero.

Usar:

```csharp
decimal
```

y definir explícitamente:

* escala
* redondeo
* momento del redondeo

Ejemplo:

```text
Money
------
Amount
Currency
```

Inicialmente:

```text
CLP
```

---

# 31. Auditoría de cálculos

Cada cálculo de remuneraciones debe poder producir un detalle:

```text
Payroll Calculation
-------------------

Sueldo base
+ Horas extras
+ Bonos
+ Asignaciones
+ Gratificación
-------------------
Total haberes

- AFP
- Salud
- Impuesto
- Otros descuentos
-------------------
Total descuentos

= Líquido
```

Además:

```text
RuleVersion
```

debe quedar registrada.

---

# 32. Recalcular remuneración

Debe existir explícitamente:

```text
Calculate
Recalculate
Approve
Close
Reopen
```

No mezclar estas operaciones.

---

# 33. Cierre de remuneraciones

Cuando un período se cierra:

```text
PayrollPeriod = CLOSED
```

Se debe impedir:

* editar conceptos
* modificar contratos que afecten el período
* modificar asistencia histórica sin proceso de reliquidación

Si existe una corrección:

```text
Correction
    |
    v
Adjustment / Recalculation
```

---

# 34. Liquidaciones

Entidad:

```text
Payslip
```

Debe almacenar:

* trabajador
* período
* número
* versión
* datos calculados
* PDF
* hash
* fecha de generación

---

# 35. Finiquitos

Debe ser un módulo separado.

Entidades:

```text
Termination
Settlement
SettlementItem
SettlementRule
```

Debe manejar diferentes causales y reglas.

Nunca mezclar finiquitos directamente con el cálculo mensual normal.

---

# 36. Documentos

Sistema central de documentos.

Tipos:

```text
CONTRACT
CONTRACT_ATTACHMENT
PAYSLIP
SETTLEMENT
CERTIFICATE
PERSON_DOCUMENT
OTHER
```

Los archivos físicos se almacenarán en object storage.

PostgreSQL almacena metadata.

---

# 37. Hash de documentos

Cada documento importante debe poder almacenar:

```text
SHA-256
```

Esto permite verificar integridad.

---

# 38. Firma electrónica

La firma electrónica será una integración desacoplada.

Interfaz conceptual:

```csharp
interface ISignatureProvider
{
    Task<SignatureRequest> CreateRequest(...);
    Task<SignatureStatus> GetStatus(...);
    Task<SignatureDocument> DownloadSignedDocument(...);
}
```

Esto permite cambiar proveedor sin modificar el dominio.

---

# 39. LRE

El Libro de Remuneraciones Electrónico debe implementarse como módulo de integración.

No mezclar su lógica directamente con la UI de Payroll.

Arquitectura:

```text
Payroll
   |
   v
LreGenerator
   |
   v
LRE Export
   |
   v
Validation
   |
   v
Submission / Integration
```

---

# 40. Integraciones externas

Las integraciones deberán estar aisladas.

Ejemplo:

```text
Integrations
├── Previred
├── SII
├── DT
├── Banking
├── Signature
└── Email
```

No colocar llamadas HTTP externas dentro de entidades de dominio.

---

# 41. Background Jobs

Utilizar un sistema de jobs para:

* generación de documentos
* generación de liquidaciones
* envío de emails
* procesos masivos
* importaciones
* exportaciones
* cálculos largos
* sincronizaciones

Ejemplos:

```text
GeneratePayslipsJob
GenerateLreJob
SendPayslipEmailJob
ImportAttendanceJob
ProcessIntegrationJob
```

La tecnología concreta de jobs puede ser Hangfire inicialmente.

---

# 42. Cache

Redis no es obligatorio en el MVP.

Se incorporará cuando exista una necesidad comprobada.

Candidatos:

```text
Session
Permissions
ReferenceData
RateLimiting
DistributedLocks
ExpensiveQueries
```

No cachear información sensible sin una estrategia clara de expiración e invalidación.

---

# 43. Frontend

Estructura:

```text
src/app/
│
├── core/
│   ├── auth/
│   ├── http/
│   ├── guards/
│   ├── interceptors/
│   └── services/
│
├── shared/
│   ├── components/
│   ├── forms/
│   ├── tables/
│   ├── pipes/
│   └── utils/
│
├── layout/
│   ├── shell/
│   ├── sidebar/
│   └── topbar/
│
└── features/
    ├── dashboard/
    ├── people/
    ├── contracts/
    ├── attendance/
    ├── leave/
    ├── payroll/
    ├── documents/
    └── administration/
```

---

# 44. Angular

Preferir:

* standalone components
* signals cuando aporten claridad
* reactive forms
* lazy loading
* route guards
* HTTP interceptors
* typed APIs
* componentes reutilizables

Evitar:

* lógica de negocio laboral dentro de componentes
* duplicación de formularios
* servicios gigantes
* estado global innecesario

---

# 45. Estado

No utilizar NgRx automáticamente.

Inicialmente:

```text
Signals
Services
RxJS
```

Utilizar NgRx únicamente si el dominio demuestra una necesidad real de estado global complejo.

---

# 46. Formularios

Los formularios deben ser:

* tipados
* reutilizables
* validados
* accesibles
* consistentes

Componentes comunes:

```text
RutInput
MoneyInput
PercentageInput
DateInput
EmployeeSelector
DepartmentSelector
CostCenterSelector
FileUpload
```

---

# 47. Tablas

El ERP tendrá muchas tablas.

Debe existir un componente común:

```text
DataTable
```

Características:

* paginación
* ordenamiento
* filtros
* búsqueda
* selección
* columnas configurables
* exportación
* responsive cuando corresponda

---

# 48. Diseño UX

Principios:

* navegación consistente
* pocas acciones ocultas
* estados claros
* validación inmediata
* mensajes comprensibles
* confirmación de acciones críticas
* indicadores de proceso
* trazabilidad

Acciones críticas:

```text
Cerrar remuneraciones
Eliminar empleado
Reabrir período
Eliminar documento
Anular operación
```

requieren confirmación.

---

# 49. Testing

## Unit Tests

Obligatorios para:

* motor de remuneraciones
* reglas
* cálculos
* fechas
* redondeos
* permisos
* dominio

---

# 50. Integration Tests

Probar:

* PostgreSQL
* API
* autenticación
* persistencia
* transacciones
* integraciones

Idealmente utilizar contenedores para pruebas.

---

# 51. E2E

Utilizar Playwright.

Casos críticos:

```text
Login
Create Employee
Create Contract
Register Attendance
Calculate Payroll
Approve Payroll
Close Payroll
Generate Payslip
Download Document
```

---

# 52. Tests de remuneraciones

Esta es una prioridad máxima.

Cada regla importante debe tener:

```text
Input
Expected Result
```

Ejemplo:

```text
Caso:
Sueldo base = X
Horas extras = Y
Gratificación = No aplica

Resultado esperado:
...
```

Debe existir una suite extensa de casos.

---

# 53. Test de regresión laboral

Cada cambio al motor de reglas debe ejecutar automáticamente todos los casos conocidos.

Nunca modificar una regla laboral sin ejecutar regresión.

---

# 54. Arquitectura de errores

Errores de negocio deben tener códigos.

Ejemplos:

```text
EMPLOYEE_NOT_FOUND
CONTRACT_NOT_ACTIVE
PAYROLL_PERIOD_CLOSED
PAYROLL_ALREADY_CALCULATED
INVALID_GRATIFICATION_CONFIGURATION
UNAUTHORIZED_PAYROLL_OPERATION
INVALID_ATTENDANCE_RECORD
```

---

# 55. Logging

Utilizar logging estructurado.

Cada request debe tener:

```text
CorrelationId
```

Ejemplo:

```text
CorrelationId:
9d2f...
```

Permite rastrear:

```text
Frontend
 -> API
 -> Service
 -> Database
 -> Job
```

---

# 56. Observabilidad

Objetivo futuro:

```text
OpenTelemetry
Prometheus
Grafana
Loki
```

Métricas:

* requests
* errores
* latencia
* jobs
* database
* CPU
* memoria
* integraciones

---

# 57. Docker

## Desarrollo

Servicios mínimos:

```yaml
services:
  postgres:
    image: postgres:18

  api:
    build: ./src/Backend

  web:
    build: ./src/Frontend
```

No incluir servicios innecesarios en el primer MVP.

---

# 58. Configuración

Nunca almacenar secretos en Git.

Incorrecto:

```text
appsettings.json

Password=123456
```

Correcto:

```text
Environment Variables
Secret Store
```

---

# 59. Variables de entorno

Ejemplo:

```text
ConnectionStrings__Default
Jwt__Authority
Storage__Connection
Email__Host
Email__ApiKey
```

---

# 60. CI/CD

GitHub Actions.

Pipeline mínimo:

```text
Push
  |
  v
Build
  |
  v
Unit Tests
  |
  v
Integration Tests
  |
  v
Frontend Tests
  |
  v
Docker Build
  |
  v
Security Checks
  |
  v
Deploy Staging
```

Producción requerirá aprobación.

---

# 61. Git

Branches:

```text
main
develop
feature/*
fix/*
hotfix/*
```

Pull Request obligatorio.

No hacer push directo a `main`.

---

# 62. Commits

Formato recomendado:

```text
feat(payroll): add payroll calculation
fix(attendance): correct overtime calculation
refactor(people): simplify employee service
test(payroll): add gratification scenarios
docs(architecture): update payroll ADR
```

---

# 63. Definition of Done

Una funcionalidad no se considera terminada hasta cumplir:

```text
[ ] Código implementado
[ ] Validaciones
[ ] Tests unitarios
[ ] Tests integración si corresponde
[ ] UI implementada
[ ] Autorización
[ ] Auditoría si corresponde
[ ] Documentación
[ ] Migración DB si corresponde
[ ] Sin errores de lint
[ ] Pull Request revisado
[ ] CI exitoso
```

---

# 64. Migraciones

Toda modificación de base:

```text
Migration
```

debe estar versionada.

Nunca eliminar una columna en producción sin revisar:

* datos históricos
* reportes
* integraciones
* migración
* rollback

---

# 65. Backups

Producción deberá contar con:

* backups automáticos
* retención
* almacenamiento separado
* pruebas de restauración

Un backup que nunca se ha restaurado no debe considerarse confiable.

---

# 66. Seguridad de datos

Datos sensibles:

* RUT
* remuneraciones
* cuentas bancarias
* contratos
* documentos personales

deben tratarse como información sensible.

Aplicar:

* TLS
* encryption at rest
* mínimo privilegio
* RBAC
* auditoría
* backups
* rotación de secretos

---

# 67. No construir todavía

Durante el MVP NO se desarrollará:

```text
Microservices
Kubernetes
Data Lake
AI
Machine Learning
Blockchain
Advanced BI
Mobile native apps
```

Estas funcionalidades no son prioritarias para validar el producto.

---

# 68. Roadmap

## Fase 0 — Arquitectura

```text
[ ] Crear repositorio
[ ] Crear documentación
[ ] Crear solución .NET
[ ] Crear Angular
[ ] Crear Docker Compose
[ ] Configurar PostgreSQL
[ ] Configurar CI
[ ] Definir arquitectura
[ ] Definir multi-tenancy
```

---

# 69. Fase 1 — Identity

```text
[ ] Usuarios
[ ] Login
[ ] Logout
[ ] Password reset
[ ] Roles
[ ] Permissions
[ ] Tenant membership
[ ] Audit
```

---

# 70. Fase 2 — Organizations

```text
[ ] Tenant
[ ] Organization
[ ] Legal Entity
[ ] Branch
[ ] Department
[ ] Position
[ ] Cost Center
```

---

# 71. Fase 3 — People

```text
[ ] Person
[ ] Employee
[ ] Datos personales
[ ] Datos laborales
[ ] Datos bancarios
[ ] Contactos
[ ] Documentos
```

---

# 72. Fase 4 — Contracts

```text
[ ] Employment Contract
[ ] Contract versions
[ ] Salary
[ ] Work schedule
[ ] Position
[ ] Cost center
[ ] Contract history
```

---

# 73. Fase 5 — Attendance

```text
[ ] Schedules
[ ] Shifts
[ ] Attendance records
[ ] Absences
[ ] Overtime
[ ] Approval
[ ] Monthly summary
```

---

# 74. Fase 6 — Leave

```text
[ ] Vacation
[ ] Permissions
[ ] Leave balances
[ ] Approval workflow
```

---

# 75. Fase 7 — Payroll

```text
[ ] Payroll period
[ ] Payroll run
[ ] Payroll items
[ ] Rules
[ ] Calculation engine
[ ] Validation
[ ] Approval
[ ] Closing
[ ] Payslip
```

---

# 76. Fase 8 — Gratificación

```text
[ ] Configuration
[ ] NOT_APPLICABLE
[ ] Legal rules
[ ] Conventional rules
[ ] Rule versioning
[ ] Calculation
[ ] Tests
[ ] Audit
```

---

# 77. Fase 9 — Integraciones laborales

```text
[ ] LRE
[ ] Previred
[ ] SII
[ ] DT
[ ] Electronic signature
```

Cada integración debe implementarse independientemente.

---

# 78. Fase 10 — Portal trabajador

```text
[ ] Profile
[ ] Payslips
[ ] Contracts
[ ] Documents
[ ] Attendance
[ ] Vacation
[ ] Requests
```

---

# 79. Fase 11 — Finanzas

```text
[ ] Chart of accounts
[ ] Accounting periods
[ ] Journal entries
[ ] Cost centers
[ ] Budget
[ ] Accounts payable
[ ] Accounts receivable
```

---

# 80. Fase 12 — Compras

```text
[ ] Suppliers
[ ] Purchase requests
[ ] Purchase orders
[ ] Receipts
[ ] Supplier invoices
[ ] Approval
```

---

# 81. Fase 13 — Ventas

```text
[ ] Customers
[ ] Quotes
[ ] Sales orders
[ ] Invoices
[ ] Receivables
```

---

# 82. Fase 14 — Inventario

```text
[ ] Products
[ ] Warehouses
[ ] Stock
[ ] Movements
[ ] Adjustments
[ ] Transfers
[ ] Inventory reports
```

---

# 83. Fase 15 — ERP integrado

El objetivo final:

```text
HR
 |
Payroll
 |
Accounting
 |
Cost Centers
 |
Budget
 |
Purchasing
 |
Inventory
 |
Sales
```

La remuneración deberá poder generar movimientos contables automáticamente.

---

# 84. Contabilidad y remuneraciones

Ejemplo conceptual:

```text
Payroll
   |
   v
Accounting Integration
   |
   +-- Salary Expense
   +-- Employer Contributions
   +-- Tax Liability
   +-- Social Security Liability
   +-- Bank/Payment
```

El módulo Payroll no debe conocer detalles internos del ledger.

Debe emitir un evento o comando de integración.

---

# 85. Eventos de dominio

Ejemplos:

```text
EmployeeCreated
ContractCreated
ContractActivated
AttendanceApproved
PayrollCalculated
PayrollApproved
PayrollClosed
PayslipGenerated
EmployeeTerminated
```

Inicialmente pueden ejecutarse dentro del mismo proceso.

La arquitectura debe permitir posteriormente mensajería externa.

---

# 86. Integración interna

No utilizar HTTP entre módulos del mismo monolito.

Incorrecto:

```text
Payroll
 -> HTTP
 -> Attendance
```

Preferir:

```text
Payroll
 -> Application Interface
 -> Attendance
```

o:

```text
Payroll
 -> Domain/Application Event
```

---

# 87. CQRS

Se utilizará CQRS de manera pragmática.

Separar:

```text
Commands
Queries
```

No crear una infraestructura CQRS extremadamente compleja.

---

# 88. Repositories

No crear repository genérico:

```csharp
IRepository<T>
```

para todas las entidades automáticamente.

Cada módulo debe decidir si necesita repository específico.

EF Core será utilizado directamente en muchos casos de aplicación.

---

# 89. Entity Framework

Preferir:

```text
DbContext
EntityTypeConfiguration
Migrations
```

Configuraciones separadas:

```text
EmployeeConfiguration
ContractConfiguration
PayrollConfiguration
```

No colocar toda la configuración en `OnModelCreating`.

---

# 90. Base de datos inicial

Tablas conceptuales:

```text
tenants
organizations
legal_entities
branches
departments
positions
cost_centers

persons
employees
employee_bank_accounts
employee_contacts

employment_contracts
employment_contract_versions

attendance_records
shifts
schedules
overtime_requests

leave_types
leave_requests
leave_balances

payroll_periods
payroll_runs
payroll_employees
payroll_items
payroll_calculations

payroll_rules
payroll_rule_versions

payslips

documents
document_versions

users
roles
permissions
user_roles
role_permissions

audit_logs
```

---

# 91. IDs

Preferir UUID/UUIDv7 para entidades distribuidas.

Las claves públicas no deberían depender de IDs secuenciales expuestos.

Los números internos de documentos pueden utilizar secuencias independientes.

---

# 92. Fechas y horarios

Almacenar timestamps de manera consistente.

Regla:

```text
UTC para timestamps técnicos
Zona horaria configurada para la organización
```

Para Chile debe existir configuración de zona horaria.

Las fechas laborales deben distinguir entre:

```text
Date
DateTime
Instant
```

No utilizar `DateTime` indiscriminadamente.

---

# 93. RUT

Crear un Value Object:

```text
Rut
```

Debe manejar:

* validación
* normalización
* formato
* comparación

No repetir lógica de RUT en cada módulo.

---

# 94. Money

Crear Value Object o estructura equivalente:

```text
Money
```

Debe evitar errores de precisión.

---

# 95. Email

Validación centralizada.

No duplicar regex de email en múltiples módulos.

---

# 96. Estados

Evitar strings arbitrarios:

```text
"activo"
"Activo"
"ACTIVE"
```

Utilizar enums o códigos controlados.

---

# 97. Catálogos

Crear catálogos centralizados para conceptos que requieran configuración.

Ejemplos:

```text
ContractType
LeaveType
PayrollItemType
DocumentType
TerminationReason
```

---

# 98. Reportes

Los reportes deben ser un módulo separado.

No colocar SQL complejo dentro de controllers.

Arquitectura:

```text
Reporting
   |
   +-- Queries
   +-- Exporters
   +-- PDF
   +-- Excel
```

---

# 99. Exportaciones

Soportar:

```text
CSV
Excel
PDF
```

según módulo.

Los procesos grandes deben ejecutarse como jobs.

---

# 100. Importaciones

El sistema deberá permitir carga masiva.

Importaciones deben pasar por:

```text
Upload
   |
Validation
   |
Preview
   |
Errors
   |
Confirmation
   |
Import
```

Nunca importar directamente sin mostrar errores.

---

# 101. Caso específico: carga masiva de remuneraciones

Debe soportar:

```text
Employee
Period
Payroll Item
Amount
```

Y conceptos configurables.

Para gratificación:

```text
NOT_APPLICABLE
```

debe ser una condición válida cuando corresponda.

Nunca exigir un monto ficticio para satisfacer una validación técnica.

---

# 102. Idempotencia

Procesos como:

* importaciones
* generación LRE
* integraciones
* pagos
* jobs

deben ser idempotentes cuando corresponda.

Ejemplo:

```text
Import ID = ABC123
```

Si el job se ejecuta dos veces, no debe duplicar información.

---

# 103. Concurrencia

Debe controlarse especialmente:

```text
Payroll closing
Payroll recalculation
Inventory adjustments
Accounting posting
```

Utilizar transacciones y mecanismos de concurrencia apropiados.

---

# 104. Transacciones

Las operaciones críticas deben ser transaccionales.

Ejemplo:

```text
ClosePayroll
    |
    +-- Validate
    +-- Persist result
    +-- Mark closed
    +-- Create audit
```

Todo debe quedar consistente.

---

# 105. Feature Flags

Permitir activar/desactivar funcionalidades progresivamente.

Ejemplo:

```text
PayrollV2
LreIntegration
AdvancedAccounting
EmployeePortal
```

No abusar de feature flags permanentes.

---

# 106. Configuración por organización

Cada organización debe poder configurar:

```text
Timezone
Currency
Payroll settings
Work week
Departments
Cost centers
Approval flows
Document settings
```

Las configuraciones sensibles deben tener auditoría.

---

# 107. Workflow

Muchos procesos tendrán aprobación:

```text
Overtime
Leave
Purchase
Payroll
Expense
```

Crear un mecanismo reutilizable de workflow.

Conceptualmente:

```text
Workflow
WorkflowStep
WorkflowInstance
WorkflowApproval
```

---

# 108. Notificaciones

Canales iniciales:

```text
In-App
Email
```

Futuro:

```text
Push
WhatsApp
```

No acoplar los módulos directamente al proveedor.

---

# 109. Email

Crear:

```text
IEmailSender
```

Permitir cambiar proveedor.

---

# 110. Archivos

Crear:

```text
IFileStorage
```

Implementaciones:

```text
LocalFileStorage
AzureBlobStorage
S3Storage
```

Desarrollo puede usar filesystem local.

Producción debe utilizar object storage.

---

# 111. Ambiente local

Objetivo:

```bash
git clone ...
docker compose up
```

y poder comenzar a desarrollar.

---

# 112. Configuración inicial del proyecto

Orden recomendado:

```text
1. Crear Git repository
2. Crear solution .NET
3. Crear Angular
4. Crear Docker Compose
5. PostgreSQL
6. Identity
7. Tenant
8. EF Core
9. Migrations
10. CI
```

---

# 113. Primer Sprint

Objetivo:

Tener una aplicación vacía pero arquitectónicamente funcional.

```text
[ ] Repository
[ ] .NET 10
[ ] Angular 22
[ ] PostgreSQL 18
[ ] Docker
[ ] Health checks
[ ] Swagger/OpenAPI
[ ] Authentication skeleton
[ ] Tenant skeleton
[ ] CI
```

---

# 114. Segundo Sprint

```text
[ ] Users
[ ] Roles
[ ] Permissions
[ ] Organization
[ ] Departments
[ ] Positions
[ ] Audit log
```

---

# 115. Tercer Sprint

```text
[ ] Person
[ ] Employee
[ ] Employee documents
[ ] Bank accounts
[ ] Contacts
```

---

# 116. Cuarto Sprint

```text
[ ] Contracts
[ ] Contract versions
[ ] Salary configuration
[ ] Work schedules
```

---

# 117. Quinto Sprint

```text
[ ] Attendance
[ ] Shifts
[ ] Overtime
[ ] Approvals
```

---

# 118. Sexto Sprint

```text
[ ] Payroll period
[ ] Payroll items
[ ] Calculation engine foundation
[ ] Rules
```

---

# 119. Séptimo Sprint

```text
[ ] Salary calculation
[ ] Discounts
[ ] Taxable items
[ ] Gratification
[ ] Payslip
```

---

# 120. Octavo Sprint

```text
[ ] Payroll approval
[ ] Payroll closing
[ ] Audit
[ ] Regression tests
[ ] Reports
```

---

# 121. MVP Definition

El MVP se considera exitoso cuando una organización puede:

```text
[ ] Crear organización
[ ] Crear usuario
[ ] Crear empleado
[ ] Crear contrato
[ ] Configurar jornada
[ ] Registrar asistencia
[ ] Registrar horas extras
[ ] Gestionar vacaciones
[ ] Crear período de remuneraciones
[ ] Calcular remuneraciones
[ ] Configurar gratificación
[ ] Generar liquidaciones
[ ] Cerrar período
[ ] Auditar cambios
```

---

# 122. Criterio especial de remuneraciones

Antes de producción:

```text
[ ] Casos normales
[ ] Casos sin gratificación
[ ] Casos con gratificación
[ ] Casos de gratificación convencional
[ ] Horas extras
[ ] Ausencias
[ ] Ingresos a mitad de mes
[ ] Términos de contrato
[ ] Descuentos
[ ] Redondeos
[ ] Topes
[ ] Reliquidaciones
```

deben estar cubiertos por tests.

---

# 123. Producción

Antes de producción:

```text
[ ] HTTPS
[ ] Backups
[ ] Monitoring
[ ] Logging
[ ] Error tracking
[ ] Database migration strategy
[ ] Disaster recovery
[ ] Security review
[ ] RBAC
[ ] Audit
[ ] Restore test
```

---

# 124. Estrategia Cloud

## MVP

Infraestructura simple:

```text
Cloud VM / Container Platform
        |
        +-- API
        +-- Angular
        +-- PostgreSQL
```

## Crecimiento

Migrar a:

```text
Managed PostgreSQL
Container Platform
Object Storage
Secret Manager
Monitoring
WAF
```

No introducir Kubernetes hasta que exista una razón real.

---

# 125. Escalamiento futuro

Si el producto crece:

```text
                     Load Balancer
                           |
               +-----------+-----------+
               |                       |
               v                       v
             API 1                   API 2
               |                       |
               +-----------+-----------+
                           |
                     PostgreSQL
                           |
                    Read replicas
```

Y eventualmente:

```text
Payroll Service
Accounting Service
Notification Service
Document Service
```

solo si la escala lo justifica.

---

# 126. Regla para microservicios

Un módulo solo se convierte en microservicio si existe al menos una razón concreta:

* escala independiente
* equipo independiente
* ciclo de despliegue independiente
* aislamiento requerido
* dependencia tecnológica específica
* disponibilidad independiente

No convertir módulos en microservicios simplemente porque "es arquitectura moderna".

---

# 127. Decisiones arquitectónicas iniciales

## ADR-001

**Decisión:** Modular Monolith.

**Motivo:** reducir complejidad inicial y mantener límites claros.

---

## ADR-002

**Decisión:** PostgreSQL.

**Motivo:** open source, robusto, excelente soporte transaccional y adecuado para SaaS.

---

## ADR-003

**Decisión:** .NET 10.

**Motivo:** LTS, ecosistema maduro y excelente integración con sistemas empresariales.

---

## ADR-004

**Decisión:** Angular.

**Motivo:** aplicación empresarial con formularios, tablas, workflows y administración compleja.

---

## ADR-005

**Decisión:** Docker.

**Motivo:** reproducibilidad de ambientes.

---

## ADR-006

**Decisión:** Multi-tenant desde el comienzo.

**Motivo:** evitar una migración arquitectónica posterior.

---

## ADR-007

**Decisión:** Motor de reglas de remuneraciones.

**Motivo:** normativa cambiante y necesidad de reproducibilidad histórica.

---

# 128. Reglas de desarrollo

1. No introducir complejidad sin necesidad.
2. No duplicar lógica.
3. No colocar reglas laborales en controllers.
4. No colocar lógica laboral en Angular.
5. No almacenar secretos en Git.
6. No usar `double` para dinero.
7. No modificar históricos silenciosamente.
8. Toda funcionalidad crítica debe tener tests.
9. Toda modificación de base debe tener migration.
10. Toda operación sensible debe auditarse.

---

# 129. Orden recomendado de construcción

```text
FOUNDATION
    |
    v
IDENTITY
    |
    v
TENANT / ORGANIZATION
    |
    v
PEOPLE
    |
    v
CONTRACTS
    |
    v
ATTENDANCE
    |
    v
LEAVE
    |
    v
PAYROLL ENGINE
    |
    v
GRATIFICATION
    |
    v
PAYSLIPS
    |
    v
LRE / INTEGRATIONS
    |
    v
PORTAL
    |
    v
ACCOUNTING
    |
    v
PURCHASING
    |
    v
SALES
    |
    v
INVENTORY
    |
    v
FULL ERP
```

---

# 130. Regla de oro del proyecto

El sistema no debe intentar ser una copia de Defontana.

Debe convertirse en una plataforma mejor diseñada para nuestro caso de uso.

Especialmente:

```text
Defontana + Workera
        |
        v
    Aprendizajes
        |
        v
Nuevo ERP
```

No:

```text
Copiar pantalla por pantalla
```

---

# 131. Objetivo final

La arquitectura final deberá permitir:

```text
                    ERP PLATFORM
                         |
        +----------------+----------------+
        |                |                |
        v                v                v
       RRHH          REMUNERACIONES     ASISTENCIA
        |                |                |
        +----------------+----------------+
                         |
                         v
                    FINANZAS
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
          COMPRAS      VENTAS     INVENTARIO
             |           |           |
             +-----------+-----------+
                         |
                         v
                    CONTABILIDAD
                         |
                         v
                     REPORTES
```

La plataforma deberá poder crecer desde una única organización hasta múltiples organizaciones sin cambiar su modelo fundamental.

---

# 132. Próximo paso técnico

Antes de implementar funcionalidades de negocio, realizar:

```text
1. Crear solución .NET 10
2. Crear Angular 22
3. Crear Docker Compose
4. Configurar PostgreSQL 18
5. Crear estructura de módulos
6. Crear Tenant
7. Crear Identity
8. Crear DbContext
9. Crear primera migration
10. Crear Health Checks
11. Crear OpenAPI
12. Crear CI
13. Crear arquitectura de tests
14. Crear documentación ADR
```

Después:

```text
People
    |
Employee
    |
Contract
    |
Attendance
    |
Payroll
```

---

# 133. Estado actual del proyecto

```text
FASE: Diseño inicial

Arquitectura: DEFINIDA
Backend: DEFINIDO
Frontend: DEFINIDO
Base de datos: DEFINIDA
Docker: DEFINIDO
Cloud strategy: DEFINIDA
Multi-tenancy: DEFINIDO
Payroll engine: DEFINIDO CONCEPTUALMENTE
Gratificación: DEFINIDA CONCEPTUALMENTE

Implementación:
PENDIENTE
```

---

# 134. Próximo documento recomendado

Antes de comenzar a programar Payroll, crear:

```text
docs/architecture/01-domain-model.md
```

Este documento debe definir en detalle:

```text
Tenant
Organization
Person
Employee
Contract
Department
Position
CostCenter
Attendance
Shift
Leave
PayrollPeriod
PayrollRun
PayrollItem
PayrollRule
PayrollRuleVersion
Payslip
Document
AuditLog
```

y sus relaciones.

Después crear:

```text
docs/payroll/payroll-engine.md
```

para especificar el motor de cálculo.

Finalmente:

```text
docs/payroll/gratification.md
```

para documentar exactamente:

* no aplica
* gratificación legal
* gratificación convencional
* configuración
* fórmulas
* vigencia
* redondeos
* casos de prueba
* auditoría

---

# 135. Regla final

**No comenzar a desarrollar el motor de remuneraciones hasta tener aprobado el modelo de dominio y los casos de negocio.**

El orden correcto es:

```text
REQUISITOS
    ↓
MODELO DE DOMINIO
    ↓
ARQUITECTURA
    ↓
BASE DE DATOS
    ↓
API
    ↓
FRONTEND
    ↓
TESTS
    ↓
IMPLEMENTACIÓN
```

No comenzar por las pantallas.

El núcleo del producto será el dominio laboral y financiero. Las pantallas deben construirse alrededor de ese dominio.

# 136. Integraciones y cumplimiento Chile

## 136.1 Objetivo

El ERP debe diseñarse desde el inicio considerando las integraciones necesarias para operar en Chile.

Las integraciones deberán implementarse como módulos independientes y desacoplados del dominio principal.

Arquitectura conceptual:

ERP
- Payroll
- Attendance
- Accounting
- Integrations
  - Dirección del Trabajo
  - Libro de Remuneraciones Electrónico (LRE)
  - Previred
  - SII
  - Firma Electrónica
  - Bancos
  - Email

Las integraciones externas nunca deberán contener lógica principal de negocio.

---

# 137. Principio de integración

Cada integración deberá utilizar una arquitectura basada en contratos y adaptadores.

Flujo:

Domain Contract
-> Application Service
-> Integration Adapter
-> External System

Ejemplo:

Payroll
-> IPreviredService
-> PreviredAdapter
-> Previred

El módulo Payroll no debe conocer:

- URLs externas
- tokens
- formatos HTTP
- credenciales
- detalles de autenticación
- implementación específica del proveedor

---

# 138. Estados de una integración

Cada integración deberá tener un estado:

DISCOVERY
ACCESS_REQUESTED
DOCUMENTATION_AVAILABLE
SANDBOX
CERTIFICATION
PRODUCTION_READY
PRODUCTION
SUSPENDED

Esto permitirá controlar el avance de cada integración.

---

# 139. Ambiente Sandbox

Siempre que el organismo o proveedor disponga de un ambiente de pruebas, se deberá utilizar antes de producción.

Flujo:

Development
-> Mock
-> Sandbox
-> Certification
-> Production

Nunca desarrollar directamente contra producción.

---

# 140. Credenciales

Las credenciales de integraciones externas nunca deben almacenarse en:

- Git
- appsettings.json
- Código fuente
- Dockerfile
- Base de datos sin protección

Utilizar, según ambiente:

- Environment Variables
- Secret Manager
- Cloud Secret Store

---

# 141. Registro de integraciones

Crear una entidad conceptual:

ExternalIntegration

Campos:

- Id
- TenantId
- Provider
- Type
- Environment
- Status
- Configuration
- CreatedAt
- UpdatedAt

Las credenciales deberán almacenarse separadamente y de forma segura.

---

# 142. Dirección del Trabajo

El ERP deberá considerar las obligaciones relacionadas con:

- registro y control de asistencia
- horas trabajadas
- horas extraordinarias
- descansos
- sistemas electrónicos de asistencia
- Libro de Remuneraciones Electrónico
- documentación laboral electrónica

La implementación deberá basarse en la normativa vigente de la Dirección del Trabajo y actualizarse cuando cambien las resoluciones aplicables.

---

# 143. Sistema de control de asistencia

El ERP deberá soportar múltiples mecanismos de marcación.

No diseñar el sistema suponiendo que existe un único dispositivo.

Mecanismos previstos:

1. Código o número de empleado
2. PIN
3. Tarjeta
4. RFID
5. NFC
6. Código QR
7. Aplicación móvil
8. Terminal dedicado
9. Biometría

La biometría será considerada una funcionalidad posterior.

---

# 144. Recomendación para el MVP

El mecanismo inicial recomendado será:

- Número o código de empleado
- PIN
- QR

Para instalaciones físicas se podrá utilizar un terminal basado en tablet o navegador.

Flujo:

Empleado
-> Terminal / Tablet
-> Identificación
-> Registro de marcación

Alternativamente:

Terminal
-> Número de empleado
-> PIN
-> Marcación

Esto reduce el costo inicial de hardware.

---

# 145. Tarjetas

Debe existir soporte para tarjetas físicas.

Modelo conceptual:

Employee
-> EmployeeCredential

EmployeeCredential:

- EmployeeId
- Type
- Identifier
- Status
- IssuedAt
- RevokedAt

Tipos posibles:

- CARD
- RFID
- NFC

No almacenar información innecesaria de la tarjeta.

---

# 146. QR

El sistema deberá soportar QR como mecanismo de marcación.

Existen dos modelos.

Modelo A: QR individual.

Cada trabajador posee un identificador QR.

Employee
-> Unique QR
-> Terminal
-> Attendance Record

Modelo B: QR dinámico.

Terminal
-> Temporary QR
-> Employee Mobile
-> Attendance API

El modelo dinámico deberá evaluarse posteriormente desde el punto de vista de seguridad.

---

# 147. Seguridad de QR

No utilizar el EmployeeId como único contenido de autenticación del QR.

El QR deberá utilizar un identificador no predecible y, cuando corresponda, tokens temporales.

Información conceptual:

- CredentialId
- Token
- Expiration
- Signature

---

# 148. PIN y número de empleado

Para terminales físicos se podrá implementar:

Número de empleado + PIN

Ejemplo:

Empleado: 10452
PIN: ****

El PIN nunca debe almacenarse en texto plano.

Debe almacenarse utilizando hashing seguro.

---

# 149. Biometría

La biometría se considera una funcionalidad de una etapa posterior.

Posibles mecanismos:

- Huella digital
- Reconocimiento facial
- Otros dispositivos biométricos

La arquitectura deberá permitir incorporar biometría sin modificar el núcleo de Attendance.

Interfaz conceptual:

IAttendanceCaptureProvider

Implementaciones:

- NumberProvider
- CardProvider
- QRProvider
- MobileProvider
- BiometricProvider

La incorporación de biometría deberá considerar previamente:

- normativa vigente
- privacidad
- protección de datos personales
- requisitos de la Dirección del Trabajo
- seguridad de la información
- proveedor del dispositivo
- proceso de certificación o autorización que corresponda

---

# 150. Modelo de marcación

Todas las formas de identificación deben terminar en la misma entidad:

AttendancePunch

Campos:

- Id
- TenantId
- EmployeeId
- Timestamp
- PunchType
- CaptureMethod
- DeviceId
- LocationId
- CredentialId
- Source
- Metadata

Ejemplo conceptual:

EmployeeId = 125
PunchType = IN
CaptureMethod = QR
DeviceId = TERMINAL-001
Timestamp = fecha/hora

---

# 151. Tipos de marcación

Tipos iniciales:

- IN
- OUT
- BREAK_START
- BREAK_END
- OTHER

El sistema deberá permitir configurar qué tipos aplican según la jornada.

---

# 152. Dispositivo

Crear la entidad:

AttendanceDevice

Campos:

- Id
- TenantId
- Name
- Type
- LocationId
- Status
- LastSeenAt
- Configuration

Ejemplo:

TERMINAL-001
Recepción
Sede Principal
ACTIVE

---

# 153. Modo Offline

Los terminales de asistencia deberían soportar eventualmente operación offline.

Cuando existe conexión:

Terminal
-> API

Cuando no existe conexión:

Terminal
-> Local Queue
-> Sincronización
-> API

Esto es importante para:

- mala conectividad
- instalaciones rurales
- sucursales
- centros de trabajo remotos

---

# 154. Sincronización de asistencia

Las marcaciones deberán tener un identificador único.

Ejemplo:

PunchId = UUID

Si una marcación se envía dos veces, el servidor debe reconocer que ya existe.

Esto garantiza idempotencia y evita duplicados.

---

# 155. Integridad de marcaciones

Una marcación registrada no deberá eliminarse físicamente de manera normal.

Si debe corregirse:

Original Punch
-> Correction
-> Audit

Debe quedar trazabilidad completa.

---

# 156. Geolocalización

Para dispositivos móviles podrá considerarse:

- Latitude
- Longitude
- Accuracy

No debe ser requisito para todos los métodos de marcación.

La utilización de ubicación deberá analizarse de acuerdo con:

- finalidad
- proporcionalidad
- privacidad
- seguridad
- configuración de la organización
- normativa vigente

---

# 157. Asistencia móvil

El ERP deberá soportar eventualmente:

- Android
- iOS
- PWA

La aplicación móvil podrá permitir:

- Entrada
- Salida
- Inicio de descanso
- Fin de descanso

Y eventualmente:

- Ubicación
- Fotografía
- QR

según configuración y cumplimiento aplicable.

---

# 158. Libro de Remuneraciones Electrónico

Crear el módulo:

Modules/Integrations/Lre

Responsabilidades:

Payroll
-> LreDataBuilder
-> LreValidator
-> LreExporter
-> LreSubmission

Separar:

Cálculo de remuneraciones

de:

Formato y envío LRE

---

# 159. Validación LRE

Antes de generar el archivo o realizar el envío:

- Trabajador válido
- RUT válido
- Período válido
- Contrato válido
- Haberes válidos
- Descuentos válidos
- Totales consistentes
- Datos previsionales completos
- Reglas de formato cumplidas

El usuario debe poder visualizar los errores antes de realizar el envío.

---

# 160. Previred

Crear:

Modules/Integrations/Previred

Flujo:

Payroll
-> PreviredExport
-> Validation
-> Sandbox / Certification
-> Production

La implementación debe comenzar con investigación de:

- mecanismos oficiales disponibles
- formatos
- autenticación
- ambiente de pruebas
- certificación
- requisitos técnicos
- restricciones de seguridad

---

# 161. SII

Crear:

Modules/Integrations/Sii

Separar las funcionalidades:

- Authentication
- Documents
- Electronic Tax Documents
- Queries
- Status

No mezclar lógica tributaria con Payroll.

---

# 162. Dirección del Trabajo

Crear:

Modules/Integrations/Dt

Separar:

- Attendance
- LRE
- Labor Documents
- Other DT Services

Cada integración deberá documentarse por separado.

---

# 163. Firma electrónica

Crear:

Modules/Integrations/ElectronicSignature

Interfaz:

ISignatureProvider

Operaciones:

- CreateRequest
- SendDocument
- GetStatus
- DownloadSignedDocument
- CancelRequest

El ERP no debe quedar atado a un proveedor específico.

---

# 164. Bancos

Etapa posterior.

Objetivos:

- generación de archivos de pago
- conciliación
- estado de pagos
- importación bancaria

Arquitectura:

Accounting
-> Banking Integration
-> Bank Adapter

Debe ser posible soportar múltiples bancos sin modificar Accounting.

---

# 165. Email

El email será una integración transversal.

Casos de uso:

- liquidación disponible
- documento firmado
- solicitud aprobada
- solicitud rechazada
- cambio de contraseña
- notificación administrativa

Interfaz:

IEmailSender

---

# 166. Webhooks

Las integraciones que lo permitan podrán utilizar webhooks.

Flujo:

External Provider
-> Webhook
-> Integration API
-> Application Event
-> ERP

Los webhooks deberán:

- autenticarse
- validarse
- ser idempotentes
- registrarse
- auditarse
- manejar reintentos

---

# 167. Reintentos

Las integraciones externas pueden fallar.

Utilizar:

- Retry
- Backoff
- Timeout
- Circuit Breaker
- Dead Letter / Failed Integration

No realizar reintentos infinitos.

---

# 168. Registro de operaciones externas

Crear:

IntegrationLog

Campos:

- Id
- TenantId
- Integration
- Operation
- RequestId
- ExternalId
- Status
- StartedAt
- CompletedAt
- ErrorCode
- ErrorMessage

No almacenar secretos ni información sensible innecesaria.

---

# 169. Dashboard de integraciones

Administración deberá poder visualizar:

Integración | Ambiente | Estado

LRE | Sandbox | OK
Previred | Sandbox | Pendiente
SII | Sandbox | OK
Firma electrónica | Producción | OK
DT | Certificación | Pendiente

También:

- última ejecución
- último error
- cantidad de errores
- última sincronización

---

# 170. Solicitud de accesos Sandbox

Durante la fase inicial del proyecto se deberán identificar y solicitar los accesos necesarios para:

- Dirección del Trabajo
- LRE
- Previred
- SII
- Firma electrónica
- Bancos

Para cada proveedor u organismo documentar:

- Nombre
- URL documentación
- URL sandbox
- Contacto
- Proceso de solicitud
- Credenciales requeridas
- Certificados
- IP whitelist
- OAuth / API Key / Certificado
- Formato
- Proceso de certificación
- Limitaciones

---

# 171. Matriz de integraciones

Mantener una matriz actualizada.

Integración: LRE
MVP: Sí
Sandbox: Por validar
Certificación: Según mecanismo oficial
Producción: Sí
Prioridad: Alta

Integración: Previred
MVP: Sí
Sandbox: Por validar
Certificación: Por validar
Producción: Sí
Prioridad: Alta

Integración: DT
MVP: Sí
Sandbox: Por validar
Certificación: Por validar
Producción: Sí
Prioridad: Alta

Integración: SII
MVP: Posterior
Sandbox: Por validar
Certificación: Por validar
Producción: Sí
Prioridad: Media

Integración: Firma electrónica
MVP: Posterior
Sandbox: Según proveedor
Certificación: Según proveedor
Producción: Sí
Prioridad: Media

Integración: Bancos
MVP: Posterior
Sandbox: Según banco
Certificación: Según banco
Producción: Sí
Prioridad: Media

Integración: Biometría
MVP: Posterior
Sandbox: Según proveedor
Certificación: Según proveedor
Producción: Posterior
Prioridad: Baja

La matriz deberá actualizarse durante el proyecto.

---

# 172. Cumplimiento del sistema de asistencia

El módulo de asistencia debe desarrollarse considerando desde el inicio los requisitos técnicos y normativos aplicables a los sistemas electrónicos de registro y control de asistencia.

Antes de comercializar el sistema como solución de control de asistencia laboral, se deberá revisar el procedimiento vigente de la Dirección del Trabajo para la autorización, certificación o reconocimiento que corresponda.

La arquitectura debe permitir someter el sistema a dicho proceso sin modificaciones estructurales.

---

# 173. Arquitectura de Attendance

Attendance

- Schedules
- Punches
- Devices
- Rules
- Capture Methods

Capture Methods:

- PIN
- CARD
- QR
- MOBILE
- BIOMETRIC

Todos los mecanismos deben terminar en el mismo núcleo Attendance.

---

# 174. Separación entre captura y cálculo

La captura solamente registra el evento.

No debe calcular inmediatamente:

- horas trabajadas
- horas extras
- atrasos
- descuentos
- remuneraciones

Flujo:

Punch
-> Attendance Processing
-> Worked Time
-> Overtime / Absence
-> Payroll

Esto permite corregir asistencia sin alterar directamente una marcación original.

---

# 175. Integración Attendance hacia Payroll

Payroll debe consumir información procesada.

Flujo:

Attendance
-> Monthly Attendance Summary
-> Payroll

Datos posibles:

- RegularHours
- OvertimeHours
- AbsenceHours
- LateMinutes
- UnpaidLeaveDays

Payroll utiliza estos resultados para calcular remuneraciones.

---

# 176. Regla de diseño

El dispositivo de asistencia no es el sistema de RRHH.

El dispositivo solamente captura:

- quién
- cuándo
- dónde o dispositivo
- cómo

El ERP determina posteriormente:

- qué jornada correspondía
- qué horas fueron trabajadas
- qué horas son extraordinarias
- qué ausencia corresponde
- qué impacto tiene en remuneraciones

---

# 177. Etapas de implementación de asistencia

Etapa 1 - MVP:

- Número de empleado
- PIN
- QR
- Terminal web/tablet
- Entrada
- Salida
- Correcciones
- Auditoría

Etapa 2:

- Tarjeta
- RFID
- NFC
- Terminal dedicado
- Offline
- Sincronización

Etapa 3:

- Aplicación móvil
- Geolocalización
- Modo offline móvil
- QR dinámico

Etapa 4:

- Biometría
- Integración con terminales biométricos
- Integraciones con proveedores especializados

---

# 178. Objetivo final

El ERP deberá poder funcionar con diferentes modalidades de captura sin cambiar el módulo de remuneraciones.

Modelo:

Attendance
-> PIN
-> CARD
-> QR
-> MOBILE
-> BIOMETRIC
-> Attendance Engine
-> Payroll Engine

La tecnología de identificación será intercambiable.

La información laboral y de remuneraciones permanecerá independiente del dispositivo utilizado.

---

# 179. Principio de futuro

No construir un ERP dependiente de un fabricante específico de relojes control.

El objetivo es soportar:

- API
- Web
- Mobile
- Terminal
- QR
- Card
- Biometric

Todos deben poder alimentar el mismo núcleo de asistencia.

---

# 180. Investigación de integraciones antes del desarrollo

Antes de implementar una integración real, se deberá realizar una ficha técnica.

Ficha:

Integration:
Provider / Organism:
Official Documentation:
Sandbox:
Certification:
Authentication:
Transport:
Format:
Rate Limits:
Required Credentials:
Required Certificates:
IP Restrictions:
Webhooks:
Error Handling:
Retry Policy:
Production Requirements:
Responsible:
Status:

No asumir que un organismo dispone de una API REST simplemente porque exista un servicio electrónico.

Debe verificarse siempre el mecanismo oficial disponible.

---

# 181. Estrategia de desarrollo de integraciones

Cada integración deberá seguir:

1. Investigación
2. Documentación
3. Solicitud de acceso
4. Mock
5. Sandbox
6. Tests
7. Certificación
8. Producción
9. Monitoring

---

# 182. Integraciones como módulos reemplazables

Una integración nunca debe contaminar el dominio.

Ejemplo:

Payroll
-> IPensionContributionProvider
-> PreviredAdapter

En el futuro:

Payroll
-> IPensionContributionProvider
-> FutureProviderAdapter

Esto permite cambiar proveedores sin reescribir Payroll.

---

# 183. Objetivo final de integraciones

El ERP deberá ser capaz de operar como plataforma central:

ERP
- RRHH
- Payroll
- Attendance
- Accounting
- Integrations

Integrations:

- Dirección del Trabajo
- Previred
- SII
- Firma electrónica
- Bancos

Las integraciones deberán ser:

- reemplazables
- auditables
- testeables
- configurables por ambiente
- seguras
- monitoreables

---

# 184. Criterio de aceptación de una integración

Una integración se considerará lista cuando:

- documentación técnica creada
- credenciales obtenidas
- ambiente sandbox configurado
- adapter implementado
- validaciones implementadas
- manejo de errores
- reintentos
- idempotencia
- auditoría
- logs
- tests unitarios
- tests de integración
- pruebas sandbox exitosas
- certificación completada cuando corresponda
- configuración de producción
- monitoring
- runbook operativo

---

# 185. Regla final de integraciones

Las integraciones no serán desarrolladas como funcionalidades aisladas.

Formarán parte de la arquitectura del ERP desde el inicio, pero se implementarán progresivamente.

Orden recomendado:

Attendance
-> Payroll
-> LRE
-> Previred
-> DT
-> Accounting
-> SII
-> Banks
-> Documents
-> Electronic Signature

La investigación y solicitud de accesos a sandbox deberá comenzar durante la fase de arquitectura, aunque la implementación productiva de cada integración ocurra posteriormente.
