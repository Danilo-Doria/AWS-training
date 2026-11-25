# Actividad: Ciclo de vida de un bucket en S3

Práctica de creación, consulta, sincronización, edición, generación de enlaces temporales prefirmados, aplicación del principio de menor privilegio mediante políticas IAM en línea y eliminación segura de un bucket en Amazon S3 mediante AWS CLI v2.

---

## Evidencia de ejecución

### Parte 1: Crear bucket (Paso 3)
Creación del bucket `practica-danilo-3007`, verificación en el listado general y consulta de la región asignada (`us-east-2`):

```bash
Coder@rwadmin-90533010005:~$ aws s3 mb s3://practica-danilo-3007
make_bucket: practica-danilo-3007

Coder@rwadmin-90533010005:~$ aws s3 ls
2026-11-23 18:18:33 practica-danilo-3007

Coder@rwadmin-90533010005:~$ aws s3api get-bucket-location --bucket practica-danilo-3007
{
    "LocationConstraint": "us-east-2"
}
```

### Parte 2: Subir archivos (Paso 7)
Creación de los archivos locales (`notas.txt`, `tareas.txt`, `secreto.txt`), subida individual con `cp`, subida en lote con `sync` filtrado y confirmación del contenido del bucket con tamaños legibles y resumen total:

```bash
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ echo "Contenido de notas" > notas.txt
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ echo "Lista de tareas" > tareas.txt
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ echo "Informacion secreta" > secreto.txt

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 cp notas.txt s3://practica-danilo-3007/
upload: ./notas.txt to s3://practica-danilo-3007/notas.txt       

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 sync . s3://practica-danilo-3007/ --exclude "*" --include "tareas.txt" --include "secreto.txt"
upload: ./tareas.txt to s3://practica-danilo-3007/tareas.txt     
upload: ./secreto.txt to s3://practica-danilo-3007/secreto.txt   

# Paso 7: Listar contenido con tamaños legibles y total
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 ls s3://practica-danilo-3007/ --recursive --human-readable --summarize
2026-11-23 18:33:35   19 Bytes notas.txt
2026-11-23 18:34:13   20 Bytes secreto.txt
2026-11-23 18:34:13   16 Bytes tareas.txt

Total Objects: 3
   Total Size: 55 Bytes
```

### Parte 3 y 4: Consultar, descargar y editar objetos (Paso 13)
Inspección de contenido en terminal (`cp ... -`), consulta de metadatos, descarga en carpeta local `descargas/`, renombramiento de objeto en S3 (`mv tareas.txt pendientes.txt`), edición local de `notas.txt` y sobrescritura en S3:

```bash
# Lectura en terminal sin descargar
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 cp s3://practica-danilo-3007/notas.txt -
Contenido de notas

# Consulta de metadatos de tareas.txt
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 ls s3://practica-danilo-3007/tareas.txt
2026-11-23 18:34:13         16 tareas.txt

# Descarga local en carpeta descargas
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ mkdir descargas
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ cd descargas
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/descargas$ aws s3 cp s3://practica-danilo-3007/secreto.txt .
download: s3://practica-danilo-3007/secreto.txt to ./secreto.txt

# Renombrado de tareas.txt a pendientes.txt dentro del bucket
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/descargas$ aws s3 mv s3://practica-danilo-3007/tareas.txt s3://practica-danilo-3007/pendientes.txt
move: s3://practica-danilo-3007/tareas.txt to s3://practica-danilo-3007/pendientes.txt

# Sobrescritura con versión actualizada de notas.txt
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 cp notas.txt s3://practica-danilo-3007/
upload: ./notas.txt to s3://practica-danilo-3007/notas.txt       

# Paso 13: Verificación de contenido actualizado (sobrescrito sin versionado)
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 cp s3://practica-danilo-3007/notas.txt -
Contenido de notas version 2.0
```

### Parte 5: Compartir sin hacerlo público (Paso 16)
Generación de una URL prefirmada temporal con caducidad de 60 segundos (`presign`):

```bash
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 presign s3://practica-danilo-3007/notas.txt --expires-in 60
https://practica-danilo-3007.s3.us-east-2.amazonaws.com/notas.txt?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIA2CUR7VRQWBBMMAMV%2F20261124%2Fus-east-2%2Fs3%2Faws4_request&X-Amz-Date=20261124T000017Z&X-Amz-Expires=60&X-Amz-SignedHeaders=host&X-Amz-Signature=eef903b326fe8032f154becbece52128362dfaba1929bf1a35bec4b0a5765a71
```

#### Verificación en navegador

- **Antes de expirar (< 60 segundos):** El navegador accede exitosamente al objeto privado mediante la firma temporal autorizada:

![Enlace temporal válido](image_1.png)

- **Paso 16 - Tras expirar (> 60 segundos):** El navegador recibe una respuesta XML de error `AccessDenied` con el mensaje `Request has expired`, demostrando que el bucket siempre se mantuvo privado y sin exposición pública permanente:

![Enlace temporal expirado](image_2.png)

### Parte 6: Menor privilegio (Pasos 21, 22 y 23)
Creación del usuario `lector-s3-danilo`, asignación de política en línea de solo lectura para objetos (`s3:GetObject` sobre `arn:aws:s3:::practica-danilo-3007/*`), generación de llaves y configuración del perfil `--profile lector-s3`:

```bash
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws iam create-user --user-name lector-s3-danilo     
{
    "User": {
        "Path": "/",
        "UserName": "lector-s3-danilo",
        "UserId": "AIDA*****************",
        "Arn": "arn:aws:iam::123456789112:user/lector-s3-danilo",
        "CreateDate": "2026-11-24T00:05:51+00:00"
    }
}

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws iam put-user-policy --user-name lector-s3-danilo --policy-name leer-objetos --policy-document file://solo-lectura.json

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws iam create-access-key --user-name lector-s3-danilo     
{
    "AccessKey": {
        "UserName": "lector-s3-danilo",
        "AccessKeyId": "AKIA*****************",
        "Status": "Active",
        "SecretAccessKey": "****************************************",
        "CreateDate": "2026-11-24T00:08:39+00:00"
    }
}

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws configure --profile lector-s3
# Configuración de credenciales y región us-east-2
```

#### Pruebas de permisos con el perfil `lector-s3`:

```bash
# Paso 21: Acción permitida (s3:GetObject)
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 cp s3://practica-danilo-3007/notas.txt - --profile lector-s3
Contenido de notas version 2.0

# Paso 22: Acción denegada (s3:ListBucket)
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 ls s3://practica-danilo-3007/ --profile lector-s3

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListObjectsV2 operation: User: arn:aws:iam::692869639265:user/lector-s3-danilo is not authorized to perform: s3:ListBucket on resource: "arn:aws:s3:::practica-danilo-3007" because no identity-based policy allows the s3:ListBucket action

# Paso 23: Acción denegada (s3:DeleteObject)
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 rm s3://practica-danilo-3007/secreto.txt --profile lector-s3
delete failed: s3://practica-danilo-3007/secreto.txt An error occurred (AccessDenied) when calling the DeleteObject operation: User: arn:aws:iam::692869639265:user/lector-s3-danilo is not authorized to perform: s3:DeleteObject on resource: "arn:aws:s3:::practica-danilo-3007/secreto.txt" because no identity-based policy allows the s3:DeleteObject action
```

### Parte 7: Eliminar recursos (Pasos 24 y 29)
Comprobación de la protección contra borrado de buckets no vacíos, vaciado recursivo, eliminación del bucket y remoción ordenada de la política en línea, llaves de acceso y usuario IAM:

```bash
# Paso 24: Intento de borrado de bucket con objetos (error esperado)
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 rb s3://practica-danilo-3007
remove_bucket failed: s3://practica-danilo-3007 An error occurred (BucketNotEmpty) when calling the DeleteBucket operation: The bucket you tried to delete is not empty

# Vaciado del bucket
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 rm s3://practica-danilo-3007/ --recursive
delete: s3://practica-danilo-3007/notas.txt
delete: s3://practica-danilo-3007/pendientes.txt
delete: s3://practica-danilo-3007/secreto.txt

# Borrado del bucket ya vacío
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 rb s3://practica-danilo-3007
remove_bucket: practica-danilo-3007

# Limpieza de recursos IAM asociados
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws iam delete-access-key --user-name lector-s3-danilo --access-key-id AKIA*****************

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws iam delete-user-policy --user-name lector-s3-danilo --policy-name leer-objetos

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws iam delete-user --user-name lector-s3-danilo     

# Paso 29: Verificación de limpieza total en IAM y S3
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws iam list-users
{
    "Users": [
        {
            "Path": "/",
            "UserName": "admin-danilo",
            "UserId": "AIDA*****************",
            "Arn": "arn:aws:iam::12345678912:user/admin-danilo",
            "CreateDate": "2026-11-20T00:08:59+00:00"
        }
    ]
}

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ aws s3 ls
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades$ 
```

---

## Preguntas de reflexión

1. **¿Por qué el nombre de tu bucket tuvo que ser distinto al de tus compañeros si cada uno tiene su propia cuenta?**  
   Porque el espacio de nombres (*namespace*) de Amazon S3 es global y compartido entre todas las cuentas de AWS a nivel mundial. Además, el nombre del bucket forma parte directa del registro de dominio DNS público (`nombre-bucket.s3.<region>.amazonaws.com`), por lo que no pueden existir dos buckets con el mismo nombre en ninguna cuenta ni región.

2. **El usuario del paso 21 pudo leer un archivo, pero en el paso 22 no pudo ver la lista. ¿Qué acción le faltaba en la política?**  
   Le faltaba el permiso `s3:ListBucket`. En S3, leer o descargar el contenido de un archivo es una acción sobre objetos (`s3:GetObject`), mientras que ver la lista de archivos que contiene el bucket es una acción sobre el contenedor en sí (`s3:ListBucket`). Bajo el principio de menor privilegio, cada acción requiere autorización explícita.

3. **En la política, ¿qué significa el `/*` al final del `Resource`? ¿Qué pasaría si se quitara?**  
   El comodín `/*` (`arn:aws:s3:::practica-danilo-3007/*`) indica que la regla aplica a **todos los objetos** dentro de ese bucket. Si se quitara (dejando únicamente `arn:aws:s3:::practica-danilo-3007`), el recurso representaría únicamente al bucket como contenedor (adecuado para acciones como `s3:ListBucket`), y cualquier intento de ejecutar `s3:GetObject` fallaría con `AccessDenied` porque esta última opera sobre objetos específicos y no sobre el bucket raíz.

4. **¿En qué se diferencia compartir un archivo con un enlace temporal de hacer público el bucket?**  
   Un enlace temporal prefirmado (`presign`) mantiene el bucket y sus objetos 100% privados; utiliza una firma criptográfica efímera ligada a las credenciales del emisor que vence en un tiempo determinado (ej. 60 segundos), limitando el acceso solo a ese archivo específico sin exponer el resto de la información. Por el contrario, hacer público el bucket modifica la política de acceso o ACLs para permitir que cualquier persona en Internet descargue o liste el contenido de forma anónima, ilimitada y sin caducidad.

5. **Según el modelo de responsabilidad compartida, si alguien deja un bucket público con datos de clientes, ¿de quién es la responsabilidad?**  
   Es responsabilidad exclusiva del **cliente** ("seguridad EN la nube"). AWS se encarga de la seguridad DE la nube (infraestructura física, centros de datos, red y aislamiento del servicio S3), mientras que la gestión de identidades, la configuración de permisos, el cifrado y la decisión de habilitar o deshabilitar el bloqueo de acceso público (*Block Public Access*) corresponden plenamente al usuario u organización administradora de la cuenta.
