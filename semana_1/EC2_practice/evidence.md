# Actividad: tu primer servidor en EC2

Práctica integral de despliegue de un servidor virtual en Amazon EC2 (Elastic Compute Cloud), configuración de red y reglas de filtrado con Security Groups, aprovisionamiento automatizado mediante scripts de User Data (`inicio.sh`), conexión remota segura vía SSH, consumo de roles de IAM mediante Instance Profiles (principio de menor privilegio sin credenciales estáticas en disco), análisis de ciclo de vida/cambio de IP pública y eliminación ordenada de toda la infraestructura mediante AWS CLI v2.

---

## Evidencia de ejecución

### Parte 1: Reconocer el terreno y validar prerequisitos (Pasos 1 a 4)
Verificación de identidad administradora (`admin-danilo`), confirmación de región de trabajo (`us-east-2`), comprobación de existencia del perfil de instancia `ec2-lector-s3` (generado en la actividad anterior), consulta dinámica del ID de la AMI de Amazon Linux 2023 mediante SSM Parameter Store y obtención de la VPC por defecto:

```bash
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws sts get-caller-identity
{
    "UserId": "AIDA*****************",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/admin-danilo"
}

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws configure get region
us-east-2

# Confirmación de existencia del perfil de instancia y rol ec2-lector-s3
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws iam get-instance-profile --instance-profile-name ec2-lector-s3
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

# Obtención del AMI ID oficial de Amazon Linux 2023 en us-east-2
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ssm get-parameters --names /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 --query "Parameters[0].Value" --output text
ami-01e4a7224fcbfe81f

# Obtención del VPC ID por defecto
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 describe-vpcs --filters Name=isDefault,Values=true --query "Vpcs[].VpcId" --output text
vpc-098234409df616272
```

### Parte 2: Par de llaves para acceso SSH (Pasos 5 a 7)
Creación del par de llaves `clave-danilo`, guardado en archivo local `.pem`, ajuste de permisos de seguridad estrictos (`chmod 400`) y comprobación en AWS:

```bash
# Creación del par de llaves y almacenamiento local de la clave privada
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 create-key-pair --key-name clave-danilo --query "KeyMaterial" --output text > clave-danilo.pem

# Asignación de permisos de solo lectura para el propietario
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ chmod 400 clave-danilo.pem

# Confirmación del par de llaves registrado en AWS
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 describe-key-pairs --query "KeyPairs[].KeyName" --output table
------------------
|DescribeKeyPairs|
+----------------+
|  clave-danilo  |
+----------------+
```

*(Nota de seguridad: Por mejores prácticas, el archivo `clave-danilo.pem` fue eliminado localmente al concluir el laboratorio y nunca es versionado en Git).*

### Parte 3: Configuración del Security Group (Pasos 8 a 12)
Creación del grupo de seguridad `web-danilo` (`sg-022fe3a75acfb92c8`), detección de IP pública propia, apertura del puerto 80 para tráfico web general (`0.0.0.0/0`) y restricción del puerto 22 exclusivamente a la IP pública del operador con máscara `/32`:

```bash
# Paso 8: Creación del Security Group
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 create-security-group --group-name web-danilo --description "HTTP abierto y SSH solo desde mi IP" --query "GroupId" --output text
sg-022fe3a75acfb92c8

# Paso 9: Consulta de IP pública local
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ curl -s https://checkip.amazonaws.com
190.**.***.**

# Paso 10: Apertura de puerto 80 (HTTP) a todo internet
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 authorize-security-group-ingress --group-id sg-022fe3a75acfb92c8 --protocol tcp --port 80 --cidr 0.0.0.0/0
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-019de0e45d5474fc4",
            "GroupId": "sg-022fe3a75acfb92c8",
            "GroupOwnerId": "123456789012",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "CidrIpv4": "0.0.0.0/0",
            "SecurityGroupRuleArn": "arn:aws:ec2:us-east-2:123456789012:security-group-rule/sgr-019de0e45d5474fc4"
        }
    ]
}

# Paso 11: Apertura de puerto 22 (SSH) limitado exclusivamente a la IP pública
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 authorize-security-group-ingress --group-id sg-022fe3a75acfb92c8 --protocol tcp --port 22 --cidr 190.**.***.**/32
{
    "Return": true,
    "SecurityGroupRules": [
        {
            "SecurityGroupRuleId": "sgr-08460a3d4ef565282",
            "GroupId": "sg-022fe3a75acfb92c8",
            "GroupOwnerId": "123456789012",
            "IsEgress": false,
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "CidrIpv4": "190.**.***.**/32",
            "SecurityGroupRuleArn": "arn:aws:ec2:us-east-2:123456789012:security-group-rule/sgr-08460a3d4ef565282"
        }
    ]
}

# Paso 12: Inspección y validación de las reglas de entrada del Security Group
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 describe-security-groups --group-ids sg-022fe3a75acfb92c8 --query "SecurityGroups[].IpPermissions[].[FromPort,IpRanges[0].CidrIp]" --output table
----------------------------
|  DescribeSecurityGroups  |
+----+---------------------+
|  80|  0.0.0.0/0          |
|  22|  190.**.***.**/32   |
+----+---------------------+
```

### Parte 4: User Data y lanzamiento de la instancia EC2 (Pasos 13 a 16)
Creación del script de User Data `inicio.sh`, lanzamiento de la instancia `t3.micro` con todos los recursos vinculados, espera activa de arranque (`wait instance-running`) y consulta de su dirección IP pública asignada:

```bash
# Paso 13: Creación de script de inicio (User Data)
cat > inicio.sh << 'EOF'
#!/bin/bash
dnf install -y httpd
systemctl enable --now httpd
echo "<h1>Hola desde mi instancia EC2</h1>" > /var/www/html/index.html
EOF

# Paso 14: Lanzamiento de la instancia EC2
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 run-instances \
    --image-id ami-01e4a7224fcbfe81f \
    --instance-type t3.micro \
    --key-name clave-danilo \
    --security-group-ids sg-022fe3a75acfb92c8 \
    --iam-instance-profile Name=ec2-lector-s3 \
    --user-data file://inicio.sh \
    --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=web-danilo}]" \
    --query "Instances[0].InstanceId" \
    --output text
i-*****************

# Paso 15: Espera activa a que el estado sea running
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 wait instance-running --instance-ids i-*****************

# Paso 16: Consulta de dirección IP pública asignada
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 describe-instances --instance-ids i-***************** --query "Reservations[0].Instances[0].PublicIpAddress" --output text
77.***.**.***
```

### Parte 5 y 6: Conexión SSH, servidor web y validación del Rol IAM (Pasos 17 a 24)
Conexión remota por SSH a la instancia EC2, validación del servidor web Apache (`httpd`) respondiendo localmente, verificación de identidad IAM del rol asumido sin necesidad de credenciales locales (`aws configure`), lectura exitosa del bucket (acción permitida) y pruebas de aislamiento de permisos denegados:

```bash
# Paso 19: Conexión SSH a la instancia
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ ssh -i clave-danilo.pem ec2-user@77.***.**.***
The authenticity of host '77.***.**.*** (77.***.**.***)' can't be established.
ED25519 key fingerprint is SHA256:CQE3QFlG0OowcFGZavedjKhAIfdrSjbygN05NNiDuHc.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '77.***.**.***' (ED25519) to the list of known hosts.
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'

# Pasos 17 y 18: Verificación del servidor web Apache instalado mediante User Data
[ec2-user@ip-172-**-**-** ~]$ curl http://localhost
<h1>Hola desde mi instancia EC2</h1>

# Paso 20: Comprobación de identidad dentro de la instancia (asume el rol automáticamente)
[ec2-user@ip-172-**-**-** ~]$ aws sts get-caller-identity
{
    "UserId": "AROA*****************:i-*****************",
    "Account": "123456789012",
    "Arn": "arn:aws:sts::123456789012:assumed-role/ec2-lector-s3/i-*****************"
}

# Paso 21: Acción permitida (lectura de objeto en bucket reportes-danilo-3007)
[ec2-user@ip-172-**-**-** ~]$ aws s3 cp s3://reportes-danilo-3007/reporte.txt -
Este es el contenido del reporte

# Paso 22: Acción denegada (listar el contenido del bucket)
[ec2-user@ip-172-**-**-** ~]$ aws s3 ls s3://reportes-danilo-3007/

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListObjectsV2 operation: User: arn:aws:sts::123456789012:assumed-role/ec2-lector-s3/i-***************** is not authorized to perform: s3:ListBucket on resource: "arn:aws:s3:::reportes-danilo-3007" because no identity-based policy allows the s3:ListBucket action

# Paso 23: Acción denegada (listar usuarios de IAM)
[ec2-user@ip-172-**-**-** ~]$ aws iam list-users

aws: [ERROR]: An error occurred (AccessDenied) when calling the ListUsers operation: User: arn:aws:sts::123456789012:assumed-role/ec2-lector-s3/i-***************** is not authorized to perform: iam:ListUsers on resource: arn:aws:iam::123456789012:user/ because no identity-based policy allows the iam:ListUsers action.
```

*(Paso 24: Se finalizó la sesión remota con `exit`).*

### Parte 7: Estados de la instancia y cambio de IP pública (Pasos 25 a 27)
Detención de la instancia (`stop-instances`), posterior reencendido (`start-instances`) y comparación de la dirección IP pública resultante frente a la inicial del Paso 16:

```bash
# Paso 25: Apagado de la instancia
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 stop-instances --instance-ids i-*****************
{
    "StoppingInstances": [
        {
            "InstanceId": "i-*****************",
            "CurrentState": {
                "Code": 64,
                "Name": "stopping"
            },
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}

# Paso 26: Encendido de la instancia y espera a estado running
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 start-instances --instance-ids i-*****************
{
    "StartingInstances": [
        {
            "InstanceId": "i-*****************",
            "CurrentState": {
                "Code": 0,
                "Name": "pending"
            },
            "PreviousState": {
                "Code": 80,
                "Name": "stopped"
            }
        }
    ]
}

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 wait instance-running --instance-ids i-*****************

# Paso 27: Consulta de nueva IP pública tras el reinicio
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 describe-instances --instance-ids i-***************** --query "Reservations[0].Instances[0].PublicIpAddress" --output text
18.224.25.145
```
> **Observación del Paso 27:** La IP pública cambió de `77.***.**.***` a `18.224.25.145`, comprobando el comportamiento dinámico y efímero de las direcciones IPv4 públicas asignadas por defecto en AWS EC2.

### Parte 8: Eliminación y limpieza integral de infraestructura (Pasos 28 a 32)
Terminación de la instancia EC2, borrado de Security Group, borrado de Key Pair, confirmación de instancia terminada y limpieza en orden de todos los recursos generados en los laboratorios de Roles y S3:

```bash
# Paso 28: Terminación de la instancia
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 terminate-instances --instance-ids i-*****************
{
    "TerminatingInstances": [
        {
            "InstanceId": "i-*****************",
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 wait instance-terminated --instance-ids i-*****************

# Paso 29: Eliminación del Security Group
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 delete-security-group --group-id sg-022fe3a75acfb92c8
{
    "Return": true,
    "GroupId": "sg-022fe3a75acfb92c8"
}

# Paso 30: Eliminación del Key Pair en AWS
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 delete-key-pair --key-name clave-danilo
{
    "Return": true,
    "KeyPairId": "key-0307822dc349f5c03"
}

# Paso 31: Verificación de instancias (estado terminated y sin IP pública)
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws ec2 describe-instances --query "Reservations[].Instances[].[InstanceId,State.Name,InstanceType,PublicIpAddress]" --output table
-----------------------------------------------------------
|                    DescribeInstances                    |
+----------------------+-------------+-----------+--------+
|  i-***************** |  terminated |  t3.micro |  None  |
+----------------------+-------------+-----------+--------+

# Paso 32: Limpieza de los recursos creados en la actividad de Roles y S3
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws iam remove-role-from-instance-profile --instance-profile-name ec2-lector-s3 --role-name ec2-lector-s3

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws iam delete-instance-profile --instance-profile-name ec2-lector-s3

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws iam delete-role-policy --role-name ec2-lector-s3 --policy-name leer-reportes

Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws iam delete-role --role-name ec2-lector-s3

# Vaciado forzoso y eliminación del bucket S3
Coder@rwadmin-90533010005:~/Vídeos/AWS/actividades/AWS-training/semana_1/EC2_practice/practica-ec2$ aws s3 rb s3://reportes-danilo-3007 --force
remove_bucket: reportes-danilo-3007
```

---

## Preguntas de reflexión

1. **En el paso 20, la instancia responde con un rol y no con un usuario. ¿De dónde sacó esas credenciales si nunca corriste `aws configure`?**  
   Las credenciales fueron entregadas automáticamente por el servicio **IMDS (Instance Metadata Service)** de AWS, accesible internamente desde la dirección IP de enlace local `169.254.169.254`. Al asociar el perfil de instancia `ec2-lector-s3` durante el lanzamiento de la máquina, AWS gestiona la autenticación con **AWS STS (Security Token Service)** en segundo plano. STS genera credenciales temporales efímeras (`AccessKeyId`, `SecretAccessKey` y `SessionToken`) vinculadas al rol asumido (`assumed-role/ec2-lector-s3/i-...`). Cuando el AWS CLI o SDK ejecuta un comando dentro de la instancia, detecta la ausencia de credenciales estáticas locales en `~/.aws/credentials` y consulta directamente a IMDS, renovándolas de forma transparente antes de su vencimiento.

2. **¿Por qué el puerto 80 puede estar abierto a todo internet y el 22 no?**  
   - **Puerto 80 (HTTP):** Es el puerto estándar para servir contenido web no cifrado al público. Su objetivo es que cualquier usuario de Internet pueda consultar el sitio o aplicación web sin restricciones de red (`0.0.0.0/0`).
   - **Puerto 22 (SSH):** Es un puerto de administración crítica que otorga una consola remota con privilegios elevados (`ec2-user` con capacidades de `sudo`). Dejarlo abierto a todo Internet (`0.0.0.0/0`) expone el servidor a escaneos automáticos de vulnerabilidades, ataques de fuerza bruta y ataques de denegación de servicio. Limitarlo con `/32` a la IP pública del administrador garantiza que únicamente su equipo pueda iniciar la negociación criptográfica de acceso.

3. **La instancia pudo leer `reporte.txt` pero no listar el bucket. ¿Dónde se decidió eso?**  
   Se decidió en la **política de permisos en línea (`leer-reportes`)** que se adjuntó al rol `ec2-lector-s3` (definida en el archivo `permisos.json`). Dicha política concedió explícitamente la acción `"Action": ["s3:GetObject"]` sobre el recurso `"arn:aws:s3:::reportes-danilo-3007/*"`. En AWS IAM rige la regla de **Denegación Implícita por defecto**: al no haberse declarado la acción `s3:ListBucket` sobre el ARN del bucket (`arn:aws:s3:::reportes-danilo-3007`), cualquier solicitud para listar su contenido resulta denegada de inmediato con `AccessDenied`.

4. **¿Por qué cambió la IP pública al apagar y encender? ¿Qué problema traería eso en un sitio web real?**  
   Las direcciones IPv4 públicas asignadas por defecto a las instancias en una VPC provienen de un grupo dinámico compartido de AWS (*pool* de direcciones). Al apagar la instancia (`stopped`), AWS libera esa IP pública para que otros clientes puedan usarla (evitando el desperdicio de direcciones IPv4 escasas). Al volver a encenderla (`start-instances`), AWS asigna una nueva IP disponible del grupo.  
   **Problema en un entorno real:** En producción, los nombres de dominio (DNS) apuntan a direcciones IP específicas mediante registros tipo `A`. Si la IP cambia cada vez que el servidor se detiene por mantenimiento o reinicio, todos los clientes y visitantes perderían el acceso hasta que se actualicen manualmente los registros DNS (proceso que además sufre retrasos por propagación de caché TTL). Para solucionar esto, en AWS se utilizan **Elastic IPs (IP estática reservada)** o se ubica la instancia detrás de un **Application Load Balancer (ALB)** con AWS Route 53.

5. **EC2 es infraestructura como servicio (IaaS). En esta práctica, ¿qué fue responsabilidad tuya y qué fue responsabilidad de AWS?**  
   Bajo el **Modelo de Responsabilidad Compartida**:
   - **Responsabilidad de AWS (Seguridad DE la Nube):** Infraestructura física de los centros de datos, servidores de cómputo, discos de almacenamiento EBS, cableado de red, suministro eléctrico ininterrumpido, aislamiento de hardware y el mantenimiento del hipervisor (Nitro/KVM).
   - **Responsabilidad del Usuario (Seguridad EN la Nube):** Elección y configuración del sistema operativo (Amazon Linux 2023), instalación y configuración del software del servidor web (`httpd`), redacción del script User Data (`inicio.sh`), definición estricta de las reglas del firewall (Security Group restringiendo SSH a `/32`), custodia y permisos del archivo de llave privada (`chmod 400`), aplicación del principio de menor privilegio en las políticas IAM asignadas y gestión del ciclo de vida/apagado de recursos para optimizar costos.

6. **Para tener este mismo servidor en una oficina, sin la nube, ¿qué habrías tenido que comprar y configurar?**  
   - **Hardware y Equipamiento Físico:** Compra de un servidor físico o torre de cómputo (con procesador, memoria RAM y discos en RAID), router empresarial, switch de red, cables estructurados Cat6/Fibra, sistema de alimentación ininterrumpida (UPS) para contingencias eléctricas y aire acondicionado para refrigeración del cuarto de servidores.
   - **Servicios de Telecomunicaciones:** Contratación de un enlace de Internet comercial con IP pública fija provista por un ISP (o configuración de Dynamic DNS empresarial).
   - **Configuración de Red y Seguridad:** Instalación del sistema operativo desde un medio de arranque USB/ISO, particionamiento de discos, apertura y redirección de puertos (NAT/Port Forwarding) en el router o firewall perimetral (puertos 80 y 22), configuración de `iptables`/`firewalld`, endurecimiento de SSH y gestión manual de copias de seguridad locales y remotas. Todo este proceso requiere semanas de compras y configuración frente a los minutos que tomó desplegarlo con la CLI de AWS.
