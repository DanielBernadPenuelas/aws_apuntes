# Conceptos Básicos de AWS IAM

## ¿Qué es IAM (Identity and Access Management)?

AWS Identity and Access Management es un servicio web que ayuda a controlar de forma segura el acceso a los recursos de AWS. Permite crear y gestionar usuarios, grupos y roles, y definir permisos de acceso a recursos.

## Componentes Principales

### 1. **Usuarios (Users)**
- Entidades individuales que acceden a AWS
- Cada usuario tiene credenciales únicas (Access Key y Secret Access Key)
- Pueden pertenecer a grupos

```
Usuario: daniel.bernad
├─ Acceso por consola (contraseña)
└─ Acceso programático (Access Keys)
```

### 2. **Grupos (Groups)**
- Colecciones de usuarios
- Facilitan la gestión de permisos para múltiples usuarios
- No pueden anidarse

```
Grupo: Desarrolladores
├─ daniel.bernad
├─ maria.lopez
└─ carlos.ruiz
```

### 3. **Roles (Roles)**
- Conjunto de permisos sin credenciales permanentes
- Se asumen temporalmente
- Ideales para:
  - Servicios de AWS (Lambda, EC2)
  - Usuarios temporales
  - Acceso entre cuentas

```
Role: LambdaExecutionRole
└─ Asumido por: Lambda, EC2, o usuarios específicos
```

### 4. **Políticas (Policies)**
- Documentos JSON que definen permisos
- Pueden ser:
  - **Políticas Administradas** (AWS las crea y mantiene)
  - **Políticas Personalizadas** (Tú las creas)

## Estructura de un ARN (Amazon Resource Name)

Un ARN identifica de forma única un recurso en AWS:

```
arn:partition:service:region:account-id:resource-type/resource-id
│   │         │       │      │          │
│   │         │       │      │          └─ Identificador del recurso
│   │         │       │      └─ ID de la cuenta
│   │         │       └─ Región (vacío si global)
│   │         └─ Servicio (s3, ec2, iam, etc)
│   └─ Partición (aws, aws-cn, aws-us-gov)
└─ Indicador de ARN
```

### Ejemplos de ARNs:

```
# Usuario IAM
arn:aws:iam::123456789012:user/daniel.bernad

# Role
arn:aws:iam::123456789012:role/LambdaExecutionRole

# Bucket S3
arn:aws:s3:::mi-bucket

# Objeto S3
arn:aws:s3:::mi-bucket/ruta/al/archivo.txt

# Instancia EC2
arn:aws:ec2:eu-west-1:123456789012:instance/i-0123456789abcdef0

# Función Lambda
arn:aws:lambda:eu-west-1:123456789012:function:mi-funcion
```

## Acciones (Actions)

Las acciones especifican QUÉ operaciones se permiten. El formato es: `servicio:accion`

### Ejemplos comunes:

```
# S3
s3:GetObject       - Leer un objeto
s3:PutObject       - Crear/actualizar objeto
s3:ListBucket      - Listar contenido del bucket

# EC2
ec2:RunInstances   - Iniciar instancia
ec2:TerminateInstances - Detener instancia
ec2:DescribeInstances  - Ver información de instancias

# IAM
iam:CreateUser     - Crear usuario
iam:AttachUserPolicy - Adjuntar política a usuario

# Wildcard
s3:*              - Todas las acciones de S3
*                 - Todas las acciones (¡PELIGROSO!)
```

## Condiciones (Conditions)

Permiten añadir restricciones adicionales a los permisos:

### Tipos de condiciones:

```
StringEquals              - Coincidencia exacta de cadena
StringLike                - Con caracteres comodín (* y ?)
IpAddress                 - Restricción por IP
DateGreaterThan          - Restricción por fecha/hora
Bool                      - Valores booleanos
ArnLike                   - Coincidencia de patrones ARN
```

### Ejemplos de condiciones:

```json
"Condition": {
  "StringEquals": {
    "aws:username": "daniel.bernad"
  }
}

"Condition": {
  "IpAddress": {
    "aws:SourceIp": ["192.0.2.0/24", "203.0.113.0/24"]
  }
}

"Condition": {
  "DateLessThan": {
    "aws:CurrentTime": "2024-12-31T23:59:59Z"
  }
}

"Condition": {
  "StringEquals": {
    "ec2:ResourceTag/Environment": "production"
  }
}
```

## Effect: Allow vs Deny

### Allow (Permitir)
- Otorga permiso explícito para realizar una acción
- Es el caso por defecto en políticas

### Deny (Denegar)
- Deniega explícitamente un permiso
- **Siempre prevalece sobre Allow**
- Se usa para excepciones restrictivas

## Tipos de Políticas

### 1. Política Basada en Identidad
- Se adjuntan a usuarios, grupos o roles
- Definen QUÉ pueden hacer

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::bucket/*"
  }]
}
```

### 2. Política Basada en Recursos
- Se adjuntan a recursos (S3, SQS, etc.)
- Definen QUIÉN puede hacerlo
- Incluyen campo "Principal"

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::123456789012:user/daniel"
    },
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::bucket/*"
  }]
}
```

## Flujo de Evaluación de Permisos

```
┌─ ¿Usuario autenticado?
│  No → DENEGAR
│  Sí ↓
├─ ¿Hay política Deny explícita?
│  Sí → DENEGAR (fin)
│  No ↓
├─ ¿Hay política Allow explícita?
│  Sí → PERMITIR (fin)
│  No ↓
└─ DENEGAR (por defecto)
```

## Mejores Prácticas

1. **Principio de Mínimo Privilegio**: Solo concede lo necesario
2. **Agrupa usuarios**: Usa grupos para gestión más fácil
3. **Usa roles para servicios**: No uses usuarios para servicios de AWS
4. **Revisa regularmente**: Audita permisos no usados
5. **Usa MFA**: Especialmente para usuarios con permisos críticos
6. **Monitorea actividades**: Usa CloudTrail para auditoría
7. **Evita credenciales root**: Crea usuarios IAM con permisos limitados
8. **Rotación de credenciales**: Cambia Access Keys regularmente
9. **Usa condiciones**: Añade capas adicionales de seguridad
10. **Documenta políticas**: Comenta el propósito de cada policy


