# Parte 1: ELB
### 1. Inicia sesión en el sandbox del curso AWS . Ve al directorio donde guarda el archivo de script de instalación de apache del laboratorio de la práctica calificada 3. Para crear un balanceador de carga, haz lo siguiente.

`aws elb create-load-balancer --load-balancer-name LeonardoMejia --listeners "Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol= HTTP,InstancePort=80" --availability-zones us-east-1d`

![image](https://github.com/LeonardoCam/DatosRedesLeonardoMejia/assets/89529410/db0a27fd-088b-4441-b708-4c8b38d5a1a0)

**_¿Cuál es el DNS_Name del balanceador de carga?_**

`"DNSName": "LeonardoMejia-475345717.us-east-1.elb.amazonaws.com"`

### 2. El comando describe-load-balancers describe el estado y las propiedades de tu(s) balanceador(es) de carga. Presenta este comando:

`aws elb describe-load-balancers --load-balancer-name LeonardoMejia`

**_¿Cuál es la salida?_**

```
> {
>     "LoadBalancerDescriptions": [
>         {
>             "LoadBalancerName": "LeonardoMejia",
>             "DNSName": "LeonardoMejia-475345717.us-east-1.elb.amazonaws.com",
>             "CanonicalHostedZoneName": "LeonardoMejia-475345717.us-east-1.elb.amazonaws.com",
>             "CanonicalHostedZoneNameID": "Z35SXDOTRQ7X7K",
>             "ListenerDescriptions": [
>                 {
>                     "Listener": {
>                         "Protocol": "HTTP",
>                         "LoadBalancerPort": 80,
> :...skipping...
> {
>     "LoadBalancerDescriptions": [
>         {
>             "LoadBalancerName": "LeonardoMejia",
>             "DNSName": "LeonardoMejia-475345717.us-east-1.elb.amazonaws.com",
>             "CanonicalHostedZoneName": "LeonardoMejia-475345717.us-east-1.elb.amazonaws.com",
>             "CanonicalHostedZoneNameID": "Z35SXDOTRQ7X7K",
>             "ListenerDescriptions": [
>                 {
>                     "Listener": {
>                         "Protocol": "HTTP",
>                         "LoadBalancerPort": 80,
>                         "InstanceProtocol": "HTTP",
>                         "InstancePort": 80
>                     },
>                     "PolicyNames": []
>                 }
>             ],
>             "Policies": {
>                 "AppCookieStickinessPolicies": [],
>                 "LBCookieStickinessPolicies": [],
>                 "OtherPolicies": []
>             },
>             "BackendServerDescriptions": [],
>             "AvailabilityZones": [
>                 "us-east-1d"
>             ],
>             "Subnets": [
>                 "subnet-0f7b2b840a7204f3f"
>             ],
>             "VPCId": "vpc-08064bb3a9a79a780",
>             "Instances": [],
>             "HealthCheck": {
>                 "Target": "TCP:80",
>                 "Interval": 30,
>                 "Timeout": 5,
>                 "UnhealthyThreshold": 2,
>                 "HealthyThreshold": 10
>             },
>             "SourceSecurityGroup": {
>                 "OwnerAlias": "948770255140",
>                 "GroupName": "default_elb_60905890-2b8b-3829-995f-e217c5376bc4"
>             },
>             "SecurityGroups": [
>                 "sg-0b1df3beee9b6c646"
>             ],
>             "CreatedTime": "2023-06-20T04:33:11.820000+00:00",
>             "Scheme": "internet-facing"
>         }
>     ]
> }
> (END)
> 
```

### 3. Creamos dos instancias EC2, cada una ejecutando un servidor web Apache. Emite lo siguiente.

`aws ec2 run-instances --image-id ami-d9a98cb0 --count 2 --instance-type t1.micro --key-name leonardo_mejia-key --security-groups leonardo_mejia --user-data file:/etc/apache2/apache2.conf --placement AvailabilityZone=us-east-1d`

**_¿Qué parte de este comando indica que deseas dos instancias EC2?_** 
`--count 2`

**_¿Qué parte de este comando garantiza que tus instancias tendrán Apache instalado?_** 
`file:/etc/apache2/apache2.conf
`

_**¿Cuál es el ID de instancia de la primera instancia?**_ 
`i-0799a56bfd6c29da2
`

**_¿Cuál es el ID de instancia de la segunda instancia?_**
`i-0cba5896e9c11c065
`

### 4. Para usar ELB, tenemos que registrar las instancias EC2. Haz lo siguiente, donde instance1_id e instance2_id son los obtenidos del comando en el paso 3.

`aws elb register-instances-with-load-balancer --load-balancer-name LeonardoMejia --instances i-0799a56bfd6c29da2 i-0cba5896e9c11c065`

**_¿Cuál es la salida?_** 

```
> {
>     "Instances": [
>         {
>             "InstanceId": "i-0cba5896e9c11c065"
>         },
>         {
>             "InstanceId": "i-0799a56bfd6c29da2"
>         }
>     ]
> }
```
**_Ahora vea el estado de la instancia de los servidores cuya carga se equilibra._** 

`aws elb describe-instance-health --load-balancer-name LeonardoMejia
`
**_¿Cuál es la salida?_** 

![image](https://github.com/LeonardoCam/DatosRedesLeonardoMejia/assets/89529410/e973a750-454d-4a37-96f8-e598c4ce4f32)

### 5. Abre el navegador del sandox. Recupera la dirección IP de tu balanceador de carga del paso 1, ingresa la URL http://nombre_dns_de_tu_balanceador_carga/ en tu navegador web. 

`http://leonardomejia-475345717.us-east-1.elb.amazonaws.com/`

¿Qué apareció en el navegador?

![image](https://github.com/LeonardoCam/DatosRedesLeonardoMejia/assets/89529410/747d66b3-9162-4212-90a5-daf02d9f4c16)

## Parte 2: CloudWatch

### 7. CloudWatch se utiliza para monitorear instancias. En este caso, queremos monitorear los dos servidores web. Inicia CloudWatch de la siguiente manera.

`aws ec2 monitor-instances --instance-ids i-0799a56bfd6c29da2 i-0cba5896e9c11c065`

**_¿Cuál es la salida?_**

```
> [cloudshell-user@ip-10-6-104-226 ~]$ aws ec2 monitor-instances --instance-ids i-0799a56bfd6c29da2 i-0cba5896e9c11c065
> {
>     "InstanceMonitorings": [
>         {
>             "InstanceId": "i-0799a56bfd6c29da2",
>             "Monitoring": {
>                 "State": "pending"
>             }
>         },
>         {
>             "InstanceId": "i-0cba5896e9c11c065",
>             "Monitoring": {
>                 "State": "pending"
>             }
>         }
>     ]
> }
> (END)
> 
```

**_Ahora examina las métricas disponibles con lo siguiente:_**

`aws cloudwatch list-metrics --namespaces "AWS/EC2"`

```
>        {
>             "Namespace": "AWS/EC2",
>             "MetricName": "MetadataNoToken",
>             "Dimensions": [
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkPacketsOut",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "CPUUtilization",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkIn",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkOut",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadBytes",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteBytes",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadOps",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteOps",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "CPUCreditUsage",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "MetadataNoToken",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0cba5896e9c11c065"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "StatusCheckFailed_System",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "StatusCheckFailed_Instance",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "StatusCheckFailed",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0799a56bfd6c29da2"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "StatusCheckFailed_System",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0cba5896e9c11c065"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "StatusCheckFailed_Instance",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0cba5896e9c11c065"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "StatusCheckFailed",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-0cba5896e9c11c065"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "MetadataNoToken",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-03a829eeada14a86b"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "MetadataNoToken",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-02b002abeafa1ae81"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkPacketsIn",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-03a829eeada14a86b"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkPacketsOut",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-03a829eeada14a86b"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "CPUUtilization",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-03a829eeada14a86b"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkIn",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-03a829eeada14a86b"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkOut",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-03a829eeada14a86b"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadBytes",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-03a829eeada14a86b"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteBytes",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-03a829eeada14a86b"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadOps",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-03a829eeada14a86b"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteOps",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-03a829eeada14a86b"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkPacketsIn",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-02b002abeafa1ae81"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkPacketsOut",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-02b002abeafa1ae81"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "CPUUtilization",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-02b002abeafa1ae81"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkIn",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-02b002abeafa1ae81"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkOut",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-02b002abeafa1ae81"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadBytes",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-02b002abeafa1ae81"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteBytes",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-02b002abeafa1ae81"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadOps",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-02b002abeafa1ae81"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteOps",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-02b002abeafa1ae81"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteOps",
>             "Dimensions": [
>                 {
>                     "Name": "ImageId",
>                     "Value": "ami-d9a98cb0"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteOps",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceType",
>                     "Value": "t1.micro"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteOps",
>             "Dimensions": []
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "CPUUtilization",
>             "Dimensions": [
>                 {
>                     "Name": "ImageId",
>                     "Value": "ami-d9a98cb0"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "CPUUtilization",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceType",
>                     "Value": "t1.micro"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "CPUUtilization",
>             "Dimensions": []
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkIn",
>             "Dimensions": [
>                 {
>                     "Name": "ImageId",
>                     "Value": "ami-d9a98cb0"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkIn",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceType",
>                     "Value": "t1.micro"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkIn",
>             "Dimensions": []
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkOut",
>             "Dimensions": [
>                 {
>                     "Name": "ImageId",
>                     "Value": "ami-d9a98cb0"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkOut",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceType",
>                     "Value": "t1.micro"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "NetworkOut",
>             "Dimensions": []
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadBytes",
>             "Dimensions": [
>                 {
>                     "Name": "ImageId",
>                     "Value": "ami-d9a98cb0"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadBytes",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceType",
>                     "Value": "t1.micro"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadBytes",
>             "Dimensions": []
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteBytes",
>             "Dimensions": [
>                 {
>                     "Name": "ImageId",
>                     "Value": "ami-d9a98cb0"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteBytes",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceType",
>                     "Value": "t1.micro"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskWriteBytes",
>             "Dimensions": []
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadOps",
>             "Dimensions": [
>                 {
>                     "Name": "ImageId",
>                     "Value": "ami-d9a98cb0"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadOps",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceType",
>                     "Value": "t1.micro"
>                 }
>             ]
>         },
>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "DiskReadOps",
>             "Dimensions": []
>         }
>     ]
> }
> (END)
```

**_¿Viste la métrica CPUUtilization en el resultado?_**

>         {
>             "Namespace": "AWS/EC2",
>             "MetricName": "CPUUtilization",
>             "Dimensions": [
>                 {
>                     "Name": "InstanceId",
>                     "Value": "i-03a829eeada14a86b"
>                 }
>             ]
>         }






