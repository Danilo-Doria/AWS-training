# AWS Training - Bitácora de Aprendizaje

Repositorio dedicado a documentar el progreso práctico, conceptual y de laboratorio en **Amazon Web Services (AWS)**, utilizando principalmente **AWS CLI v2** y la consola de administración bajo las mejores prácticas de arquitectura y seguridad en la nube.

---

## 📁 Estructura del Repositorio

El aprendizaje y los laboratorios se organizan por semanas formativas y prácticas temáticas. Actualmente, la **Semana 1** contiene cuatro actividades prácticas especializadas:

```text
AWS-training/
├── README.md
└── semana_1/
    ├── IAM_practice/              # Actividad 1: Gestión de identidades, usuarios y ciclo de vida en IAM
    │   ├── 01-info.md
    │   ├── 02-command.md
    │   ├── 03-activity.md
    │   ├── evidence-raw.txt
    │   └── evidence.md
    ├── bucket_practice/           # Actividad 2: Almacenamiento en S3, URLs prefirmadas y menor privilegio
    │   ├── 01-info.md
    │   ├── 02-command.md
    │   ├── 03-activity.md
    │   ├── descargas/
    │   ├── evidence-raw.txt
    │   ├── evidence.md
    │   ├── image_1.png
    │   ├── image_2.png
    │   ├── notas.txt
    │   ├── secreto.txt
    │   ├── solo-lectura.json
    │   └── tareas.txt
    ├── role_practice/             # Actividad 3: Roles IAM, políticas de confianza y perfiles de instancia EC2
    │   ├── 01-info.md
    │   ├── 02-command.md
    │   ├── 03-activity.md
    │   ├── confianza.json
    │   ├── evidence-raw.txt
    │   ├── evidence.md
    │   ├── permisos.json
    │   └── reporte.txt
    └── EC2_practice/              # Actividad 4: Servidores virtuales EC2, Security Groups, User Data y roles
        ├── 01-info.md
        ├── 02-command.md
        ├── 03-activity.md
        ├── evidence-raw.txt
        ├── evidence.md
        └── practica-ec2/
            └── inicio.sh
```

---

## 📌 Índice de Contenidos

- [📁 Estructura del Repositorio](#-estructura-del-repositorio)
- [🚀 Semana 1: Fundamentos de IAM, S3, Roles y Cómputo en EC2](#-semana-1-fundamentos-de-iam-s3-roles-y-cómputo-en-ec2)
  - [Actividad 1: Fundamentos de IAM y AWS CLI (`IAM_practice`)](#-actividad-1-fundamentos-de-iam-y-aws-cli-iam_practice)
  - [Actividad 2: Almacenamiento en la Nube con Amazon S3 y Menor Privilegio (`bucket_practice`)](#-actividad-2-almacenamiento-en-la-nube-con-amazon-s3-y-menor-privilegio-bucket_practice)
  - [Actividad 3: Roles de IAM y Perfiles de Instancia para EC2 (`role_practice`)](#-actividad-3-roles-de-iam-y-perfiles-de-instancia-para-ec2-role_practice)
  - [Actividad 4: Servidores Virtuales en Amazon EC2 (`EC2_practice`)](#-actividad-4-servidores-virtuales-en-amazon-ec2-ec2_practice)
- [💻 Requisitos y Entorno de Trabajo](#-requisitos-y-entorno-de-trabajo)
- [👨‍💻 Autor](#-autor)

---

## 🚀 Semana 1: Fundamentos de IAM, S3, Roles y Cómputo en EC2

En esta primera semana se cubren los pilares fundamentales de AWS: seguridad e identidades en IAM, almacenamiento de objetos en S3, delegación de credenciales efímeras con roles y el despliegue de infraestructura de cómputo en la nube mediante instancias EC2.

---

### 🔹 Actividad 1: Fundamentos de IAM y AWS CLI (`IAM_practice`)

#### 🎯 Objetivos de la Actividad
- Comprender la arquitectura de seguridad, autenticación y autorización en AWS mediante **IAM (Identity and Access Management)**.
- Dominar el uso de **AWS CLI v2** para la gestión de identidades, perfiles, credenciales y políticas.
- Aplicar de forma práctica el **principio de menor privilegio** (*Least Privilege*).
- Implementar y comprobar el ciclo de vida completo de un usuario IAM (creación, metadatos/etiquetas, asociación de políticas, pruebas de autorización y eliminación segura).

#### 📁 Estructura y Archivos

| Archivo | Descripción |
|---|---|
| [`semana_1/IAM_practice/01-info.md`](./semana_1/IAM_practice/01-info.md) | Fundamentos conceptuales de IAM: usuarios, grupos, roles, políticas JSON, perfiles de instancia y reglas de evaluación. |
| [`semana_1/IAM_practice/02-command.md`](./semana_1/IAM_practice/02-command.md) | Cheat sheet y guía de referencia rápida de comandos AWS CLI para IAM. |
| [`semana_1/IAM_practice/03-activity.md`](./semana_1/IAM_practice/03-activity.md) | Guía paso a paso del laboratorio: *Ciclo de vida de un usuario IAM*. |
| [`semana_1/IAM_practice/evidence.md`](./semana_1/IAM_practice/evidence.md) | Informe de evidencias del laboratorio con trazas de ejecución CLI y respuestas a preguntas de reflexión. |
| [`semana_1/IAM_practice/evidence-raw.txt`](./semana_1/IAM_practice/evidence-raw.txt) | Registro crudo de terminal con los comandos y respuestas obtenidas. |

#### 🛠️ Actividad Práctica: Ciclo de Vida de un Usuario IAM

En este laboratorio se realizó la gestión integral de un usuario IAM exclusivamente mediante la terminal:

1. **Creación e Identidad:**
   - Creación del usuario `dev-danilo` y verificación de atributos con `aws iam get-user`.
   - Constatación del estado inicial: sin permisos (*Deny* implícito por defecto).

2. **Edición y Metadatos:**
   - Asignación de etiquetas organizacionales (`area=desarrollo`) usando `aws iam tag-user`.
   - Renombrado del usuario a `lector-danilo` con `aws iam update-user`, comprobando la actualización de su ARN manteniendo el mismo `UserId` y etiquetas.

3. **Asignación de Permisos:**
   - Asociación de la política administrada por AWS `AmazonEC2ReadOnlyAccess` mediante `aws iam attach-user-policy`.

4. **Prueba del Principio de Menor Privilegio:**
   - Generación de credenciales programáticas (`AccessKeyId` y `SecretAccessKey`).
   - Configuración de un perfil local independiente (`aws configure --profile lector-danilo`).
   - **Acción permitida:** `aws ec2 describe-instances --profile lector-danilo` respondió con éxito (`Reservations: []`).
   - **Acción denegada:** `aws iam list-users --profile lector-danilo` arrojó error `AccessDenied` (*no identity-based policy allows the iam:ListUsers action*), validando el aislamiento de permisos.

5. **Revocación de Permisos:**
   - Desvinculación de la política (`aws iam detach-user-policy`).
   - Comprobación de que la acción de EC2 previamente autorizada ahora responde con error `UnauthorizedOperation`.

6. **Limpieza Segura y Orden de Eliminación:**
   - Cumplimiento del orden estricto de borrado en AWS IAM para evitar dependencias bloqueadas:
     1. Eliminación de llaves de acceso (`aws iam delete-access-key`).
     2. Verificación de desvinculación de políticas.
     3. Eliminación del usuario (`aws iam delete-user`).
     4. Confirmación de inexistencia del usuario y limpieza de perfiles en `~/.aws/credentials` y `~/.aws/config`.

#### 💡 Aprendizajes Clave
- **Deny por defecto:** Un usuario nuevo no puede realizar ninguna acción en la cuenta hasta recibir una política explícita.
- **Precedencia de permisos:** Un `Deny` explícito siempre invalida cualquier `Allow`.
- **Integridad referencial en IAM:** AWS no permite eliminar usuarios con llaves o políticas activas para evitar credenciales huérfanas.
- **Gestión segura de credenciales:** El uso de perfiles nombrados (`--profile`) previene la sobreescritura accidental de credenciales administradoras locales.

---

### 🔹 Actividad 2: Almacenamiento en la Nube con Amazon S3 y Menor Privilegio (`bucket_practice`)

#### 🎯 Objetivos de la Actividad
- Comprender la arquitectura de almacenamiento de objetos con **Amazon S3 (Simple Storage Service)**.
- Dominar el uso de comandos de alto nivel (`aws s3`) y bajo nivel (`aws s3api`) en **AWS CLI v2**.
- Administrar el ciclo de vida de objetos en S3: subida, descarga, inspección, sincronización por lotes (`sync`), renombramiento y eliminación.
- Implementar mecanismos seguros para compartir archivos privados mediante **enlaces temporales prefirmados (`presign`)** sin exponer el bucket al público.
- Aplicar el **principio de menor privilegio** en S3 creando políticas en línea personalizadas en IAM (`s3:GetObject` vs `s3:ListBucket`).
- Conocer la regla de integridad y protección de borrado de buckets no vacíos (*BucketNotEmpty*).

#### 📁 Estructura y Archivos

| Archivo | Descripción |
|---|---|
| [`semana_1/bucket_practice/01-info.md`](./semana_1/bucket_practice/01-info.md) | Fundamentos de Amazon S3: buckets, objetos, claves, prefijos, reglas de nomenclatura, características y analogías. |
| [`semana_1/bucket_practice/02-command.md`](./semana_1/bucket_practice/02-command.md) | Cheat sheet de comandos CLI (`aws s3` y `aws s3api`) para buckets, objetos, sincronización y URLs prefirmadas. |
| [`semana_1/bucket_practice/03-activity.md`](./semana_1/bucket_practice/03-activity.md) | Guía paso a paso del laboratorio: *Ciclo de vida de un bucket en S3*. |
| [`semana_1/bucket_practice/evidence.md`](./semana_1/bucket_practice/evidence.md) | Documento formal de evidencias con trazas de terminal, capturas de pantalla de validación y preguntas de reflexión. |
| [`semana_1/bucket_practice/evidence-raw.txt`](./semana_1/bucket_practice/evidence-raw.txt) | Registro de salida directa de terminal con las pruebas ejecutadas. |
| [`semana_1/bucket_practice/solo-lectura.json`](./semana_1/bucket_practice/solo-lectura.json) | Definición de política IAM en formato JSON con permisos mínimos (`s3:GetObject`). |
| [`semana_1/bucket_practice/image_1.png`](./semana_1/bucket_practice/image_1.png) | Captura de validación: enlace prefirmado funcionando dentro del tiempo límite (< 60s). |
| [`semana_1/bucket_practice/image_2.png`](./semana_1/bucket_practice/image_2.png) | Captura de validación: error XML `AccessDenied` / `Request has expired` tras expirar el enlace (> 60s). |

#### 🛠️ Actividad Práctica: Ciclo de Vida de un Bucket en S3

Se implementó el ciclo de vida completo de un bucket privado `practica-danilo-3007` en la región `us-east-2`:

1. **Creación y Verificación de Región:**
   - Creación del bucket mediante `aws s3 mb s3://practica-danilo-3007`.
   - Consulta de ubicación con `aws s3api get-bucket-location`, confirmando `us-east-2`.

2. **Carga y Sincronización de Archivos:**
   - Creación de archivos de prueba (`notas.txt`, `tareas.txt`, `secreto.txt`).
   - Carga individual con `aws s3 cp` y carga en lote selectiva con `aws s3 sync . ... --include ...`.
   - Inspección con `aws s3 ls ... --recursive --human-readable --summarize` (3 objetos, 55 Bytes).

3. **Consulta, Descarga y Modificación de Objetos:**
   - Lectura directa en terminal sin descargar mediante `aws s3 cp s3://.../notas.txt -`.
   - Descarga de `secreto.txt` en carpeta local `descargas/`.
   - Renombramiento de `tareas.txt` a `pendientes.txt` dentro de S3 usando `aws s3 mv`.
   - Modificación local de `notas.txt` y resubida comprobando la sobrescritura directa sin versionado.

4. **Compartición Segura mediante URLs Prefirmadas (`presign`):**
   - Generación de enlace temporal con expiración de 60 segundos (`aws s3 presign ... --expires-in 60`).
   - **Validación dentro del tiempo:** Acceso exitoso al contenido desde el navegador web (`image_1.png`).
   - **Validación post-expiración:** Bloqueo automático con respuesta XML `AccessDenied` / `Request has expired` (`image_2.png`), demostrando que el bucket nunca requirió abrirse al público.

5. **Aislamiento y Menor Privilegio con Perfil IAM:**
   - Creación del usuario `lector-s3-danilo` y asignación de política en línea `leer-objetos` (`solo-lectura.json` limitando a `s3:GetObject` sobre el ARN del bucket con comodín `/*`).
   - Configuración de perfil CLI local (`--profile lector-s3`).
   - **Prueba permitida:** Lectura exitosa de `notas.txt`.
   - **Prueba denegada 1:** `aws s3 ls` falló con `AccessDenied` (requiere `s3:ListBucket`).
   - **Prueba denegada 2:** `aws s3 rm` falló con `AccessDenied` (requiere `s3:DeleteObject`).

6. **Protección y Limpieza Ordenada:**
   - Comprobación del error `BucketNotEmpty` al intentar borrar el bucket con objetos dentro.
   - Vaciado recursivo (`aws s3 rm ... --recursive`) y eliminación del bucket (`aws s3 rb`).
   - Eliminación ordenada de credenciales y usuario IAM (`delete-access-key` $\rightarrow$ `delete-user-policy` $\rightarrow$ `delete-user`).
   - Confirmación final de la cuenta completamente limpia en S3 e IAM.

#### 💡 Aprendizajes Clave
- **Namespace Global vs. Ubicación Regional:** Los nombres de los buckets son globales únicos en todo AWS (afectan registros DNS), aunque se alojan físicamente en una región elegida.
- **Diferenciación de Permisos en S3:** `s3:GetObject` (nivel objeto `/*`) y `s3:ListBucket` (nivel bucket) son acciones totalmente independientes.
- **Uso de Enlaces Prefirmados:** Permiten compartir temporalmente contenido privado sin alterar las políticas de seguridad ni abrir el bucket a Internet.
- **Modelo de Responsabilidad Compartida:** La seguridad del bucket y evitar filtraciones de datos públicos es responsabilidad del cliente ("seguridad EN la nube").

---

### 🔹 Actividad 3: Roles de IAM y Perfiles de Instancia para EC2 (`role_practice`)

#### 🎯 Objetivos de la Actividad
- Comprender la arquitectura de **Roles de IAM**, su función en la delegación de permisos y el uso de credenciales temporales generadas por **STS (Security Token Service)**.
- Diferenciar con precisión la **política de confianza** (*Trust Policy* - ¿quién puede asumir el rol?) de la **política de permisos** (*Permissions Policy* - ¿qué acciones se autorizan sobre qué recursos?).
- Configurar un **perfil de instancia** (*Instance Profile*) mediante AWS CLI para conectar el rol a una instancia EC2 sin almacenar llaves ni credenciales locales fijas.
- Aplicar el **principio de menor privilegio** definiendo una política en línea que restrinja exclusivamente la lectura de objetos (`s3:GetObject`) en el bucket de reportes.
- Evaluar las reglas de seguridad e integridad referencial de IAM (obligatoriedad de política de confianza y error `DeleteConflict` ante intentos de eliminación indebida).

#### 📁 Estructura y Archivos

| Archivo | Descripción |
|---|---|
| [`semana_1/role_practice/01-info.md`](./semana_1/role_practice/01-info.md) | Fundamentos conceptuales de roles, perfiles de instancia, políticas de confianza vs. permisos y STS. |
| [`semana_1/role_practice/02-command.md`](./semana_1/role_practice/02-command.md) | Cheat sheet de comandos AWS CLI para gestión de roles, perfiles de instancia y políticas. |
| [`semana_1/role_practice/03-activity.md`](./semana_1/role_practice/03-activity.md) | Guía paso a paso del laboratorio: *Un rol para tu instancia EC2*. |
| [`semana_1/role_practice/confianza.json`](./semana_1/role_practice/confianza.json) | Política de confianza que autoriza al servicio EC2 (`ec2.amazonaws.com`) para asumir el rol (`sts:AssumeRole`). |
| [`semana_1/role_practice/permisos.json`](./semana_1/role_practice/permisos.json) | Política de permisos en línea con privilegios mínimos (`s3:GetObject`) sobre el bucket de reportes. |
| [`semana_1/role_practice/reporte.txt`](./semana_1/role_practice/reporte.txt) | Archivo de prueba creado y alojado en el bucket S3 para comprobación de lectura. |
| [`semana_1/role_practice/evidence-raw.txt`](./semana_1/role_practice/evidence-raw.txt) | Registro crudo de terminal con la secuencia completa de comandos ejecutados. |
| [`semana_1/role_practice/evidence.md`](./semana_1/role_practice/evidence.md) | Informe formal de evidencias del laboratorio con trazas de ejecución CLI y respuestas a preguntas de reflexión. |

#### 🛠️ Actividad Práctica: Un Rol para tu Instancia EC2

En este laboratorio se configuró un rol de menor privilegio y su correspondiente perfil de instancia para preparar el acceso seguro desde EC2:

1. **Preparación del Recurso:**
   - Creación del bucket privado `reportes-danilo-3007` y subida del objeto `reporte.txt`.
2. **Definición de Políticas:**
   - Elaboración de `confianza.json` permitiendo únicamente a `ec2.amazonaws.com` asumir el rol.
   - Elaboración de `permisos.json` permitiendo únicamente `s3:GetObject` sobre `arn:aws:s3:::reportes-danilo-3007/*`.
3. **Creación del Rol y Validación de Requisitos:**
   - Intento fallido de creación sin política de confianza (comprobando que AWS rechaza la operación).
   - Creación exitosa del rol `ec2-lector-s3` adjuntando la política de confianza requerida.
4. **Asignación de Permisos y Metadatos:**
   - Creación y adjunción de la política en línea `leer-reportes`.
   - Incorporación de descripción del rol y etiquetas (`semana=1`).
5. **Enlace con Perfil de Instancia:**
   - Creación del perfil de instancia `ec2-lector-s3` y vinculación del rol dentro del perfil.
6. **Comprobación de Integridad Referencial:**
   - Intento de eliminación del rol mediante `aws iam delete-role`, provocando y documentando el error `DeleteConflict` al encontrarse vinculado al perfil de instancia.

#### 💡 Aprendizajes Clave
- **Roles vs. Usuarios:** Los roles entregan credenciales temporales que se renuevan de forma automática y transparente, eliminando el riesgo de llaves permanentes expuestas en servidores o repositorios.
- **Separación de Confianza y Permisos:** Un rol requiere ineludiblemente una política de confianza (*quién*) y una política de permisos (*qué*).
- **Perfiles de Instancia en la CLI:** La consola gráfica de AWS asocia el perfil de instancia automáticamente al crear un rol para EC2, pero desde la CLI es un paso manual obligatorio.
- **Integridad y Orden de Limpieza:** AWS protege los recursos impidiendo el borrado de un rol si aún está referenciado por un perfil de instancia.

---

### 🔹 Actividad 4: Servidores Virtuales en Amazon EC2 (`EC2_practice`)

#### 🎯 Objetivos de la Actividad
- Lanzar y administrar servidores virtuales en **Amazon EC2 (Elastic Compute Cloud)** mediante **AWS CLI v2**.
- Automatizar la instalación y puesta en marcha de un servidor web Apache (`httpd`) mediante scripts de aprovisionamiento en arranque (**User Data: `inicio.sh`**).
- Configurar y aplicar reglas de firewall perimetral con **Security Groups** siguiendo el principio de menor exposición (puerto 80 abierto al público y puerto 22 SSH restringido a la IP del administrador `/32`).
- Gestionar el acceso remoto seguro vía **SSH** mediante pares de llaves criptográficas (`clave-danilo`).
- Conectar y comprobar el consumo de credenciales temporales generadas por STS a través del **perfil de instancia (`ec2-lector-s3`)**, leyendo objetos en S3 sin configurar llaves fijas locales ni usar `aws configure`.
- Analizar los estados de ciclo de vida de la instancia (`running`, `stopped`, `terminated`) y el comportamiento dinámico de las **direcciones IPv4 públicas**.
- Realizar la terminación definitiva y limpieza ordenada de todos los recursos aprovisionados a lo largo de las prácticas.

#### 📁 Estructura y Archivos

| Archivo | Descripción |
|---|---|
| [`semana_1/EC2_practice/01-info.md`](./semana_1/EC2_practice/01-info.md) | Fundamentos conceptuales de Amazon EC2: instancias, AMIs, tipos de instancia, EBS, Security Groups, VPCs y estados de ciclo de vida. |
| [`semana_1/EC2_practice/02-command.md`](./semana_1/EC2_practice/02-command.md) | Cheat sheet y guía de referencia rápida de comandos AWS CLI para EC2, pares de llaves y Security Groups. |
| [`semana_1/EC2_practice/03-activity.md`](./semana_1/EC2_practice/03-activity.md) | Guía paso a paso del laboratorio: *Tu primer servidor en EC2*. |
| [`semana_1/EC2_practice/evidence.md`](./semana_1/EC2_practice/evidence.md) | Informe formal de evidencias del laboratorio con trazas de ejecución CLI y respuestas a preguntas de reflexión. |
| [`semana_1/EC2_practice/evidence-raw.txt`](./semana_1/EC2_practice/evidence-raw.txt) | Registro crudo de terminal con la secuencia completa de comandos ejecutados. |
| [`semana_1/EC2_practice/practica-ec2/inicio.sh`](./semana_1/EC2_practice/practica-ec2/inicio.sh) | Script de User Data para automatizar la instalación de Apache (`httpd`) y publicación del sitio web de bienvenida. |

#### 🛠️ Actividad Práctica: Tu Primer Servidor en EC2

En este laboratorio se integraron todos los conceptos previos (IAM, S3, Roles y Redes) en un despliegue completo de cómputo en la nube:

1. **Reconocimiento y Validación de Prerequisitos:**
   - Verificación de identidad administradora (`admin-danilo`), región (`us-east-2`) y confirmación del perfil de instancia `ec2-lector-s3`.
   - Consulta dinámica del AMI ID oficial de Amazon Linux 2023 (`ami-01e4a7224fcbfe81f`) mediante SSM Parameter Store y localización de la VPC por defecto (`vpc-098234409df616272`).

2. **Par de Llaves Criptográficas:**
   - Creación del par de llaves `clave-danilo`, almacenamiento local de la clave privada `.pem` y restricción estricta de permisos (`chmod 400`).

3. **Aislamiento Perimetral con Security Groups:**
   - Creación del grupo `web-danilo` (`sg-022fe3a75acfb92c8`).
   - Apertura del puerto 80 (HTTP) a nivel global (`0.0.0.0/0`) y restricción del puerto 22 (SSH) exclusivamente a la IP pública del operador (`/32`).

4. **Aprovisionamiento y Lanzamiento de la Instancia:**
   - Creación del script de User Data `inicio.sh` para instalación y arranque automático del servidor web Apache.
   - Despliegue de la instancia `t3.micro` con `run-instances`, inyectando AMI, llaves, Security Group, script de inicio y el perfil `ec2-lector-s3`.
   - Obtención de la IP pública asignada (`77.***.**.***`).

5. **Acceso SSH y Validación del Rol IAM:**
   - Conexión remota interactiva por SSH y comprobación del servidor web respondiendo localmente (`curl http://localhost`).
   - Comprobación de identidad (`aws sts get-caller-identity`), validando que la máquina opera con el rol `ec2-lector-s3` sin requerir credenciales locales ni `aws configure`.
   - **Acción permitida:** Lectura exitosa de `reporte.txt` desde el bucket S3 (`reportes-danilo-3007`).
   - **Acciones denegadas:** Bloqueo automático con `AccessDenied` al intentar listar el bucket (`s3:ListBucket`) o listar usuarios de IAM (`iam:ListUsers`), validando el aislamiento estricto de permisos.

6. **Comprobación de Ciclo de Vida e IP Dinámica:**
   - Detención (`stop-instances`) y reencendido (`start-instances`) de la máquina virtual, evidenciando el cambio de IP pública a `18.224.25.145`.

7. **Limpieza Integral de Recursos:**
   - Terminación de la instancia (`terminate-instances`), borrado del Security Group y eliminación del par de llaves en AWS.
   - Limpieza ordenada de recursos remanentes de roles y S3: desvinculación y borrado del perfil de instancia, eliminación de la política en línea, borrado del rol y vaciado forzoso del bucket S3.

#### 💡 Aprendizajes Clave
- **Automatización mediante User Data:** Permite configurar instancias desde el primer arranque sin intervención manual posterior.
- **Seguridad en Redes con Security Groups:** El principio de menor privilegio aplica también a redes; puertos de administración crítica como el 22 jamás deben exponerse a `0.0.0.0/0`.
- **Seguridad de Cómputo sin Secretos Estáticos:** Los Instance Profiles combinados con IMDS y STS eliminan la necesidad de guardar credenciales fijas de larga duración dentro de servidores o repositorios.
- **Naturaleza Efímera de las IPs Públicas:** En AWS, las IPs públicas básicas cambian al apagar y encender la instancia, haciendo indispensable el uso de Elastic IPs o balanceadores de carga (ALB) en entornos productivos.
- **Modelo de Responsabilidad Compartida en IaaS:** AWS garantiza la seguridad física y del hipervisor, pero la seguridad del sistema operativo, parches, reglas de firewall y código es 100% responsabilidad del usuario.

---

## 💻 Requisitos y Entorno de Trabajo

- **AWS CLI v2** configurado con credenciales de laboratorio.
- Shell compatible (Bash / Zsh) en Linux / macOS / WSL.
- Gestión de credenciales bajo el estándar de menor privilegio (nunca versionar llaves secretas en Git).

---

## 👨‍💻 Autor

- GitHub: **[Danilo-Doria](https://github.com/Danilo-Doria)**
- LinkedIn: **[Danilo Doria Diaz](https://www.linkedin.com/in/danilo-doria-diaz)**
- Mail: **danilodoria519@gmail.com**
