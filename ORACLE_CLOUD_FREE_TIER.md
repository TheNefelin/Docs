# Oracle Cloud Free Tier - Guía de uso seguro

## Objetivo

Crear una infraestructura gratuita para proyectos personales:

* API ASP.NET Core (.NET)
* PostgreSQL
* Docker
* Nginx
* HTTPS

Usando únicamente recursos incluidos en Oracle Cloud Always Free.

---

# 1. Cuenta Oracle Cloud

## Registro

Requisitos:

* Cuenta Oracle Cloud.
* Tarjeta para validación.
* Correo electrónico.
* Teléfono.

Notas:

* La tarjeta se usa para verificar identidad.
* No significa que se realicen cobros si se usan únicamente recursos Always Free.
* Usar una tarjeta con alertas activadas.

---

# 2. Reglas para evitar cobros

## Antes de crear recursos

Siempre verificar:

* Que diga "Always Free Eligible".
* Que la región seleccionada tenga capacidad disponible.
* Que el recurso no sea una versión de pago.

Evitar crear:

* Instancias fuera de Always Free.
* Bases de datos gestionadas de pago.
* Load Balancers de pago.
* Servicios de Kubernetes gestionados si no son necesarios.
* Backups adicionales no incluidos.

---

# 3. Recursos recomendados

Arquitectura:

```
Oracle Cloud VM

Ubuntu ARM

├── Docker
│
├── API ASP.NET Core
│
├── PostgreSQL
│
└── Nginx + HTTPS
```

---

# 4. Máquina virtual recomendada

Usar:

* Compute Instance ARM Always Free.
* Ubuntu.
* Shape Always Free.

Ejemplo:

```
VM.Standard.A1.Flex
```

Configuración recomendada:

* CPU: según disponibilidad Always Free.
* RAM: según disponibilidad Always Free.

No superar los límites gratuitos de Oracle.

---

# 5. Instalación inicial

Actualizar sistema:

```bash
sudo apt update
sudo apt upgrade -y
```

Instalar Docker:

```bash
sudo apt install docker.io docker-compose-plugin
```

Comprobar:

```bash
docker --version
```

---

# 6. Base de datos

Preferencia:

PostgreSQL en Docker.

Ejemplo:

```
docker compose

api
 |
postgres
```

Ventajas:

* Fácil backup.
* Fácil migración.
* Misma configuración en desarrollo y producción.

---

# 7. Seguridad básica

Cambiar puerto SSH:

* No dejar configuraciones por defecto.

Usar:

* Llaves SSH.
* Firewall.
* Fail2ban.

Abrir solamente:

```
22  SSH
80  HTTP
443 HTTPS
```

Cerrar:

```
5432 PostgreSQL público
```

La base de datos debe estar accesible solo internamente.

---

# 8. Copias de seguridad

Nunca confiar solo en la nube.

Crear backups:

PostgreSQL:

```bash
pg_dump database > backup.sql
```

Guardar:

* En otro almacenamiento.
* En tu equipo local.
* En otro proveedor si es importante.

---

# 9. Control de costes

Configurar:

* Budget mensual.
* Alertas de consumo.

Revisar periódicamente:

* Compute.
* Storage.
* Networking.
* IPs públicas.

---

# 10. Errores comunes

## Error: crear una VM normal

Problema:

Puede generar coste.

Solución:

Confirmar:

```
Always Free Eligible
```

---

## Error: dejar recursos abandonados

Ejemplos:

* Discos.
* IPs reservadas.
* Snapshots.

Aunque apagues una VM, algunos recursos pueden seguir generando costes.

---

# 11. Plan de crecimiento

Inicio:

```
1 VM Oracle Free
 |
 ├── API .NET
 ├── PostgreSQL
 └── Docker
```

Cuando crezca:

```
VM API
 |
RDS/PostgreSQL gestionado
```

o migrar a:

* AWS.
* Azure.
* Google Cloud.
* VPS privado.

---

# Checklist antes de publicar

[ ] HTTPS configurado

[ ] Backup probado

[ ] PostgreSQL no expuesto públicamente

[ ] Firewall activo

[ ] Usuario SSH con llave

[ ] Recursos marcados como Always Free

[ ] Alertas de presupuesto activadas

---

## Regla de oro

Si no estás seguro de si un servicio cuesta dinero:

NO lo crees hasta comprobar que dice:

"Always Free Eligible"
