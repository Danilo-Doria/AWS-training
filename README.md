# AWS Training - Bitácora de Aprendizaje Semanal

Repositorio dedicado a documentar el progreso práctico, conceptual y de laboratorio en **Amazon Web Services (AWS)** semana a semana, utilizando principalmente **AWS CLI v2** y la consola de administración bajo las mejores prácticas de arquitectura y seguridad en la nube.

---

## 📌 Índice de Semanas

- [Semana 1: Fundamentos de IAM y AWS CLI](#-semana-1-fundamentos-de-iam-y-aws-cli)

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

## 💻 Requisitos y Entorno de Trabajo

- **AWS CLI v2** configurado con credenciales de laboratorio.
- Shell compatible (Bash / Zsh) en Linux / macOS / WSL.
- Gestión de credenciales bajo el estándar de menor privilegio (nunca versionar llaves secretas en Git).

---

## 👨‍💻 Author

- GitHub: **[Danilo-Doria](https://github.com/Danilo-Doria)**
- LinkedIn: **[Danilo Doria Diaz](https://www.linkedin.com/in/danilo-doria-diaz)**
- Mail: **danilodoria519@gmail.com**