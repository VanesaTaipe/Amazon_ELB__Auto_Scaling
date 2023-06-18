#  Auto Scaling
<BR>>
**__Parte 1: escalar hacia arriba__**<BR>
<BR>1. **Cambia al directorio donde guarda el archivo de script de instalación de apache:**
 
<BR> Inicie una instancia de la siguiente manera.

```
aws autoscaling create-launch-configuration --launch-configuration-name taipe-lc --image-id ami-d9a98cb0 --instance-type t1.micro --key-name taipe-key --security-groups taipe --user-data file://./apache-install
```
El comando ```aws autoscaling create-launch-configuration``` no produce ninguna salida cuando se ejecuta correctamente.
Pero para saber si creo correctamente el lanzamiento se puede usar el comando.
<BR> 

Descripción: 

```
aws autoscaling describe-launch-configurations --launch-configuration-names taipe-lc
```
```
{
    "LaunchConfigurations": [
        {
            "LaunchConfigurationName": "taipe-lc",
            "LaunchConfigurationARN": "arn:aws:autoscaling:us-east-1:075250088937:launchConfiguration:2abb346b-f24b-4e02-9149-ee7cee2f422f:launchConfigurationName/taipe-lc",
            "ImageId": "ami-d9a98cb0",
            "KeyName": "taipe-key",
            "SecurityGroups": [
                "taipe"
            ],
            "ClassicLinkVPCSecurityGroups": [],
            "UserData": "IyFiaW4vYmFzaAojUHJldmVudCBhcHQtZ2V0IGZyb20gYnJpbmdpbmcgdXAgYW55IGludGVyYWN0aXZlIHF1ZXJpZXMKZXhwb3J0IERFQklBTl9GUk9OVEVORD1ub25pbnRlcmFjdGl2ZQojIFVwZ3JhZGUgcGFja2FnZXMKYXB0LWdldCB1cGRhdGUKIyBJbnN0YWxsIEFwYWNoZQphcHQtZ2V0aW5zdGFsbCBhcGFjaGUyIC15Cg==",
            "InstanceType": "t1.micro",
            "KernelId": "",
            "RamdiskId": "",
            "BlockDeviceMappings": [],
            "InstanceMonitoring": {
                "Enabled": true
            },
            "CreatedTime": "2023-06-17T02:20:45.793000+00:00",
            "EbsOptimized": false
        }
    ]
}

```
<BR>A continuación, crea un equilibrador de carga.
```
aws elb create-load-balancer --load-balancer-name taipe-elb --listeners "Protocol=HTTP, LoadBalancerPort=80, InstanceProtocol=HTTP,InstancePort=80" --availability-zones us-east-1c
```

```
{
    "DNSName": "taipe-elb-600151083.us-east-1.elb.amazonaws.com"
}

```
La salida que se  proporcionó indica que se ha creado un Elastic Load Balancer (ELB) con el nombre DNS `taipe-elb-600151083.us-east-1.elb.amazonaws.com`. Un ELB distribuye el tráfico entrante entre múltiples instancias de Amazon EC2 en una o más zonas de disponibilidad. El nombre DNS del ELB es la dirección que puedes usar para enviar solicitudes al balanceador de carga.


<BR>2. **Ahora creamos un grupo de escalado automático:**

```
aws autoscaling create-auto-scaling-group --auto-scaling-group-name taipe-asg --launch-configuration-name taipe-lc --min-size 1 --max-size 3 --load-balancer-names taipe-elb --availability-zones us-east-1c
```
El comando``` aws autoscaling create-auto-scaling-group``` se ejecuta correctamente, no debería producir ninguna salida.

<BR>3. **Para describir el grupo de escalado automático que acabas de crear:**
Para  verificar si el grupo de Auto Scaling se creó ejecutando el comando ``` aws autoscaling describe-auto-scaling-groups```  y buscando el nombre del grupo de Auto Scaling en la salida.
```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names taipe-asg
```
```
{
    "AutoScalingGroups": [
        {
            "AutoScalingGroupName": "taipe-asg",
            "AutoScalingGroupARN": "arn:aws:autoscaling:us-east-1:075250088937:autoScalingGroup:6b0690f3-3bae-4916-a796-f4fffbfcdbf1:autoScalingGroupName/taipe-asg",
            "LaunchConfigurationName": "taipe-lc",
            "MinSize": 1,
            "MaxSize": 3,
            "DesiredCapacity": 1,
            "DefaultCooldown": 300,
            "AvailabilityZones": [
                "us-east-1c"
            ],
            "LoadBalancerNames": [
                "taipe-elb"
            ],
            "TargetGroupARNs": [],
            "HealthCheckType": "EC2",
            "HealthCheckGracePeriod": 0,
            "Instances": [
                {
                    "InstanceId": "i-097798ce9631a25df",
                    "InstanceType": "t1.micro",
                    "AvailabilityZone": "us-east-1c",
                    "LifecycleState": "InService",
                    "HealthStatus": "Healthy",
                    "LaunchConfigurationName": "taipe-lc",
                    "ProtectedFromScaleIn": false
                }
            ],
            "CreatedTime": "2023-06-17T02:39:28.017000+00:00",
            "SuspendedProcesses": [],
            "VPCZoneIdentifier": "",
            "EnabledMetrics": [],
            "Tags": [],
            "TerminationPolicies": [
                "Default"
            ],
            "NewInstancesProtectedFromScaleIn": false,
            "ServiceLinkedRoleARN": "arn:aws:iam::075250088937:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling",
            "TrafficSources": [
                {
                    "Identifier": "taipe-elb",
                    "Type": "elb"
                }
            ]
        }
    ]
}

```

¿Cuál es la salida? Deberías ver que se crea una nueva instancia EC2. Si no lo ves, espera 2 minutos y vuelve a ejecutar el comando. ¿Cuál es el ID de la instancia?
* La salida que has proporcionado muestra información sobre el grupo de Auto Scaling ```taipe-asg``` que se creó. Según la salida, se ha creado una instancia EC2 en el grupo de Auto Scaling. El ID de la instancia es ```i-097798ce9631a25df```, como se muestra en la sección "Instances" de la salida.

<BR>4. **Ahora creamos una política de escalado hacía arriba seguida de una alarma de
CloudWatch:**
<BR>**Valor del PolicyARN**
```
aws autoscaling put-scaling-policy --auto-scaling-group-name taipe-asg --policy-name taipe-scaleup --scaling-adjustment 1 --adjustment-type ChangeInCapacity --cooldown 120
```
```
{
    "PolicyARN": "arn:aws:autoscaling:us-east-1:075250088937:scalingPolicy:305dfcce-6699-4a01-a399-1286721df3c0:autoScalingGroupName/taipe-asg:policyName/taipe-scaleup",
    "Alarms": []
}

```
El campo PolicyARN contiene el nombre de recurso de Amazon (ARN) de la política, que identifica de manera única la política. En este caso, el ARN es ```arn:aws:autoscaling:us-east-1:075250088937:scalingPolicy:305dfcce-6699-4a01-a399-1286721df3c0:autoScalingGroupName/taipe-asg:policyName/taipe-scaleup```, lo que indica que la política está asociada con el grupo de Auto Scaling ```taipe-asg``` y tiene el nombre ```taipe-scaleup```.
**Crear una alarma de Amazon CloudWatch**

```
aws cloudwatch put-metric-alarm --alarm-name taipe-highcpualarm --metric-name CPUUtilization --namespace AWS/EC2 --statistic Average --period 120 --threshold 70 --comparison-operator GreaterThanThreshold --dimensions "Name=AutoScalingGroupName,Value=taipe-asg" --evaluation-periods 1 --alarm-actions arn:aws:autoscaling:us-east-1:075250088937:scalingPolicy:305dfcce-6699-4a01-a399-1286721df3c0:autoScalingGroupName/taipe-asg:policyName/taipe-scaleup
```
El comando ```aws cloudwatch put-metric-alarm``` y no recibes ninguna salida en la consola, es posible que la alarma de CloudWatch se haya creado correctamente.

<BR>**Descripción del alarma.**
```
aws cloudwatch describe-alarms --alarm-names taipe-highcpualarm
```
```
{
    "MetricAlarms": [
        {
            "AlarmName": "taipe-highcpualarm",
            "AlarmArn": "arn:aws:cloudwatch:us-east-1:075250088937:alarm:taipe-highcpualarm",
            "AlarmConfigurationUpdatedTimestamp": "2023-06-17T02:53:22.863000+00:00",
            "ActionsEnabled": true,
            "OKActions": [],
            "AlarmActions": [
                "arn:aws:autoscaling:us-east-1:075250088937:scalingPolicy:305dfcce-6699-4a01-a399-1286721df3c0:autoScalingGroupName/taipe-asg:policyName/taipe-scaleup"
            ],
            "InsufficientDataActions": [],
            "StateValue": "OK",
            "StateReason": "Threshold Crossed: 1 datapoint [0.322580645161292 (17/06/23 02:53:00)] was not greater than the threshold (70.0).",
            "StateReasonData": "{\"version\":\"1.0\",\"queryDate\":\"2023-06-17T02:55:05.492+0000\",\"startDate\":\"2023-06-17T02:53:00.000+0000\",\"statistic\":\"Average\",\"period\":120,\"recentDatapoints\":[0.322580645161292],\"threshold\":70.0,\"evaluatedDatapoints\":[{\"timestamp\":\"2023-06-17T02:53:00.000+0000\",\"sampleCount\":1.0,\"value\":0.322580645161292}]}",
            "StateUpdatedTimestamp": "2023-06-17T02:55:05.497000+00:00",
            "MetricName": "CPUUtilization",
            "Namespace": "AWS/EC2",
            "Statistic": "Average",
            "Dimensions": [
                {
                    "Name": "AutoScalingGroupName",
                    "Value": "taipe-asg"
                }
            ],
            "Period": 120,
            "EvaluationPeriods": 1,
            "Threshold": 70.0,
            "ComparisonOperator": "GreaterThanThreshold",
            "StateTransitionedTimestamp": "2023-06-17T02:55:05.497000+00:00"
        }
    ],
    "CompositeAlarms": []
}
```

La salida que ha proporcionado muestra información sobre la alarma de CloudWatch `taipe-highcpualarm` que se creó. Según la salida, la alarma se activará cuando el valor promedio de la métrica `CPUUtilization` para las instancias en el grupo de Auto Scaling `taipe-asg` supere el 70% durante un período de 120 segundos. Cuando se active la alarma, se ejecutará la acción especificada en el campo `"AlarmActions"`, que en este caso es la política de escalado `taipe-scaleup`.

La alarma actualmente está en estado `"OK"`, lo que significa que no se ha activado. La razón se proporciona en el campo `"StateReason"`, que indica que el último valor de la métrica `CPUUtilization` no superó el umbral del 70%.


<BR>5. **Inicia sesión en la instancia EC2 desde el paso 1 mediante ssh:**

Instalamos strees en el root.

```
apt-get install
stress stress--cpu 1
```
Repite el siguiente comando cada 2 minutos hasta que veas la segunda instancia EC2 y luego una tercera instancia EC2.
```
stress --cpu 1
stress: info: [3685] dispatching hogs: 1 cpu, 0 io, 0 vm, 0 hdd
stress --cpu 2
stress: info: [3690] dispatching hogs: 2 cpu, 0 io, 0 vm, 0 hdd
stress --cpu 3
stress: info: [3693] dispatching hogs: 3 cpu, 0 io, 0 vm, 0 hdd
```
**Descripción:**

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name taipe -asg
```
Salida:
```
{
    "AutoScalingGroups": [
        {
            "AutoScalingGroupName": "taipe-asg",
            "AutoScalingGroupARN": "arn:aws:autoscaling:us-east-1:075250088937:autoScalingGroup:6b0690f3-3bae-4916-a796-f4fffbfcdbf1:autoScalingGroupName/taipe-asg",
            "LaunchConfigurationName": "taipe-lc",
            "MinSize": 1,
            "MaxSize": 3,
            "DesiredCapacity": 3,
            "DefaultCooldown": 300,
            "AvailabilityZones": [
                "us-east-1c"
            ],
            "LoadBalancerNames": [
                "taipe-elb"
            ],
            "TargetGroupARNs": [],
            "HealthCheckType": "EC2",
            "HealthCheckGracePeriod": 0,
            "Instances": [
                {
                    "InstanceId": "i-031ad32028bcf53c8",
                    "InstanceType": "t1.micro",
                    "AvailabilityZone": "us-east-1c",
                    "LifecycleState": "Pending",
                    "HealthStatus": "Healthy",
                    "LaunchConfigurationName": "taipe-lc",
                    "ProtectedFromScaleIn": false
                },
                {
                    "InstanceId": "i-055cbcb23c52464ae",
                    "InstanceType": "t1.micro",
                    "AvailabilityZone": "us-east-1c",
                    "LifecycleState": "InService",
                    "HealthStatus": "Healthy",
                    "LaunchConfigurationName": "taipe-lc",
                    "ProtectedFromScaleIn": false
                },
                {
                    "InstanceId": "i-097798ce9631a25df",
                    "InstanceType": "t1.micro",
                    "AvailabilityZone": "us-east-1c",
                    "LifecycleState": "InService",
                    "HealthStatus": "Healthy",
                    "LaunchConfigurationName": "taipe-lc",
                    "ProtectedFromScaleIn": false
                }
            ],
            "CreatedTime": "2023-06-17T02:39:28.017000+00:00",
            "SuspendedProcesses": [],
            "VPCZoneIdentifier": "",
            "EnabledMetrics": [],
            "Tags": [],
            "TerminationPolicies": [
                "Default"
            ],
            "NewInstancesProtectedFromScaleIn": false,
            "ServiceLinkedRoleARN": "arn:aws:iam::075250088937:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling",
            "TrafficSources": [
                {
                    "Identifier": "taipe-elb",
                    "Type": "elb"
                }
            ]
        }
    ]
}
```
Como máximo mostrara 3 instancias porque pusimos el comando  ```--max-size 3```

>
**__Parte 2: reducir la escala__**<BR>
<BR>6. **Ahora exploraremos cómo AWS puede controlar el escalado hacia abajo mediante la
eliminación de algunas de las máquinas virtuales creadas:**
<BR>

**Valor de PolicyARN**
```
 
aws autoscaling put-scaling-policy --auto-scaling-group-name taipe-asg --policy-name taipe-scaledown --scaling-adjustment -1 --adjustment-type ChangeInCapacity --cooldown 120
```
```
{
    "PolicyARN": "arn:aws:autoscaling:us-east-1:075250088937:scalingPolicy:d4025a2d-f993-43fe-94df-fd43af82352f:autoScalingGroupName/taipe-asg:policyName/taipe-scaledown",
    "Alarms": []
}

```
El valor de PolicyARN es: 

```
arn:aws:autoscaling:us-east-1:075250088937:scalingPolicy:d4025a2d-f993-43fe-94df-fd43af82352f:autoScalingGroupName/taipe-asg:policyName/taipe-scaledown
```
**CloudWatch**
```
aws cloudwatch put-metric-alarm --alarm-name taipe-lowcpualarm --metric-name CPUUtilization --namespace AWS/EC2 --statistic Average --period 120 --threshold 30 --comparison-operator LessThanThreshold --dimensions "Name=AutoScalingGroupName,Value=taipe-asg" --evaluation-periods 1 --alarm-actions arn:aws:autoscaling:us-east-1:075250088937:scalingPolicy:d4025a2d-f993-43fe-94df-fd43af82352f:autoScalingGroupName/taipe-asg:policyName/taipe-scaledown

```
Este comando crea una alarma de CloudWatch llamada ```taipe-lowcpualarm``` que monitorea la métrica ```CPUUtilization``` para el grupo de Auto Scaling especificado. La alarma se activa cuando el uso promedio de la CPU es inferior al 30% durante un período de 120 segundos. Cuando se activa la alarma, se ejecutará la política de escalado ```taipe-scaledown``` que creó anteriormente para el grupo de Auto Scaling``` taipe-asg```.
Si no ve ninguna salida después de ejecutar este comando, significa que el comando se ejecutó correctamente y se creó la alarma. Puede verificar que se creó la alarma ejecutando el comando ```aws cloudwatch describe-alarms --alarm-names taipe-lowcpualarm```.


 
<BR>7. **Cambia al terminal de la instancia EC2. Escribe ctrl+c para detener el comando stress.
Vuelva a la ventana del terminal y repite el siguiente comando cada 2 minutos hasta
que vea solo un EC2 en su grupo de escalado automático.**

```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name taipe-asg

```

```
{
    "AutoScalingGroups": [
        {
            "AutoScalingGroupName": "taipe-asg",
            "AutoScalingGroupARN": "arn:aws:autoscaling:us-east-1:075250088937:autoScalingGroup:6b0690f3-3bae-4916-a796-f4fffbfcdbf1:autoScalingGroupName/taipe-asg",
            "LaunchConfigurationName": "taipe-lc",
            "MinSize": 1,
            "MaxSize": 3,
            "DesiredCapacity": 1,
            "DefaultCooldown": 300,
            "AvailabilityZones": [
                "us-east-1c"
            ],
            "LoadBalancerNames": [
                "taipe-elb"
            ],
            "TargetGroupARNs": [],
            "HealthCheckType": "EC2",
            "HealthCheckGracePeriod": 0,
            "Instances": [
                {
                    "InstanceId": "i-055cbcb23c52464ae",
                    "InstanceType": "t1.micro",
                    "AvailabilityZone": "us-east-1c",
                    "LifecycleState": "InService",
                    "HealthStatus": "Healthy",
                    "LaunchConfigurationName": "taipe-lc",
                    "ProtectedFromScaleIn": false
                }
            ],
            "CreatedTime": "2023-06-17T02:39:28.017000+00:00",
            "SuspendedProcesses": [],
            "VPCZoneIdentifier": "",
            "EnabledMetrics": [],
            "Tags": [],
            "TerminationPolicies": [
                "Default"
            ],
            "NewInstancesProtectedFromScaleIn": false,
            "ServiceLinkedRoleARN": "arn:aws:iam::075250088937:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling",
            "TrafficSources": [
                {
                    "Identifier": "taipe-elb",
                    "Type": "elb"
                }
            ]
        }
    ]
}
```
Este es el resultado del comando `aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name taipe-asg`, que muestra información detallada sobre el grupo de Auto Scaling especificado. La salida muestra que el grupo de Auto Scaling se llama `taipe-asg` y está asociado con la configuración de lanzamiento `taipe-lc`. El tamaño mínimo del grupo es 1 y el tamaño máximo es 3, con una capacidad deseada de 1. Actualmente hay una instancia EC2 en el grupo, con el ID `i-055cbcb23c52464ae`, que se encuentra en la zona de disponibilidad `us-east-1c` y está en estado `InService`. También se muestra información sobre el balanceador de carga asociado, las políticas de terminación y otras configuraciones del grupo de Auto Scaling. 


>
**__Parte 3: Limpieza__**<BR>
<BR>8. **Elimina el grupo de escalado automático:**

```
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name taipe-asg --force-delete
```
El comando ```aws autoscaling delete-auto-scaling-group``` no produce ningún resultado cuando se ejecuta correctamente. Si el comando se ejecuta con la opción ```--force-delete```, eliminará el grupo de Auto Scaling especificado junto con todas las instancias asociadas al grupo, sin esperar a que finalicen todas las instancias.
<BR>**Elimina tu configuración de lanzamiento** 
<BR>
```aws autoscaling describe-auto-scaling-groups --auto-scaling-group-name taipe-asg```
 
```
{
    "AutoScalingGroups": []
}
```
 La salida indica que no se encontró ningún grupo de Auto Scaling con el nombre especificado. Esto significa que el grupo de Auto Scaling taipe-asg ya no existe, probablemente porque se eliminó.
<BR>
<BR>**Eliminar el grupo de Auto Scaling**
```aws autoscaling delete-launch-configuration --launch-configuration-name taipe-lc```

El comando ```delete-launch-configuration``` de aws autoscaling no produce ningún resultado cuando se ejecuta correctamente. Este comando se utiliza para eliminar la configuración de inicio especificada. La configuración de inicio no debe adjuntarse a un grupo de Auto Scaling. Cuando se completa esta llamada, la configuración de inicio ya no está disponible para su uso.

**Importante:**

Para eliminar un Elastic Load Balancer (ELB), puede usar el comando ```aws elb delete-load-balancer --load-balancer-name``` con el nombre de su ELB. Por ejemplo, si su ELB se llama taipe-elb, el comando es  ```aws elb delete-load-balancer --load-balancer-name taipe-elb```. Si el comando se ejecuta correctamente, no debería haber ninguna salida.


