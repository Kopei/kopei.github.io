---
layout: post
title: Openshift Template
categories: [openshift, kubernetes]
description: 
keywords: openshift
catalog: true
multilingual: false
tags: kubernetes, openshift
date: 2018-01-04
---

### Openshift的模板
定义：模板是一组可以被参数化的对象，这组对象被处理后可以被用于创建服务，构建配置，部署配置。

### 模板代码例子
```yaml
apiVersion: v1
kind: Template
metadata:
  name: redis-template #1
  annotations:
    description: "Description"  #2
    iconClass: "icon-redis"  #3
    tags: "database,nosql"  #4
objects:   #5
- apiVersion: v1
  kind: Pod
  metadata:
    name: redis-master
  spec:
    containers:
    - env:
      - name: REDIS_PASSWORD
        value: ${REDIS_PASSWORD}   #6
      image: dockerfile/redis
      name: master
      ports:
      - containerPort: 6379
        protocol: TCP
parameters:  #7
- description: Password used for Redis authentication
  from: '[A-Z0-9]{8}'   
  generate: expression
  name: REDIS_PASSWORD
labels:      
  redis: master
```
1. 模板名
2. 可选描述
3. 展示的icon（搜索"openshift-logos-icon"）
4. 这个模板的tag
5. 模板将会创建的对象
6. 模板处理的时候输入参数
7. 模板参数
8. 任意产生表达式
9. 所以对象将被打上标签

### 复杂一点的json模板
```json
{
  "kind": "Template",
  "apiVersion": "v1",
  "metadata": {
    "name": "postgresql-persistent",
    "creationTimestamp": null,
    "annotations": {
      "openshift.io/display-name": "PostgreSQL (Persistent)",
      "description": "PostgreSQL database service, with persistent storage. For more information about using this template, including OpenShift considerations, see https://github.com/sclorg/postgresql-container/blob/master/9.5.\n\nNOTE: Scaling to more than one replica is not supported. You must have persistent volumes available in your cluster to use this template.",
      "iconClass": "icon-postgresql",
      "tags": "database,postgresql",
      "template.openshift.io/long-description": "This template provides a standalone PostgreSQL server with a database created.  The database is stored on persistent storage.  The database name, username, and password are chosen via parameters when provisioning this service.",
      "template.openshift.io/provider-display-name": "Red Hat, Inc.",
      "template.openshift.io/documentation-url": "https://docs.openshift.org/latest/using_images/db_images/postgresql.html",
      "template.openshift.io/support-url": "https://access.redhat.com"
    }
  },
  "message": "The following service(s) have been created in your project: ${DATABASE_SERVICE_NAME}.\n\n       Username: ${POSTGRESQL_USER}\n       Password: ${POSTGRESQL_PASSWORD}\n  Database Name: ${POSTGRESQL_DATABASE}\n Connection URL: postgresql://${DATABASE_SERVICE_NAME}:5432/\n\nFor more information about using this template, including OpenShift considerations, see https://github.com/sclorg/postgresql-container/blob/master/9.5.",
  "labels": {
    "template": "postgresql-persistent-template"
  },
  "objects": [
    {
      "kind": "Secret",
      "apiVersion": "v1",
      "metadata": {
        "name": "${DATABASE_SERVICE_NAME}",
        "annotations": {
          "template.openshift.io/expose-username": "{.data['database-user']}",
          "template.openshift.io/expose-password": "{.data['database-password']}"
        }
      },
      "stringData" : {
        "database-user" : "${POSTGRESQL_USER}",
        "database-password" : "${POSTGRESQL_PASSWORD}"
      }
    },
    {
      "kind": "Service",
      "apiVersion": "v1",
      "metadata": {
        "name": "${DATABASE_SERVICE_NAME}",
        "creationTimestamp": null,
        "annotations": {
          "template.openshift.io/expose-uri": "postgres://{.spec.clusterIP}:{.spec.ports[?(.name==\"postgresql\")].port}"
        }
      },
      "spec": {
        "ports": [
          {
            "name": "postgresql",
            "protocol": "TCP",
            "port": 5432,
            "targetPort": 5432,
            "nodePort": 0
          }
        ],
        "selector": {
          "name": "${DATABASE_SERVICE_NAME}"
        },
        "type": "ClusterIP",
        "sessionAffinity": "None"
      },
      "status": {
        "loadBalancer": {}
      }
    },
    {
      "kind": "PersistentVolumeClaim",
      "apiVersion": "v1",
      "metadata": {
        "name": "${DATABASE_SERVICE_NAME}"
      },
      "spec": {
        "accessModes": [
          "ReadWriteOnce"
        ],
        "resources": {
          "requests": {
            "storage": "${VOLUME_CAPACITY}"
          }
        }
      }
    },
    {
      "kind": "DeploymentConfig",
      "apiVersion": "v1",
      "metadata": {
        "name": "${DATABASE_SERVICE_NAME}",
        "creationTimestamp": null
      },
      "spec": {
        "strategy": {
          "type": "Recreate"
        },
        "triggers": [
          {
            "type": "ImageChange",
            "imageChangeParams": {
              "automatic": true,
              "containerNames": [
                "postgresql"
              ],
              "from": {
                "kind": "ImageStreamTag",
                "name": "postgresql:${POSTGRESQL_VERSION}",
                "namespace": "${NAMESPACE}"
              },
              "lastTriggeredImage": ""
            }
          },
          {
            "type": "ConfigChange"
          }
        ],
        "replicas": 1,
        "selector": {
          "name": "${DATABASE_SERVICE_NAME}"
        },
        "template": {
          "metadata": {
            "creationTimestamp": null,
            "labels": {
              "name": "${DATABASE_SERVICE_NAME}"
            }
          },
          "spec": {
            "containers": [
              {
                "name": "postgresql",
                "image": " ",
                "ports": [
                  {
                    "containerPort": 5432,
                    "protocol": "TCP"
                  }
                ],
                "readinessProbe": {
                  "timeoutSeconds": 1,
                  "initialDelaySeconds": 5,
                  "exec": {
                    "command": [ "/bin/sh", "-i", "-c", "psql -h 127.0.0.1 -U $POSTGRESQL_USER -q -d $POSTGRESQL_DATABASE -c 'SELECT 1'"]
                  }
                },
                "livenessProbe": {
                  "timeoutSeconds": 1,
                  "initialDelaySeconds": 30,
                  "tcpSocket": {
                    "port": 5432
                  }
                },
                "env": [
                  {
                    "name": "POSTGRESQL_USER",
                    "valueFrom": {
                      "secretKeyRef" : {
                        "name" : "${DATABASE_SERVICE_NAME}",
                        "key" : "database-user"
                      }
                    }
                  },
                  {
                    "name": "POSTGRESQL_PASSWORD",
                    "valueFrom": {
                      "secretKeyRef" : {
                        "name" : "${DATABASE_SERVICE_NAME}",
                        "key" : "database-password"
                      }
                    }
                  },
                  {
                    "name": "POSTGRESQL_DATABASE",
                    "value": "${POSTGRESQL_DATABASE}"
                  }
                ],
                "resources": {
                  "limits": {
                    "memory": "${MEMORY_LIMIT}"
                  }
                },
                "volumeMounts": [
                  {
                    "name": "${DATABASE_SERVICE_NAME}-data",
                    "mountPath": "/var/lib/pgsql/data"
                  }
                ],
                "terminationMessagePath": "/dev/termination-log",
                "imagePullPolicy": "IfNotPresent",
                "capabilities": {},
                "securityContext": {
                  "capabilities": {},
                  "privileged": false
                }
              }
            ],
            "volumes": [
              {
                "name": "${DATABASE_SERVICE_NAME}-data",
                "persistentVolumeClaim": {
                  "claimName": "${DATABASE_SERVICE_NAME}"
                }
              }
            ],
            "restartPolicy": "Always",
            "dnsPolicy": "ClusterFirst"
          }
        }
      },
      "status": {}
    }
  ],
  "parameters": [
    {
      "name": "MEMORY_LIMIT",
      "displayName": "Memory Limit",
      "description": "Maximum amount of memory the container can use.",
      "value": "512Mi",
      "required": true
    },
    {
      "name": "NAMESPACE",
      "displayName": "Namespace",
      "description": "The OpenShift Namespace where the ImageStream resides.",
      "value": "openshift"
    },
    {
      "name": "DATABASE_SERVICE_NAME",
      "displayName": "Database Service Name",
      "description": "The name of the OpenShift Service exposed for the database.",
      "value": "postgresql",
      "required": true
    },
    {
      "name": "POSTGRESQL_USER",
      "displayName": "PostgreSQL Connection Username",
      "description": "Username for PostgreSQL user that will be used for accessing the database.",
      "generate": "expression",
      "from": "user[A-Z0-9]{3}",
      "required": true
    },
    {
      "name": "POSTGRESQL_PASSWORD",
      "displayName": "PostgreSQL Connection Password",
      "description": "Password for the PostgreSQL connection user.",
      "generate": "expression",
      "from": "[a-zA-Z0-9]{16}",
      "required": true
    },
    {
      "name": "POSTGRESQL_DATABASE",
      "displayName": "PostgreSQL Database Name",
      "description": "Name of the PostgreSQL database accessed.",
      "value": "sampledb",
      "required": true
    },
    {
      "name": "VOLUME_CAPACITY",
      "displayName": "Volume Capacity",
      "description": "Volume space available for data, e.g. 512Mi, 2Gi.",
      "value": "1Gi",
      "required": true
    },
    {
      "name": "POSTGRESQL_VERSION",
      "displayName": "Version of PostgreSQL Image",
      "description": "Version of PostgreSQL image to be used (9.2, 9.4, 9.5 or latest).",
      "value": "9.5",
      "required": true
    }
  ]
}
```