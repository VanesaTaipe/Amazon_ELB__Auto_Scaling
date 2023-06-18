#  Amazon ELB - Auto Scaling
**Amazon ELB**<BR>
**__Parte 1: ELB__**
<BR>1. **Crear un balanceador de carga:**

```
aws elb create-load-balancer --load-balancer-name taipe --listeners "Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=80" --availability-zones us-east-1d

```<BR>
La salida que obtenemos nos indica el DNS_Name del balanceador de carga.
```
{
    "DNSName": "taipe-931254177.us-east-1.elb.amazonaws.com"
}
```<BR> La clave DNSName y un valor de ```taipe-931254177.us-east-1.elb.amazonaws.com```. Esto es un nombre DNS de Amazon Web Services (AWS) para un Balanceador de Carga Elástico (ELB) en la región us-east-1.                                                                                   
 <BR> 2.  **El comando describe-load-balancers describe el estado y las propiedades de tu(s) balanceador(es) de carga:**<BR>  
```
aws elb describe-load-balancers --load-balancer-name taipe
```
La salida brinda una descripción del ELB de Amazon Web Cervices.
```
{
    "LoadBalancerDescriptions": [
        {
            "LoadBalancerName": "taipe",
            "DNSName": "taipe-931254177.us-east-1.elb.amazonaws.com",
            "CanonicalHostedZoneName": "taipe-931254177.us-east-1.elb.amazonaws.com",
            "CanonicalHostedZoneNameID": "Z35SXDOTRQ7X7K",
            "ListenerDescriptions": [
                {
                    "Listener": {
                        "Protocol": "HTTP",
                        "LoadBalancerPort": 80,
                        "InstanceProtocol": "HTTP",
                        "InstancePort": 80
                    },
                    "PolicyNames": []
                }
            ],
            "Policies": {
                "AppCookieStickinessPolicies": [],
                "LBCookieStickinessPolicies": [],
                "OtherPolicies": []
            },
            "BackendServerDescriptions": [],
            "AvailabilityZones": [
                "us-east-1d"
            ],
            "Subnets": [
                "subnet-0665aa225f13aa3e5"
            ],
            "VPCId": "vpc-0f8c790d41c4fe225",
            "Instances": [],
            "HealthCheck": {
                "Target": "TCP:80",
                "Interval": 30,
                "Timeout": 5,
                "UnhealthyThreshold": 2,
                "HealthyThreshold": 10
            },
            "SourceSecurityGroup": {

```
En este caso, la salida indica que hay un Balanceador de Carga Elástico (ELB)  de carga llamado taipe con un nombre DNS de ``` taipe-931254177.us-east-1.elb.amazonaws.com```. El balanceador de carga tiene un oyente configurado para el protocolo HTTP en el puerto 80 del balanceador de carga y el puerto 80 de la instancia. El balanceador de carga está disponible en la zona de disponibilidad ``` us-east-1``` y está asociado con una subred específica en una VPC específica. Actualmente, no hay instancias registradas con el balanceador de carga. 
<BR>3. **Creamos dos instancias EC2, cada una ejecutando un servidor web Apache:**
```
aws ec2 run-instances --image-id ami-d9a98cb0 --count 2 --instance-type t1.micro --key-name taipe-key --security-groups taipe --user-data file://./apache-install --placement AvailabilityZone=us-east-1d
```
Nos brinda las siguiente salida.
```
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 1,
            "ImageId": "ami-d9a98cb0",
            "InstanceId": "i-0a993703af8ab5bd7",
            "InstanceType": "t1.micro",
            "KernelId": "aki-88aa75e1",
            "KeyName": "taipe-key",
            "LaunchTime": "2023-06-15T14:37:54+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-1d",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-31-90-173.ec2.internal",
            "PrivateIpAddress": "172.31.90.173",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-0665aa225f13aa3e5",
            "VpcId": "vpc-0f8c790d41c4fe225",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "cf6e866d-1b09-4742-84db-090743331eec",
            "EbsOptimized": false,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2023-06-15T14:37:54+00:00",
                        "AttachmentId": "eni-attach-0d616eca533c67b94",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "taipe",
                            "GroupId": "sg-0a09e65ba14e155e0"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "12:64:0d:18:83:2d",
                    "NetworkInterfaceId": "eni-0cceb8f6d33044f9c",
                    "OwnerId": "075250088937",
                    "PrivateDnsName": "ip-172-31-90-173.ec2.internal",
                    "PrivateIpAddress": "172.31.90.173",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-31-90-173.ec2.internal",
                            "PrivateIpAddress": "172.31.90.173"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-0665aa225f13aa3e5",
                    "VpcId": "vpc-0f8c790d41c4fe225",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/sda1",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "taipe",
                    "GroupId": "sg-0a09e65ba14e155e0"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "paravirtual",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1,
                "HttpEndpoint": "enabled",
                "HttpProtocolIpv6": "disabled",
                "InstanceMetadataTags": "disabled"
            },
            "EnclaveOptions": {
                "Enabled": false
            },
            "PrivateDnsNameOptions": {
                "HostnameType": "ip-name",
                "EnableResourceNameDnsARecord": false,
                "EnableResourceNameDnsAAAARecord": false
            },
            "MaintenanceOptions": {
                "AutoRecovery": "default"
            },
            "CurrentInstanceBootMode": "legacy-bios"
        },
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-d9a98cb0",
            "InstanceId": "i-09ba5dbeabb17ff6b",
            "InstanceType": "t1.micro",
            "KernelId": "aki-88aa75e1",
            "KeyName": "taipe-key",
            "LaunchTime": "2023-06-15T14:37:54+00:00",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-1d",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-31-93-80.ec2.internal",
            "PrivateIpAddress": "172.31.93.80",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-0665aa225f13aa3e5",
            "VpcId": "vpc-0f8c790d41c4fe225",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "cf6e866d-1b09-4742-84db-090743331eec",
            "EbsOptimized": false,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2023-06-15T14:37:54+00:00",
                        "AttachmentId": "eni-attach-0394576439ec74dc0",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching",
                        "NetworkCardIndex": 0
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "taipe",
                            "GroupId": "sg-0a09e65ba14e155e0"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "12:00:cf:96:f0:b5",
                    "NetworkInterfaceId": "eni-07049787cf680e96e",
                    "OwnerId": "075250088937",
                    "PrivateDnsName": "ip-172-31-93-80.ec2.internal",
                    "PrivateIpAddress": "172.31.93.80",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-31-93-80.ec2.internal",
                            "PrivateIpAddress": "172.31.93.80"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-0665aa225f13aa3e5",
                    "VpcId": "vpc-0f8c790d41c4fe225",
                    "InterfaceType": "interface"
                }
            ]

```
¿Qué parte de este comando indica que deseas dos instancias EC2? ¿Qué parte de este comando garantiza que tus instancias tendrán Apache instalado? ¿Cuál es el ID de instancia de la primera instancia? ¿Cuál es el ID de instancia de la segunda instancia?<
* La parte del comando que indica que deseas lanzar dos instancias EC2 es  ``` --count 2 ```  en Amazon EC2.<BR>
* La parte del comando que garantiza que tus instancias tendrán Apache instalado es ``` --user-data file://./apache-install ```, que especifica un archivo de datos de usuario que contiene un script para instalar Apache en las instancias después de que se inicien.
* La primera instancia su ID es ``` i-0a993703af8ab5bd7``` .
* La segunda instancia su ID es ``` i-09ba5dbeabb17ff6b``` .

<BR>4. **Para usar ELB, tenemos que registrar las instancias EC2:**<BR>

Usamos el siguiente comando:
```
aws elb register-instances-with-load-balancer --load-balancer-name taipe --instances i-0a993703af8ab5bd7   i-09ba5dbeabb17ff6b

```
<BR>La salida es:
```
{
    "Instances": [
        {
            "InstanceId": "i-0a993703af8ab5bd7"
        },
        {
            "InstanceId": "i-09ba5dbeabb17ff6b"
        }
    ]
}
```

**Ahora vea el estado de la instancia de los servidores cuya carga se equilibra.**
<BR>```  aws elb describe-instance-health --load-balancer-name taipe```
<BR>La salida es:
```
{
    "Instances": [
        {
            "InstanceId": "i-0a993703af8ab5bd7",
            "State":  "OutOfService",
            "ReasonCode": "Instance",
            "Description": "Instance has faildes at least the Unhealthy Threshold number od health checks
consecutively."
        },
        {
            "InstanceId": "i-09ba5dbeabb17ff6b",
             "State":  "OutOfService",
            "ReasonCode": "Instance",
            "Description": "Instance has faildes at least the Unhealthy Threshold number od health checks
consecutively."
        }
    ]
}
```
La salida muestra que ambas instancias, ```i-0a993703af8ab5bd7``` e  ```i-09ba5dbeabb17ff6b```, están en estado ```OutOfService``` . El ELB no dirigirá el tráfico a estas instancias hasta que pasen los chequeos de salud. El  hecho de que el ELB no dirija el tráfico a instancias que no pasan los chequeos de salud es una característica de seguridad diseñada para garantizar que solo las instancias saludables reciban tráfico.

<BR> 5. **Recupera la dirección IP de tu balanceador de carga del paso 1, ingresa la URL ```http://taipe-931254177.us-east-1.elb.amazonaws.com``` en tu navegador web.
 ¿Qué apareció en el navegador?**
<BR> El siguiente link muestra el resultado:
<https://github.com/VanesaTaipe/imageb/blob/main/sa.png> 

<BR> 6. **Abre dos ventanas de terminal adicionales y ssh en ambos servidores web. En cada uno, cd al directorio DocumentRoot (probablemente /usr/local/apache/htdocs) y modifique la página de inicio predeterminada, index.html, de la siguiente manera.**
```
<html><body><h1>¡Funciona!</h1>
<p>La solicitud se envió a la instancia 1.</p>
<p>La solicitud fue atendida por el servidor web 1.</p>
</body><html>
```
El resultado se muestra en el siguiente link: <https://github.com/VanesaTaipe/imageb/blob/main/Captura%20de%20pantalla%202023-06-17%20214336.png>

```
<html><body><h1>¡Funciona!</h1>
<p>La solicitud se envió a la instancia 2.</p>
<p>La solicitud fue atendida por el servidor web 2.</p>
</body><html>
```
El resultado se muestra en el siguiente link: 
<https://github.com/VanesaTaipe/imageb/blob/main/Captura%20de%20pantalla%202023-06-17%20215019.png>

Para el segundo servidor, haz lo mismo excepto que use la instancia 2 y el servidor 2 para las líneas 2 y 3. En el navegador web, accede a tu balanceador de carga 4 veces (actualícelo/recárgalo 4 veces). Esto genera 4 solicitudes a tu balanceador de carga.
¿Cuántas solicitudes atendió el servidor web 1? ¿Cuántas solicitudes atendió el servidor web 2?
* Si el balanceador de carga con dos servidores web, y accedes al balanceador de carga 4 veces, por lo tanto, el servidor web 1 atendió 2 solicitudes y el servidor web 2  atiendo 2 solicitudes.

<BR>**__Parte 2: CloudWatch__**
<BR>7. **CloudWatch se utiliza para monitorear instancias. En este caso, queremos monitorear los dos servidores web.**<BR> 
Iniciamos CloudWatch de la siguiente manera.
<BR>
**Monitorear instancias**

```  aws ec2 monitor-instances --instance-ids i-0a993703af8ab5bd7 i-09ba5dbeabb17ff6b ```

<BR>La salida que brinda es la siguiente:
<BR>
```
{
    "InstanceMonitorings": [
        {
            "InstanceId": "i-0a993703af8ab5bd7",
          
         "Monitoring": {
       
         "State": "pending"

            }

        },

        {

            "InstanceId": "i-09ba5dbeabb17ff6b",

            "Monitoring": {

                "State": "pending"
            }
 }
```
La salida que has compartido parece ser el resultado de un comando para verificar el estado de monitoreo detallado de las instancias EC2 en Amazon Web Services (AWS). La salida muestra que ambas instancias, i-0a993703af8ab5bd7 e i-09ba5dbeabb17ff6b, tienen un estado de monitoreo detallado pending, lo que significa que el monitoreo detallado está habilitado para estas instancias.
<BR>**Métricas disponibles**
<BR>```aws cloudwatch list-metrics --namespaces "AWS/EC2"```

```
{
    "Metrics": [
        {
            "Namespace": "AWS/EC2",
            "MetricName": "DiskReadOps,
            "Dimensions":
                {
                    "Name": "InstanceId",
                    "Value": "i-0ef8dde34a0bc2cd14"
                }
            ]
        },
        {
            "Namespace": "AWS/EC2",
            "MetricName": "DiskReadBytes",
            "Dimensions": [
                {
                    "Name": "InstanceId",
                    "Value": "i-0a993703af8ab5bd7"
                }
            ]
        },
        {
            "Namespace": "AWS/EC2",
            "MetricName": "CPUUtilization",
            "Dimensions": [
                {
                    "Name": "InstanceId",
                    "Value": "i-0528cd56037b8c861"
                }
            ]
        },
       
            "Namespace": "AWS/EC2",
            "MetricName": "CPUUtilization",
            "Dimensions": [
                {
                    "Name": "InstanceId",
                    "Value": "i-081184a3f50f4f972"
                }
            ]
        }
}
```
¿Viste la métrica ```CPUUtilization``` en el resultado?
* Si, la ```CPUUtilization ```se muestra.

<BR>8. **Ahora configuramos una métrica para recopilar la utilización de la CPU. Obtener la
hora actual con date -u. Esta será tu hora de inicio. La hora de finalización debe ser 30
minutos más tarde.**
<BR> 
```
aws cloudwatch get-metric-statistics --metric-name CPUUtilization --start-time 2023-06-15T16:50:56Z --end-time 2023-06-15T16:51:24Z --period 3600 --namespace AWS/EC2 --statistics Maximum --dimensions "Name=InstanceId,Value=i-09ba5dbeabb17ff6b" 
```
La salida es la siguiente:
```
{
    "Label": "CPUUtilization",
    "Datapoints": [
        {
            "Timestamp": "2023-06-15T16:50:00+00:00",
            "Maximum": 0.677966101694925,
            "Unit": "Percent"
        }
    ]
}

```
El campo ```Label``` indica que los datos son sobre ```CPUUtilization```. El campo ```Datapoints ```contiene una matriz de objetos, cada uno representando un punto de datos. En este caso, solo hay un punto de datos, con una ```Timestamp```de ```"2023-06-15T16:50:00+00:00"```, un valor Maximum de ```0.677966101694925``` y un``` Unit Perce"```. Esto significa que en el momento especificado, la utilización máxima de la CPU fue aproximadamente del 0.678%, o alrededor del 0.68%.

<BR>9. **Apache tiene un herramienta benchmark llamada ab. Si desea ver más información sobre ab, consulte http://httpd.apache.org/docs/2.0/programs/ab.html.**
```
 ab -n 50 -c 50 http://taipe-931254177.us-east-1.elb.amazonaws.com/
```


```
Server Software:        Apache/2.2.22
Server Hostname:        taipe-931254177.us-east-1.elb.amazonaws.com
Server Port:            80

Document Path:          /
Document Length:        152 bytes

Concurrency Level:      50
Time taken for tests:   0.018 seconds
Complete requests:      50
Failed requests:        10
    (Connect: 0, Receive: 0, Length: 10, Exceptions: 0)
Write errors:
Total transferred:      21340 bytes
HTML transferred:       7590 bytes
Requests per second:    2722.72 [#/sec] (mean)
Time per request:       18.364 [ms] (mean)
Time per request:       0.367 [ms] (mean, across all concurrent requests)
Transfer rate:          1134.82 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        1    2   0.3      0       2
Processing:     3   11   2.6     11      16
Waiting:        3   11   2.6     10      16
Total:          5   13   2.5     13      17

Percentage of the requests served within a certain time (ms)
50%    13
66%    13
75%    14
80%    15
90%    16
95%    17
98%    17
99%    17
100%   17 (longest request)

```
¿Qué significan ``` -n 50 ```y  ```-c 5```? ¿Cuál es la salida?
* El parámetro ```-n``` especifica el número total de solicitudes a realizar durante la prueba de rendimiento, mientras que el parámetro``` -c ``` especifica el nivel de concurrencia, es decir, el número de solicitudes múltiples a realizar al mismo tiempo. En este caso, ```-n 50``` indica que se realizarán un total de 50 solicitudes y ```-c 5``` indica que se realizarán 5 solicitudes simultáneamente.

<BR>10. **Ahora queremos examinar la métrica de latencia del ELB. Utiliza el siguiente comando con las mismas horas de inicio y finalización que especificó en el paso 8.**

```
aws cloudwatch get-metric-statistics --metric-name Latency --start-time 2023-06-15T18:54:18Z --end-time 2023-06-15T18:55:59Z --period 3600 --namespace AWS/ELB --statistics Maximum --dimensions Name=LoadBalancerName,Value=taipe
```

```
{
    "Label": "Latency",
    "Datapoints": [ ]

 }
```
La salida indica que la métrica solicitada es “Latency” y que no se han devuelto puntos de datos, como se indica por el arreglo “Datapoints” .
```
 aws cloudwatch get-metric-statistics --metric-name RequestCount --start-time 2023-06-15T18:54:18Z --end-time 2023-06-15T19:54:18Z --period 3600 --namespace AWS/ELB --statistics Sum --dimensions Name=LoadBalancerName,Value=taipe
```
```
{
    "Label": "RequestCount",
    "Datapoints": [
        {
            "Timestamp": "2023-06-15T18:54:00+00:00",
            "Sum": 3.0,
            "Unit": "Count"
        }
    ]
}

```
La salida indica que la métrica solicitada es ``` “RequestCount”``` y que se ha devuelto un solo punto de datos.
El punto de datos muestra que en el momento especificado por el campo ```“Timestamp”: (2023-06-15T18:54:00+00:00)```, el balanceador de carga procesó un total de 3 solicitudes, como se indica en el campo ```“Sum”```. La unidad utilizada para medir el número de solicitudes es ```“Count”```, como se indica en el campo ``` “Unit”```.

<BR>**__Parte 3: Limpieza__**
<BR>11. **Necesitamos cancelar el registro de las instancias de ELB**

```
aws elb deregister-instances-from-load-balancer --load-balancer-name taipe --instances i-0a993703af8ab5bd7 i-09ba5dbeabb17ff6b
```
```
{
    "Instances": []
}
```
Esta salida muestra los resultados de un comando de AWS Elastic Load Balancing (ELB) que desregistra instancias de un balanceador de carga. La salida indica que el comando se ejecutó correctamente y que ya no hay instancias registradas en el balanceador de carga, como se indica por el arreglo ```“Instances”``` vacío.
<BR>12. **A continuación, eliminamos la instancia de ELB de la siguiente manera.**
```
aws elb delete-load-balancer --load-balancer-name taipe
```
Este comando de AWS Elastic Load Balancing (ELB) se utiliza para eliminar un balanceador de carga. El comando se ejecuta correctamente y no devuelve ningún mensaje de error, significa que el balanceador de carga especificado por el parámetro``` --load-balancer-name``` (en este caso, ```“taipe”``` ) ha sido eliminado correctamente.
```
aws ec2 terminate-instances --instance-ids i-0a993703af8ab5bd7
```

```
{
    "TerminatingInstances": [
        {
            "InstanceId": "i-0a993703af8ab5bd7",
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
```
La salida muestra que la instancia con el ID``` i-0a993703af8ab5bd7b``` fue terminada con éxito. El campo CurrentState contiene un objeto con información sobre el estado actual de la instancia. En este caso, el Code es 32 y el Name es ```"shutting-down"```, lo que indica que la instancia está en proceso de apagado.


```
aws ec2 terminate-instances --instance-ids i-09ba5dbeabb17ff6b
```
```
{
    "TerminatingInstances": [
        {
            "InstanceId": "i-09ba5dbeabb17ff6b",
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
```
La salida muestra que la instancia con el ID ```i-09ba5dbeabb17ff6b``` fue terminada con éxito. El campo CurrentState contiene un objeto con información sobre el estado actual de la instancia. En este caso, el Code es 32 y el Name es ```"shutting-down"```, lo que indica que la instancia está en proceso de apagado.

