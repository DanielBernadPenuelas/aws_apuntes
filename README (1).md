

Proyecto de AWS trata de subir todos mis apuntes que me he ido haciendo de AWS

## Contenido del Proyecto

Este repositorio contiene:

- **Políticas de IAM**: Ejemplos de permisos de AWS correctamente configurados
- **S3 Bucket Policies**: Políticas de acceso a almacenamiento S3
- **Role Policies**: Políticas para roles de IAM
- **Documentación**: Guías y explicaciones de cada política

## Estructura del Proyecto

```
aws-iam-policies/
├── README.md
├── docs/
│   ├── conceptos-basicos.md
│   ├── mejores-practicas.md
│   └── guia-seguridad.md
├── policies/
│   ├── s3/
│   │   ├── s3-read-only.json
│   │   ├── s3-full-access.json
│   │   └── s3-conditional-access.json
│   ├── ec2/
│   │   ├── ec2-full-access.json
│   │   └── ec2-read-only.json
└── 
```

## Conceptos Clave

### Estructura de una Política IAM

Toda política JSON de AWS tiene esta estructura:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Descripción opcional del statement",
      "Effect": "Allow" | "Deny",
      "Principal": { "AWS": "arn:aws:iam::ACCOUNT_ID:user/USERNAME" },
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": "arn:aws:s3:::bucket-name/*",
      "Condition": { ... }
    }
  ]
}
```

### Elementos Principales

- **Version**: Versión de la política (generalmente "2012-10-17")
- **Statement**: Array de declaraciones de permisos
- **Effect**: "Allow" (permitir) o "Deny" (denegar)
- **Principal**: Quién tiene estos permisos
- **Action**: Qué acciones se permiten
- **Resource**: Sobre qué recursos aplican los permisos
- **Condition**: Condiciones opcionales para aplicar el permiso

## Ejemplos Incluidos

### 1. Acceso de Solo Lectura a S3

Permite leer objetos con tag específico de producción.

### 2. Acceso Completo a S3

Permisos totales sobre buckets S3.

### 3. Acceso Condicional EC2

Permisos sobre instancias EC2 según tags.

---

**Autor**: Daniel Bernad Peñuelas  
**Última actualización**: Septiembre 2026  
