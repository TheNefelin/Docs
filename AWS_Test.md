## Rol del Arquitecto

1. Los 4 valores del Manifiesto Ágil
    - Individuos e interacciones sobre procesos y herramientas
    - Software funcionando sobre documentación extensiva
    - Colaboración con el cliente sobre negociación contractual
    - Respuesta ante el cambio sobre seguir un plan

2. Roles principales de un equipo Ágil/Scrum
    - Product Owner (Dueño del Producto)
    - Scrum Master (Facilitador)
    - Development Team (Equipo de Desarrollo)

3. Reuniones de Scrum (Ceremonias)
    - Daily Stand-up (Reunión diaria)
    - Sprint Planning (Planificación del Sprint)
    - Sprint Review (Revisión del trabajo completado) ✅
    - Sprint Retrospective (Retrospectiva)

4. Escenario de calidad en diseño arquitectónico
    - Una descripción específica de un requisito de calidad
    (Los requisitos de calidad no funcionales como escalabilidad, disponibilidad, seguridad son fundamentales en el diseño arquitectónico)

5. Reunión que revisa trabajo y planifica próximo ciclo
    - Sprint Review - Ocurre al final del sprint, donde se muestra el trabajo completado y se planifica el próximo ciclo basado en el feedback.

6. Pilar de Eficiencia de Rendimiento en AWS Contribuye mediante:
    - Selección óptima de tipos de instancias EC2 y servicios
    - Escalado automático según demanda
    - Alto rendimiento con baja latencia
    - Monitoreo continuo con CloudWatch

7. Diagrama de Secuencias - Muestra la interacción entre objetos en el tiempo y el flujo de mensajes.
    - Diagrama UML para visión dinámica

8. Arquitectura de software según la industria
Es la estructura fundamental de un sistema que define:
    - Componentes y sus relaciones
    - Principios y decisiones de diseño
    - Comportamiento visible externamente
    - Atributos de calidad (escalabilidad, mantenibilidad)

9. Desventajas principales de SaaS vs On-Premise
Dependencia del proveedor (vendor lock-in)
    - Limitaciones de personalización
    - Conectividad constante a internet requerida
    - Menor control sobre actualizaciones y cambios
    - Preocupaciones de seguridad y privacidad de datos

# Fundamentos de Arquitectura - Respuestas

## 1. Características que distinguen al modelo PaaS
- **Entorno de desarrollo completo** en la nube
- **Sin gestión de infraestructura** subyacente
- **Enfoque en el código** de aplicación, no en el entorno
- **Despliegue automatizado** y escalado integrado

## 2. Herramienta para gestionar picos de demanda eficientemente
- **Auto Scaling Groups** (para EC2)
- **AWS Lambda** (para cómputo serverless)
- **Amazon EC2 Spot Instances** (para cargas flexibles)

## 3. Característica clave de nube pública
- **Recursos compartidos gestionados por un proveedor**

## 4. Desventaja común de modelo híbrido
- **Complejidad de gestión** de dos entornos diferentes

## 5. Componente principal arquitectura cliente-servidor
- **Cliente que solicita servicios al servidor**

## 6. Función principal capa de lógica en 3 capas
- **Procesar la lógica de negocio** y reglas de la aplicación

## 7. Cómo serverless mejora la elasticidad
- **Escalado automático instantáneo** desde cero a miles de ejecuciones
- **Sin capacidad ociosa**: Cero costo cuando no hay tráfico

## 8. Cómo el aislamiento mejora la resiliencia
- **Contiene fallos** para evitar propagación en el sistema
- **Arquitecturas de microservicios** que fallan independientemente
- **Zonas de disponibilidad** separadas físicamente

