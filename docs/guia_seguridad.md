# Guía de Seguridad en AWS

## 1. Modelo de Responsabilidad Compartida
La seguridad en AWS es una tarea conjunta entre Amazon Web Services y el cliente:
* **Seguridad de la nube (AWS):** AWS protege la infraestructura global que ejecuta todos los servicios (hardware, software, redes y instalaciones de los centros de datos).
* **Seguridad en la nube (Cliente):** Eres responsable de la seguridad de tus datos, sistemas operativos, configuraciones de red, control de acceso e identidad (IAM) y cifrado.

## 2. Gestión de Identidad y Accesos (IAM)
* **Evita el usuario Root:** No uses la cuenta raíz para tareas cotidianas; protégela con autenticación multifactor (MFA).
* **Principio de menor privilegio:** Concede a los usuarios y aplicaciones solo los permisos mínimos necesarios para realizar sus tareas.
* **Usa roles y grupos:** Prefiere asignar permisos a través de grupos o roles de IAM en lugar de crear credenciales permanentes para cada usuario.

## 3. Protección de Datos
* **Cifrado en reposo:** Utiliza AWS KMS (Key Management Service) para cifrar bases de datos (como Amazon RDS) y almacenamiento (como buckets de Amazon S3).
* **Cifrado en tránsito:** Exige el uso de HTTPS/TLS para todas las comunicaciones hacia y desde tus servicios.
* **Bloqueo de acceso público:** Asegúrate de que los buckets de S3 no expongan información confidencial a internet.

## 4. Detección de Amenazas y Monitoreo
* **Amazon GuardDuty:** Habilita este servicio para detectar comportamientos maliciosos o no autorizados de manera inteligente mediante aprendizaje automático.
* **AWS CloudTrail:** Registra toda la actividad de la API en tu cuenta para auditorías de seguridad.
* **AWS Config:** Evalúa y audita de forma continua las configuraciones de tus recursos.

## 5. Seguridad de Red
* **Grupos de seguridad y NACLs:** Restringe el tráfico entrante y saliente a nivel de instancia y de subred.
* **AWS WAF:** Protege tus aplicaciones web contra ataques comunes como la inyección SQL o el *cross-site scripting* (XSS).