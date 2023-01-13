# Домашнее задание №7

## Знакомство с ElasticSearch.

### Цель:

1. Научитесь разворачивать ES в AWS и использовать полнотекстовый нечеткий поиск.

### Необходимо:

1. Развернуть Instance ES – желательно в AWS
2. Создать в ES индекс, в нём должно быть обязательное поле text типа string
3. Создать для индекса pattern
4. Добавить в индекс как минимум 3 документа желательно со следующим содержанием:

- "моя мама мыла посуду а кот жевал сосиски"
- "рама была отмыта и вылизана котом"
- "мама мыла раму"

5. Написать запрос нечеткого поиска к этой коллекции документов ко ключу "мама ела сосиски"
6. Расшарить коллекцию postman (желательно сдавать в таком формате)
7. Прислать ссылку на коллекцию

### Решение:

1.1) **Запуск ec2 инстанса в aws**

```
aws ec2 run-instances \
    --image-id ami-06878d265978313ca \
    --instance-type t2.micro \
    --key-name mykeyname \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=otus-elastic-search-hw}]'            
```

<details>
<summary>Output</summary><pre>
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-06878d265978313ca",
            "InstanceId": "**************",
            "InstanceType": "t2.micro",
            "KeyName": "mykeyname",
            "LaunchTime": "2023-01-12T03:56:31.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-1b",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-***-**-**-**.ec2.internal",
            "PrivateIpAddress": "*.*.*.*",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-**************",
            "VpcId": "vpc-*******************",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "*-*-*-*-*",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2023-01-12T03:56:31.000Z",
                        "AttachmentId": "eni-attach-*******************",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching"
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "default",
                            "GroupId": "sg-******************"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "*:*:*:*:*:*",
                    "NetworkInterfaceId": "eni-*******************",
                    "OwnerId": "**************",
                    "PrivateDnsName": "ip-*-*-*-*.ec2.internal",
                    "PrivateIpAddress": "*.*.*.*",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-*-*-*-*.ec2.internal",
                            "PrivateIpAddress": "*.*.*.*"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-**************",
                    "VpcId": "vpc-*******************",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/sda1",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "default",
                    "GroupId": "sg-******************"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "otus-elastic-search-hw"
                }
            ],
            "VirtualizationType": "hvm",
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
                "HttpEndpoint": "enabled"
            }
        }
    ],
    "OwnerId": "**************",
    "ReservationId": "r-*******************"
}
</pre></details>



1.2) **Настраиваем security group для доступа к инстансу**

![This is an image](https://raw.githubusercontent.com/spendmail/otus_nosql_hw/main/docs/07_elasticsearch/screenshots/security_group_set_up.png)

2.1) **Установка ElasticSearch**

```
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic.gpg
echo "deb [signed-by=/usr/share/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
sudo apt update
sudo apt install -y elasticsearch
```

<details>
<summary>Output</summary><pre>
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
elasticsearch
0 upgraded, 1 newly installed, 0 to remove and 43 not upgraded.
Need to get 315 MB of archives.
After this operation, 526 MB of additional disk space will be used.
Get:1 https://artifacts.elastic.co/packages/7.x/apt stable/main amd64 elasticsearch amd64 7.17.8 [315 MB]
Fetched 315 MB in 7s (48.1 MB/s)                                                                                                                                                                          
Selecting previously unselected package elasticsearch.
(Reading database ... 63919 files and directories currently installed.)
Preparing to unpack .../elasticsearch_7.17.8_amd64.deb ...
Creating elasticsearch group... OK
Creating elasticsearch user... OK
Unpacking elasticsearch (7.17.8) ...
Setting up elasticsearch (7.17.8) ...
### NOT starting on installation, please execute the following statements to configure elasticsearch service to start automatically using systemd
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
### You can start elasticsearch service by executing
sudo systemctl start elasticsearch.service
Created elasticsearch keystore in /etc/elasticsearch/elasticsearch.keystore
Scanning processes...                                                                                                                                                                                      
Scanning linux images...

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
</pre></details>



2.2) **Конфигурирование и запуск ElasticSearch**

```
echo "network.host: 0.0.0.0" | sudo tee --append /etc/elasticsearch/elasticsearch.yml
echo "discovery.type: single-node" | sudo tee --append /etc/elasticsearch/elasticsearch.yml
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

<details>
<summary>Output</summary><pre>
network.host: 0.0.0.0
discovery.type: single-node
Synchronizing state of elasticsearch.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable elasticsearch
Created symlink /etc/systemd/system/multi-user.target.wants/elasticsearch.service → /lib/systemd/system/elasticsearch.service.
</pre></details>



2.3-3) **Создание индекса**

```
curl \
--request PUT \
--header 'Content-Type: application/json' \
--data '{"mappings":{"properties":{"text":{"type":"text","fields":{"keyword":{"type":"keyword","ignore_above":256}}}}}}' \
http://107.21.153.46:9200/otus-hw
```

<details>
<summary>Output</summary><pre>
{"acknowledged":true,"shards_acknowledged":true,"index":"otus-hw"}
</pre></details>


4.1) **Добавление документов**

```
curl \
--request POST \
--header 'Content-Type: application/json' \
--data-binary '@bulk.json' \
http://107.21.153.46:9200/_bulk
```

<details>
<summary>Output</summary><pre>
{
  "took": 9,
  "errors": false,
  "items": [
    {
      "create": {
        "_index": "otus-hw",
        "_type": "_doc",
        "_id": "1",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 0,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "create": {
        "_index": "otus-hw",
        "_type": "_doc",
        "_id": "2",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1,
        "status": 201
      }
    },
    {
      "create": {
        "_index": "otus-hw",
        "_type": "_doc",
        "_id": "3",
        "_version": 1,
        "result": "created",
        "_shards": {
          "total": 2,
          "successful": 1,
          "failed": 0
        },
        "_seq_no": 2,
        "_primary_term": 1,
        "status": 201
      }
    }
  ]
}
</pre></details>



5.1) **Создание запроса**

```
curl \
--request GET \
--header 'Content-Type: application/json' \
--data '{"query":{"match":{"text":"мама ела сосиски"}}}' \
http://107.21.153.46:9200/otus-hw/_search
```

<details>
<summary>Output</summary><pre>
{
  "took": 608,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 2,
      "relation": "eq"
    },
    "max_score": 1.241674,
    "hits": [
      {
        "_index": "otus-hw",
        "_type": "_doc",
        "_id": "1",
        "_score": 1.241674,
        "_source": {
          "text": "моя мама мыла посуду а кот жевал сосиски"
        }
      },
      {
        "_index": "otus-hw",
        "_type": "_doc",
        "_id": "3",
        "_score": 0.5820575,
        "_source": {
          "text": "мама мыла раму"
        }
      }
    ]
  }
}
</pre></details>
