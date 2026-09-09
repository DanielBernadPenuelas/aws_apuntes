# Mejores Prácticas de Seguridad en AWS IAM

## 1. Principio de Mínimo Privilegio

### Hacer:
```json
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:ListBucket"
  ],
  "Resource": "arn:aws:s3:::specific-bucket/*"
}
```

### Evitar:
```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```

**Por qué**: Limita el daño potencial en caso de compromiso de credenciales.

---

## 2. Gestión de Usuarios y Grupos

### Estructura Recomendada:

```
Organización
├── Grupo: Administradores
│   └── Política: AdministratorAccess
├── Grupo: Desarrolladores
│   ├── Política: EC2FullAccess
│   ├── Política: S3FullAccess
│   └── Política: CloudWatchLogsFullAccess
├── Grupo: DevOps
│   ├── Política: IAMFullAccess (limitado)
│   └── Política: CloudFormationFullAccess
└── Grupo: Lectura
    └── Política: ReadOnlyAccess
```

### Creación de Usuario Nuevo:

1. Crear usuario sin credenciales permanentes
2. Asignarlo a grupo apropiado
3. Habilitar MFA
4. Generar Access Keys solo si es necesario
5. Documentar el propósito

---

## 3. Protección de la Cuenta Root

### Mejores Prácticas:

```
1. Usar credenciales root solo para:
   ✓ Crear cuenta AWS
   ✓ Cambiar plan de facturación
   ✓ Cerrar cuenta

2. Habilitar MFA en root:
   aws iam enable-mfa-device \
     --serial-number arn:aws:iam::123456789012:mfa/root-mfa \
     --authentication-code1 123456 \
     --authentication-code2 654321

3. Guardar credenciales en lugar seguro:
   ✓ Bóveda segura
   ✓ Gestor de contraseñas empresarial
   ✓ Nunca en email o documentos

4. No usar Access Keys de root
```

---

## 4. Rotación de Credenciales

### Access Keys:

```bash
# Ver Access Keys actuales
aws iam list-access-keys --user-name daniel.bernad

# Crear nueva Access Key
aws iam create-access-key --user-name daniel.bernad

# Desactivar Access Key antigua
aws iam update-access-key \
  --user-name daniel.bernad \
  --access-key-id AKIAIOSFODNN7EXAMPLE \
  --status Inactive

# Esperar 24 horas para confirmar que funciona la nueva
# Después, eliminar la antigua:
aws iam delete-access-key \
  --user-name daniel.bernad \
  --access-key-id AKIAIOSFODNN7EXAMPLE
```

### Recomendación:
- Rotación cada **90 días**
- Inmediata si hay sospecha de compromiso

---

## 5. Políticas Basadas en Tags

### Problema:
Gestionar 100+ usuarios con permisos específicos es complejo.

### Solución:
Usar tags para organizar recursos y aplicar políticas dinámicamente.

### Implementación:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2ManagementByTeam",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:RebootInstances"
      ],
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Team": "${aws:username}"
        }
      }
    }
  ]
}
```

**Ventajas:**
- Escalable
- Reducción de políticas duplicadas
- Cambios automáticos con tags

---

## 6. Auditoría y Monitoreo

### CloudTrail:

```bash
# Habilitar CloudTrail
aws cloudtrail create-trail \
  --name OrganizationTrail \
  --s3-bucket-name my-trail-logs \
  --is-multi-region-trail

# Ver eventos recientes
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=Username,AttributeValue=daniel.bernad \
  --max-results 10
```

### CloudWatch:

```json
{
  "MetricAlarms": [
    {
      "AlarmName": "UnauthorizedOperations",
      "MetricName": "UnauthorizedOperationCount",
      "Threshold": 1,
      "ComparisonOperator": "GreaterThanOrEqualToThreshold"
    }
  ]
}
```

### AccessAnalyzer:

```bash
# Crear analizador
aws accessanalyzer create-analyzer \
  --analyzer-name MyAnalyzer \
  --type ACCOUNT

# Validar políticas
aws accessanalyzer validate-policy \
  --policy-document file://policy.json \
  --policy-type IDENTITY_POLICY
```

---

## 7. Condiciones de Seguridad Avanzadas

### Por IP Corporativa:

```json
{
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": [
        "203.0.113.0/24",
        "198.51.100.0/24"
      ]
    }
  }
}
```

### MFA Requerido:

```json
{
  "Condition": {
    "Bool": {
      "aws:MultiFactorAuthPresent": "true"
    }
  }
}
```

### Período de Tiempo:

```json
{
  "Condition": {
    "DateGreaterThanEquals": {
      "aws:CurrentTime": "2024-01-01T00:00:00Z"
    },
    "DateLessThanEquals": {
      "aws:CurrentTime": "2024-12-31T23:59:59Z"
    }
  }
}
```

### Solo Acceso desde VPC:

```json
{
  "Condition": {
    "StringEquals": {
      "aws:SourceVpc": "vpc-12345678"
    }
  }
}
```

---

## 8. Gestión de Roles Inter-Cuentas

### Caso de Uso:
Permitir que usuarios de una cuenta asuman role en otra.

### Configuración:

**Cuenta A (Origen):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::987654321098:role/CrossAccountRole"
    }
  ]
}
```

**Cuenta B (Destino):**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/daniel.bernad"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "unique-id-abc123"
        }
      }
    }
  ]
}
```

### Uso:
```bash
aws sts assume-role \
  --role-arn arn:aws:iam::987654321098:role/CrossAccountRole \
  --role-session-name daniel-session \
  --external-id unique-id-abc123
```

---

## 9. Políticas Explícitas de Denegación

### Caso: Proteger recursos críticos

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyDeleteDatabaseFromProduction",
      "Effect": "Deny",
      "Action": "rds:DeleteDBInstance",
      "Resource": "arn:aws:rds:*:*:db/production-*",
      "Condition": {
        "StringNotEquals": {
          "aws:PrincipalTag/Role": "DatabaseAdmin"
        }
      }
    }
  ]
}
```

---

## 10. Checklist de Seguridad IAM

- [ ] Root account protegida con MFA
- [ ] Todos los usuarios tienen MFA habilitado
- [ ] Sin Access Keys de root
- [ ] Usuarios agrupados correctamente
- [ ] Políticas siguen principio de mínimo privilegio
- [ ] Revisión trimestral de permisos
- [ ] CloudTrail habilitado
- [ ] Alertas configuradas para actividades sospechosas
- [ ] Documentación de roles y responsabilidades
- [ ] Plan de respuesta ante incidentes
- [ ] Políticas de contraseña fuertes
- [ ] Rotación de credenciales programada

---

## 11. Herramientas Útiles

### AWS CLI:
```bash
# Listar usuarios sin MFA
aws iam get-credential-report

# Ver políticas de un usuario
aws iam list-user-policies --user-name daniel.bernad

# Simular acción
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/daniel \
  --action-names s3:GetObject
```

### AWS CloudFormation:
Automatizar creación de usuarios y roles

### AWS Config:
Monitorear cambios en configuración de IAM

### IAM Access Analyzer:
Identificar recursos con acceso público o inter-cuentas no deseado


