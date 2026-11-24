# AWS Training - Bitácora de Aprendizaje Semanal

Repositorio dedicado a documentar el progreso práctico, conceptual y de laboratorio en **Amazon Web Services (AWS)** semana a semana, utilizando principalmente **AWS CLI v2** y la consola de administración bajo las mejores prácticas de arquitectura y seguridad en la nube.

---

## 📌 Índice de Semanas

- [Semana 1: Fundamentos de IAM y AWS CLI](#-semana-1-fundamentos-de-iam-y-aws-cli)
- [Semana 2: Almacenamiento en la Nube con Amazon S3 y Menor Privilegio](#-semana-2-almacenamiento-en-la-nube-con-amazon-s3-y-menor-privilegio)

---

## 🚀 Semana 1: Fundamentos de IAM y AWS CLI

### 🎯 Objetivos de la Semana
- Comprender la arquitectura de seguridad, autenticación y autorización en AWS mediante **IAM (Identity and Access Management)**.
- Dominar el uso de **AWS CLI v2** para la gestión de identidades, perfiles, credenciales y políticas.
- Aplicar de forma práctica el **principio de menor privilegio** (*Least Privilege*).
- Implementar y comprobar el ciclo de vida completo de un usuario IAM (creación, metadatos/etiquetas, asociación de políticas, pruebas de autorización y eliminación segura).

### 📁 Estructura y Archivos

| Archivo | Descripción |
|---|---|
| [`semana_1/01-info.md`](semana_1/01-info.md) | Fundamentos conceptuales de IAM: usuarios, grupos, roles, políticas JSON, perfiles de instancia y reglas de evaluación. |
| [`semana_1/02-command.md`](semana_1/02-command.md) | Cheat sheet y guía de referencia rápida de comandos AWS CLI para IAM. |
| [`semana_1/03-activity.md`](semana_1/03-activity.md) | Guía paso a paso del laboratorio: *Ciclo de vida de un usuario IAM*. |
| [`semana_1/evidence-week-1.md`](semana_1/evidence-week-1.md) | Informe de evidencias del laboratorio con trazas de ejecución CLI y respuestas a preguntas de reflexión. |
| [`semana_1/evidence.txt`](semana_1/evidence.txt) | Registro crudo de terminal con los comandos y respuestas obtenidas. |

### 🛠️ Actividad Práctica: Ciclo de Vida de un Usuario IAM

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

### 💡 Aprendizajes Clave
- **Deny por defecto:** Un usuario nuevo no puede realizar ninguna acción en la cuenta hasta recibir una política explícita.
- **Precedencia de permisos:** Un `Deny` explícito siempre invalida cualquier `Allow`.
- **Integridad referencial en IAM:** AWS no permite eliminar usuarios con llaves o políticas activas para evitar credenciales huérfanas.
- **Gestión segura de credenciales:** El uso de perfiles nombrados (`--profile`) previene la sobreescritura accidental de credenciales administradoras locales.

---

## 🚀 Semana 2: Almacenamiento en la Nube con Amazon S3 y Menor Privilegio

### 🎯 Objetivos de la Semana
- Comprender la arquitectura de almacenamiento de objetos con **Amazon S3 (Simple Storage Service)**.
- Dominar el uso de comandos de alto nivel (`aws s3`) y bajo nivel (`aws s3api`) en **AWS CLI v2**.
- Administrar el ciclo de vida de objetos en S3: subida, descarga, inspección, sincronización por lotes (`sync`), renombramiento y eliminación.
- Implementar mecanismos seguros para compartir archivos privados mediante **enlaces temporales prefirmados (`presign`)** sin exponer el bucket al público.
- Aplicar el **principio de menor privilegio** en S3 creando políticas en línea personalizadas en IAM (`s3:GetObject` vs `s3:ListBucket`).
- Conocer la regla de integridad y protección de borrado de buckets no vacíos (*BucketNotEmpty*).

### 📁 Estructura y Archivos

| Archivo | Descripción |
|---|---|
| [`semana_2/01-info.md`](semana_2/01-info.md) | Fundamentos de Amazon S3: buckets, objetos, claves, prefijos, reglas de nomenclatura, características y analogías. |
| [`semana_2/02-command.md`](semana_2/02-command.md) | Cheat sheet de comandos CLI (`aws s3` y `aws s3api`) para buckets, objetos, sincronización y URLs prefirmadas. |
| [`semana_2/03-activity.md`](semana_2/03-activity.md) | Guía paso a paso del laboratorio: *Ciclo de vida de un bucket en S3*. |
| [`semana_2/evidence-week-2.md`](semana_2/evidence-week-2.md) | Documento formal de evidencias con trazas de terminal, capturas de pantalla de validación y preguntas de reflexión. |
| [`semana_2/evidence-raw.txt`](semana_2/evidence-raw.txt) | Registro de salida directa de terminal con las pruebas ejecutadas. |
| [`semana_2/solo-lectura.json`](semana_2/solo-lectura.json) | Definición de política IAM en formato JSON con permisos mínimos (`s3:GetObject`). |
| [`semana_2/image_1.png`](semana_2/image_1.png) | Captura de validación: enlace prefirmado funcionando dentro del tiempo límite (< 60s). |
| [`semana_2/image_2.png`](semana_2/image_2.png) | Captura de validación: error XML `AccessDenied` / `Request has expired` tras expirar el enlace (> 60s). |

### 🛠️ Actividad Práctica: Ciclo de Vida de un Bucket en S3

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

### 💡 Aprendizajes Clave
- **Namespace Global vs. Ubicación Regional:** Los nombres de los buckets son globales únicos en todo AWS (afectan registros DNS), aunque se alojan físicamente en una región elegida.
- **Diferenciación de Permisos en S3:** `s3:GetObject` (nivel objeto `/*`) y `s3:ListBucket` (nivel bucket) son acciones totalmente independientes.
- **Uso de Enlaces Prefirmados:** Permiten compartir temporalmente contenido privado sin alterar las políticas de seguridad ni abrir el bucket a Internet.
- **Modelo de Responsabilidad Compartida:** La seguridad del bucket y evitar filtraciones de datos públicos es responsabilidad del cliente ("seguridad EN la nube").

---

## 💻 Requisitos y Entorno de Trabajo

- **AWS CLI v2** configurado con credenciales de laboratorio.
- Shell compatible (Bash / Zsh) en Linux / macOS / WSL.
- Gestión de credenciales bajo el estándar de menor privilegio (nunca versionar llaves secretas en Git).

---

## 👨‍💻 Author

- GitHub: **[Danilo-Doria](https://github.com/Danilo-Doria)**
- LinkedIn: **[Danilo Doria Diaz](https://www.linkedin.com/in/danilo-doria-diaz)**
- Mail: **danilodoria519@gmail.com**
