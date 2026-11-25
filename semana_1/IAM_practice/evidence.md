# Actividad: Ciclo de vida de un usuario IAM

Práctica de creación, consulta, edición, asignación de permisos (principio de menor privilegio) y eliminación de un usuario IAM mediante AWS CLI v2.

---

## Evidencia de ejecución

### Parte 1: Crear usuario (Paso 3)
Verificación de identidad inicial, creación del usuario `dev-danilo` y consulta de sus datos:

```bash
Coder@aws-cli:~\$ aws sts get-caller-identity
{
    "UserId": "AIDA*****************",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/admin-danilo"
}

Coder@aws-cli:~\$ aws iam create-user --user-name dev-danilo
{
    "User": {
        "Path": "/",
        "UserName": "dev-danilo",
        "UserId": "AIDA*****************",
        "Arn": "arn:aws:iam::123456789012:user/dev-danilo",
        "CreateDate": "2026-11-20T19:29:35+00:00"
    }
}

Coder@aws-cli:~\$ aws iam get-user --user-name dev-danilo
{
    "User": {
        "Path": "/",
        "UserName": "dev-danilo",
        "UserId": "AIDA*****************",
        "Arn": "arn:aws:iam::123456789012:user/dev-danilo",
        "CreateDate": "2026-11-20T19:29:35+00:00"
    }
}
```

### Parte 2: Editar usuario (Paso 6)
Asignación de etiqueta `area=desarrollo`, cambio de nombre a `lector-danilo` y verificación de actualización del ARN:

```bash
Coder@aws-cli:~\$ aws iam tag-user --user-name dev-danilo --tags Key=area,Value=desarrollo

Coder@aws-cli:~\$ aws iam update-user --user-name dev-danilo --new-user-name lector-danilo

Coder@aws-cli:~\$ aws iam get-user --user-name lector-danilo
{
    "User": {
        "Path": "/",
        "UserName": "lector-danilo",
        "UserId": "AIDA*****************",
        "Arn": "arn:aws:iam::123456789012:user/lector-danilo",
        "CreateDate": "2026-11-20T19:29:35+00:00",
        "Tags": [
            {
                "Key": "area",
                "Value": "desarrollo"
            }
        ]
    }
}
```

### Parte 3: Dar permisos (Paso 8)
Asociación de la política administrada `AmazonEC2ReadOnlyAccess` y confirmación:

```bash
Coder@aws-cli:~\$ aws iam attach-user-policy --user-name lector-danilo --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

Coder@aws-cli:~\$ aws iam list-attached-user-policies --user-name lector-danilo
{
    "AttachedPolicies": [
        {
            "PolicyName": "AmazonEC2ReadOnlyAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess"
        }
    ]
}
```

### Parte 4: Probar el menor privilegio (Pasos 12 y 13)
Generación de llaves de acceso (credenciales redactadas por seguridad), configuración del perfil y pruebas de acciones permitidas y denegadas:

```bash
Coder@aws-cli:~\$ aws iam create-access-key --user-name lector-danilo
{
    "AccessKey": {
        "UserName": "lector-danilo",
        "AccessKeyId": "AKIA*****************",
        "Status": "Active",
        "SecretAccessKey": "****************************************",
        "CreateDate": "2026-11-20T19:58:00+00:00"
    }
}

Coder@aws-cli:~\$ aws sts get-caller-identity --profile lector-danilo
{
    "UserId": "AIDA*****************",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/lector-danilo"
}

# Paso 12: Acción permitida (EC2 ReadOnly)
Coder@aws-cli:~\$ aws ec2 describe-instances --profile lector-danilo
{
    "Reservations": []
}

# Paso 13: Acción no permitida (IAM ListUsers)
Coder@aws-cli:~\$ aws iam list-users --profile lector-danilo

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListUsers operation: User: arn:aws:iam::123456789012:user/lector-danilo is not authorized to perform: iam:ListUsers on resource: arn:aws:iam::123456789012:user/ because no identity-based policy allows the iam:ListUsers action.
```

### Parte 5: Quitar permisos (Paso 15)
Desvinculación de la política y comprobación de revocación de permisos:

```bash
Coder@aws-cli:~\$ aws iam detach-user-policy --user-name lector-danilo --policy-arn arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

Coder@aws-cli:~\$ aws iam list-attached-user-policies --user-name lector-danilo
{
    "AttachedPolicies": []
}

# Paso 15: Repetición de consulta EC2 tras quitar permisos
Coder@aws-cli:~\$ aws ec2 describe-instances --profile lector-danilo

aws: [ERROR]: An error occurred (UnauthorizedOperation) when calling the DescribeInstances operation: You are not authorized to perform this operation. User: arn:aws:iam::123456789012:user/lector-danilo is not authorized to perform: ec2:DescribeInstances because no identity-based policy allows the ec2:DescribeInstances action
```

### Parte 6: Eliminar recursos (Paso 19)
Eliminación en orden de las llaves de acceso, el usuario y verificación final de limpieza:

```bash
Coder@aws-cli:~\$ aws iam delete-access-key --user-name lector-danilo --access-key-id AKIA*****************

Coder@aws-cli:~\$ aws iam delete-user --user-name lector-danilo

# Paso 19: Confirmación de que el usuario ya no existe en la cuenta
Coder@aws-cli:~\$ aws iam list-users
{
    "Users": [
        {
            "Path": "/",
            "UserName": "admin-danilo",
            "UserId": "AIDA*****************",
            "Arn": "arn:aws:iam::123456789012:user/admin-danilo",
            "CreateDate": "2026-11-20T00:08:59+00:00"
        }
    ]
}
```

*(Nota: También se eliminó el bloque `[profile lector-danilo]` y `[lector-danilo]` de los archivos locales `~/.aws/config` y `~/.aws/credentials`).*

---

## Preguntas de reflexión

1. **¿Qué podía hacer el usuario justo después de crearlo, antes de adjuntarle una política? ¿Por qué?**
   No podía realizar ninguna acción ya que en AWS y AWS IAM, todo usuario nuevo nace con cero permisos por defecto y solo puede ejecutar acciones que se le permitan explícitamente mediante una política.

2. **En el error del paso 13, ¿qué tres datos te dice el mensaje?**
   * **Quién intentó la acción:** El ARN del usuario (`arn:aws:iam::123456789012:user/lector-danilo`).
   * **Qué acción intentó ejecutar:** La operación de API `iam:ListUsers`.
   * **Sobre qué recurso y por qué falló:** Sobre el recurso `arn:aws:iam::123456789012:user/`, indicando que ninguna política basada en identidad autoriza esa acción.

3. **¿Por qué cambió el Arn al renombrar el usuario?**
   Porque el ARN (Amazon Resource Name) de un usuario IAM incluye su ruta y su nombre legible (`.../user/<nombre>`). Aunque su identificador interno (`UserId`) se mantiene igual, al cambiar el nombre se actualiza su ruta de identificación externa.

4. **¿Por qué AWS obliga a borrar las llaves antes de borrar el usuario?**
   Por seguridad e integridad referencial: las llaves de acceso son credenciales activas que podrían seguir usándose en aplicaciones o scripts. Obligar a eliminarlas primero evita que queden credenciales huérfanas o activas sin un propietario asociado al que auditar.

5. **¿Qué pasaría si le hubieras dado `AdministratorAccess` en vez de solo lectura? ¿En qué caso real estaría justificado?**
   El usuario tendría control total sobre todos los servicios, recursos y facturación de la cuenta (incluyendo crear o borrar infraestructura crítica y otros usuarios), rompiendo el principio de menor privilegio. En un entorno real solo se justifica para cuentas de administradores principales de infraestructura o equipos de arquitectura/DevOps encargados de gestionar la seguridad y configuración global de la cuenta.
