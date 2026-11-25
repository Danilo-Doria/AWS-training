# Actividad: un rol para tu instancia EC2

Práctica de preparación de recurso en S3, definición de políticas de confianza y permisos en formato JSON, creación de un rol de IAM bajo el principio de menor privilegio, configuración de perfil de instancia para Amazon EC2 y comprobación de las reglas de integridad referencial mediante AWS CLI v2.

---

## Evidencia de ejecución

### Parte 1: Preparar el recurso a proteger (Pasos 1, 2 y 3)
Creación del bucket privado `reportes-danilo-3007`, creación local del archivo `reporte.txt`, carga hacia Amazon S3 y verificación del objeto listado en el bucket:

```bash
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws s3 mb s3://reportes-danilo-3007
make_bucket: reportes-danilo-3007

danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ echo "Este es el contenido del reporte" > reporte.txt

danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws s3 cp reporte.txt s3://reportes-danilo-3007/
upload: ./reporte.txt to s3://reportes-danilo-3007/reporte.txt

# Paso 3: Confirmación de objeto existente en el bucket
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws s3 ls s3://reportes-danilo-3007
2026-11-24 22:11:42         33 reporte.txt
```

### Parte 2: Escribir las dos políticas (Pasos 4, 5 y 6)
Definición de la política de confianza (`confianza.json`) autorizando exclusivamente al servicio EC2 (`sts:AssumeRole`), definición de la política de permisos (`permisos.json`) autorizando únicamente lectura de objetos (`s3:GetObject`), y validación de sintaxis JSON con `python3 -m json.tool`:

```bash
# Paso 4: Política de confianza (quién puede asumir el rol)
cat > confianza.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": { "Service": "ec2.amazonaws.com" },
    "Action": "sts:AssumeRole"
  }]
}
EOF

# Paso 5: Política de permisos (qué puede hacer quien tenga el rol)
cat > permisos.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::reportes-danilo-3007/*"
  }]
}
EOF

# Paso 6: Comprobar que los archivos son JSON válidos
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ python3 -m json.tool confianza.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}

danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ python3 -m json.tool permisos.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::reportes-danilo-3007/*"
        }
    ]
}
```

### Parte 3: Crear el rol (Pasos 7, 8, 9 y 10)
Validación de la obligatoriedad de la política de confianza (error provocado), creación exitosa del rol `ec2-lector-s3` con `confianza.json`, consulta de atributos/ARN y constatación del estado inicial sin permisos asociados:

```bash
# Paso 7: Intento de creación sin política de confianza (error esperado)
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam create-role --role-name ec2-lector-s3

aws: [ERROR]: An error occurred (ParamValidation): the following arguments are required: --assume-role-policy-document

# Paso 8: Creación exitosa del rol adjuntando la política de confianza
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam create-role --role-name ec2-lector-s3 --assume-role-policy-document file://confianza.json
{
    "Role": {
        "Path": "/",
        "RoleName": "ec2-lector-s3",
        "RoleId": "AROA*****************",
        "Arn": "arn:aws:iam::123456789012:role/ec2-lector-s3",
        "CreateDate": "2026-11-25T03:49:54+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "ec2.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        }
    }
}

# Paso 9: Consulta de los datos del rol y obtención de su ARN
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam get-role --role-name ec2-lector-s3
{
    "Role": {
        "Path": "/",
        "RoleName": "ec2-lector-s3",
        "RoleId": "AROA*****************",
        "Arn": "arn:aws:iam::123456789012:role/ec2-lector-s3",
        "CreateDate": "2026-11-25T03:49:54+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "ec2.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "MaxSessionDuration": 3600,
        "MaxSigninSessionDuration": 3600,
        "RoleLastUsed": {}
    }
}

# Paso 10: Verificación de estado inicial (sin políticas en línea asignadas)
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam list-role-policies --role-name ec2-lector-s3
{
    "PolicyNames": []
}
```

### Parte 4: Dar permisos (Pasos 11, 12 y 13)
Asociación de la política en línea `leer-reportes` basada en `permisos.json`, confirmación de registro en el rol y lectura de la definición de permisos directamente desde AWS:

```bash
# Paso 11: Asignación de política en línea de menor privilegio
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam put-role-policy --role-name ec2-lector-s3 --policy-name leer-reportes --policy-document file://permisos.json

# Paso 12: Confirmación de la política en el listado
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam list-role-policies --role-name ec2-lector-s3
{
    "PolicyNames": [
        "leer-reportes"
    ]
}

# Paso 13: Inspección del contenido de la política leer-reportes en AWS
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam get-role-policy --role-name ec2-lector-s3 --policy-name leer-reportes
{
    "RoleName": "ec2-lector-s3",
    "PolicyName": "leer-reportes",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject"
                ],
                "Resource": "arn:aws:s3:::reportes-danilo-3007/*"
            }
        ]
    }
}
```

### Parte 5: Editar metadatos del rol (Pasos 14, 15 y 16)
Incorporación de descripción explicativa `Rol para que EC2 lea reportes`, etiquetado organizacional (`semana=1`) y consulta consolidada del rol con sus nuevos atributos:

```bash
# Paso 14: Adición de descripción al rol
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam update-role --role-name ec2-lector-s3 --description "Rol para que EC2 lea reportes"

# Paso 15: Asignación de etiqueta
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam tag-role --role-name ec2-lector-s3 --tags Key=semana,Value=1

# Paso 16: Verificación de metadatos, descripción y etiquetas
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam get-role --role-name ec2-lector-s3
{
    "Role": {
        "Path": "/",
        "RoleName": "ec2-lector-s3",
        "RoleId": "AROA*****************",
        "Arn": "arn:aws:iam::123456789012:role/ec2-lector-s3",
        "CreateDate": "2026-11-25T03:49:54+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "ec2.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "Description": "Rol para que EC2 lea reportes",
        "MaxSessionDuration": 3600,
        "MaxSigninSessionDuration": 3600,
        "Tags": [
            {
                "Key": "semana",
                "Value": "1"
            }
        ],
        "RoleLastUsed": {}
    }
}
```

### Parte 6: Perfil de instancia (Pasos 17, 18 y 19)
Creación del perfil de instancia `ec2-lector-s3` (contenedor para EC2), inserción del rol dentro del perfil y verificación de contención:

```bash
# Paso 17: Creación del perfil de instancia
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam create-instance-profile --instance-profile-name ec2-lector-s3
{
    "InstanceProfile": {
        "Path": "/",
        "InstanceProfileName": "ec2-lector-s3",
        "InstanceProfileId": "AIPA*****************",
        "Arn": "arn:aws:iam::123456789012:instance-profile/ec2-lector-s3",
        "CreateDate": "2026-11-25T04:13:42+00:00",
        "Roles": []
    }
}

# Paso 18: Vinculación del rol dentro del perfil de instancia
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam add-role-to-instance-profile --instance-profile-name ec2-lector-s3 --role-name ec2-lector-s3

# Paso 19: Confirmación de que el perfil de instancia contiene el rol
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam get-instance-profile --instance-profile-name ec2-lector-s3
{
    "InstanceProfile": {
        "Path": "/",
        "InstanceProfileName": "ec2-lector-s3",
        "InstanceProfileId": "AIPA*****************",
        "Arn": "arn:aws:iam::123456789012:instance-profile/ec2-lector-s3",
        "CreateDate": "2026-11-25T04:13:42+00:00",
        "Roles": [
            {
                "Path": "/",
                "RoleName": "ec2-lector-s3",
                "RoleId": "AROA*****************",
                "Arn": "arn:aws:iam::123456789012:role/ec2-lector-s3",
                "CreateDate": "2026-11-25T03:49:54+00:00",
                "AssumeRolePolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement": [
                        {
                            "Effect": "Allow",
                            "Principal": {
                                "Service": "ec2.amazonaws.com"
                            },
                            "Action": "sts:AssumeRole"
                        }
                    ]
                }
            }
        ],
        "Tags": []
    }
}
```

### Parte 7: Comprobar las reglas de IAM (Paso 20)
Intento de eliminación del rol mediante `aws iam delete-role`, provocando y verificando el error `DeleteConflict` debido a que el rol se encuentra activo dentro de un perfil de instancia:

```bash
# Paso 20: Intento de borrado de rol asociado (error esperado)
danilo@Danilo:~/Documents/Riwi 2026/complements/AWS/AWS-training/semana_1/role_practice$ aws iam delete-role --role-name ec2-lector-s3

aws: [ERROR]: An error occurred (DeleteConflict) when calling the DeleteRole operation: Cannot delete entity, must remove roles from instance profile first.
```

*(Nota: Conforme a lo estipulado en la guía del laboratorio, el bucket `reportes-danilo-3007` con `reporte.txt`, el rol `ec2-lector-s3` y el perfil de instancia `ec2-lector-s3` se mantuvieron creados y listos para ser consumidos en la posterior práctica de Amazon EC2).*

---

## Preguntas de reflexión

1. **¿Qué diferencia hay entre la política de confianza y la política de permisos? ¿Qué pasaría si el rol tuviera solo una de las dos?**  
   - **Política de confianza (*Trust Policy*):** Define **quién** (qué entidad principal, servicio de AWS como `ec2.amazonaws.com` o cuenta externa) tiene autorización para asumir el rol (`sts:AssumeRole`) y recibir credenciales temporales. Es una política obligatoria basada en recursos incrustada en el rol.
   - **Política de permisos (*Permissions Policy*):** Define **qué** acciones puede ejecutar quien tenga el rol asumido sobre qué recursos específicos (por ejemplo, `s3:GetObject` sobre `arn:aws:s3:::reportes-danilo-3007/*`).
   - **Si solo tuviera política de confianza:** El servicio EC2 podría asumir el rol y obtener tokens temporales de STS, pero al no tener permisos asignados (*Deny* implícito por defecto), cualquier comando o llamada a la API de AWS fallaría inmediatamente con `AccessDenied`.
   - **Si solo tuviera política de permisos:** En AWS no es posible crear un rol sin política de confianza (como se evidenció en el Paso 7 con el error `ParamValidation`). Y de existir conceptualmente, ninguna entidad o servicio tendría permiso para ponérselo, haciendo imposible usar sus permisos.

2. **¿Por qué es más seguro que la instancia use un rol en vez de guardar una llave de acceso con `aws configure`?**  
   - **Eliminación de secretos estáticos en disco:** Con `aws configure`, el `AccessKeyId` y el `SecretAccessKey` quedan almacenados en texto plano en el archivo local `~/.aws/credentials`. Si un atacante compromete el servidor mediante una vulnerabilidad web, acceso SSH no autorizado o una copia de seguridad expuesta (snapshot/AMI), las llaves pueden ser extraídas y utilizadas desde cualquier parte de Internet de forma indefinida.
   - **Credenciales temporales y rotación automática:** Mediante un perfil de instancia, AWS Security Token Service (STS) entrega credenciales temporales a través del servicio de metadatos de la instancia (IMDS). Estas credenciales caducan en pocas horas y el AWS CLI o SDK las renueva automáticamente sin intervención humana.
   - **Menor superficie de ataque:** Si una credencial temporal llega a ser interceptada, su validez es sumamente corta, reduciendo drásticamente la ventana de exposición y evitando credenciales huérfanas o permanentes.

3. **Con la política `leer-reportes`, ¿la instancia podrá listar el contenido del bucket? ¿Y borrar `reporte.txt`? Explica por qué.**  
   - **Listar el contenido (`aws s3 ls`):** **No podrá.** En Amazon S3, listar los objetos de un bucket requiere la acción `s3:ListBucket` aplicada a nivel de contenedor (`arn:aws:s3:::reportes-danilo-3007`). La política `leer-reportes` únicamente concede `s3:GetObject` sobre la ruta de objetos (`/*`).
   - **Borrar `reporte.txt` (`aws s3 rm`):** **No podrá.** Eliminar un archivo requiere la acción explícita `s3:DeleteObject`.
   - **Explicación:** En AWS IAM rige el principio de menor privilegio con *Deny* implícito: toda acción no declarada de manera explícita en el bloque `"Action"` está estrictamente denegada.

4. **En el error del paso 20, ¿qué le falta por hacer antes de poder borrar el rol?**  
   El error `DeleteConflict` evidencia las reglas de integridad referencial de AWS IAM, que impiden dejar dependencias rotas o contenedores huérfanos. Antes de poder borrar el rol `ec2-lector-s3`, se debe realizar la limpieza en el siguiente orden secuencial:
   1. **Desvincular el rol del perfil de instancia:** `aws iam remove-role-from-instance-profile --instance-profile-name ec2-lector-s3 --role-name ec2-lector-s3`.
   2. **Eliminar la política en línea del rol:** `aws iam delete-role-policy --role-name ec2-lector-s3 --policy-name leer-reportes` (o desvincular políticas administradas si las hubiera).
   3. **Eliminar el perfil de instancia (opcional para limpieza completa):** `aws iam delete-instance-profile --instance-profile-name ec2-lector-s3`.
   4. **Eliminar el rol:** `aws iam delete-role --role-name ec2-lector-s3`.

5. **Si mañana la instancia también necesita subir archivos al bucket, ¿qué cambiarías y en cuál de los dos archivos?**  
   - Se modificaría **únicamente el archivo `permisos.json`** (la política de permisos), ya que es el documento encargado de definir qué acciones tiene permitidas la identidad.
   - En `permisos.json`, se agregaría la acción `"s3:PutObject"` dentro de la lista `"Action"`:
     ```json
     {
       "Version": "2012-10-17",
       "Statement": [{
         "Effect": "Allow",
         "Action": [
           "s3:GetObject",
           "s3:PutObject"
         ],
         "Resource": "arn:aws:s3:::reportes-danilo-3007/*"
       }]
     }
     ```
   - Posteriormente se actualizaría en AWS corriendo:  
     `aws iam put-role-policy --role-name ec2-lector-s3 --policy-name leer-reportes --policy-document file://permisos.json`
   - El archivo `confianza.json` **no se modificaría**, debido a que la entidad autorizada para asumir el rol sigue siendo la misma (`ec2.amazonaws.com`).
