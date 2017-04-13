# Documentation

Based on 'Dcoumentation - PHP' at https://docs.openshift.org/latest/using_images/s2i_images/php.html

## Source-to-Image (S2I)

OpenShift Origin provides S2I enabled PHP images for building and running PHP applications. The PHP S2I builder image assembles your application source with any required dependencies to create a new image containing your PHP application. This resulting image can be run either by OpenShift Origin or by Docker.

NOTE: S2I is prefered over a Docker file, because by restricting build operations as an S2I facilitates instead of allowing arbitrary actions, as a Dockerfile would allow, the PaaS operator can avoid accidental or intentional abuses of the build system.

## PHP

Currently, OpenShift Origin provides version ***5.5*** and ***5.6*** of PHP.

## Image

Images can be referenced in image streams.

## Image Stream

An image stream comprises any number of container images identified by tags. It presents a single virtual view of related images, similar to a Docker image repository. 

See https://docs.openshift.org/latest/dev_guide/managing_images.html#dev-guide-managing-images

An image stream in OpenShift Origin comprises zero or more container images identified by tags.

## Image Stream Definitions

The PHP image supports a number of environment variables which can be set to control the configuration and behavior of the PHP runtime.

To set these environment variables as part of your image, you can place them into a .s2i/environment file inside your source code repository.

See also https://docs.openshift.org/latest/dev_guide/builds/build_strategies.html#environment-files

Example .s2i/environment :

```javascript
to do: provide an example
```

If you provide a .s2i/environment file in your source repository, S2I reads this file during the build. This allows customization of the build behavior as the assemble script may use these variables.

## Quick Start Templates

See https://docs.openshift.org/latest/dev_guide/app_tutorials/quickstarts.html

A quickstart is a basic example of an application running on OpenShift Origin. Quickstarts come in a variety of languages and frameworks, and are defined in a template, which is constructed from a set of services, build configurations, and deployment configurations. 

## Web Framework Quickstart Templates

These quickstarts provide a basic application of the indicated framework and language:

- CakePHP: a PHP web framework (includes a MySQL database)

- Dancer: a Perl web framework (includes a MySQL database)

- Django: a Python web framework (includes a PostgreSQL database)

- NodeJS: a NodeJS web application (includes a MongoDB database)

- Rails: a Ruby web framework (includes a PostgreSQL database)

### Web Framework Quickstart Template of CakePHP

Source: https://github.com/openshift/origin/blob/master/examples/quickstarts/cakephp-mysql.json

***openshift/templates/cakephp-mysql.json***

```javascript
{
  "kind": "Template",
  "apiVersion": "v1",
  "metadata": {
    "name": "cakephp-mysql-example",
    "annotations": {
      "openshift.io/display-name": "CakePHP + MySQL (Ephemeral)",
      "description": "An example CakePHP application with a MySQL database. For more information about using this template, including OpenShift considerations, see https://github.com/openshift/cakephp-ex/blob/master/README.md.\n\nWARNING: Any data stored will be lost upon pod destruction. Only use this template for testing.",
      "tags": "quickstart,php,cakephp",
      "iconClass": "icon-php",
      "template.openshift.io/long-description": "This template defines resources needed to develop a CakePHP application, including a build configuration, application deployment configuration, and database deployment configuration.  The database is stored in non-persistent storage, so this configuration should be used for experimental purposes only.",
      "template.openshift.io/provider-display-name": "Red Hat, Inc.",
      "template.openshift.io/documentation-url": "https://github.com/openshift/cakephp-ex",
      "template.openshift.io/support-url": "https://access.redhat.com"
    }
  },
  "message": "The following service(s) have been created in your project: ${NAME}, ${DATABASE_SERVICE_NAME}.\n\nFor more information about using this template, including OpenShift considerations, see https://github.com/openshift/cake-ex/blob/master/README.md.",
  "labels": {
    "template": "cakephp-mysql-example"
  },
  "objects": [
    {
      "kind": "Secret",
      "apiVersion": "v1",
      "metadata": {
        "name": "${NAME}"
      },
      "stringData" : {
        "database-user" : "${DATABASE_USER}",
        "database-password" : "${DATABASE_PASSWORD}",
        "cakephp-secret-token" : "${CAKEPHP_SECRET_TOKEN}",
        "cakephp-security-salt" : "${CAKEPHP_SECURITY_SALT}",
        "cakephp-security-cipher-seed" : "${CAKEPHP_SECURITY_CIPHER_SEED}"
      }
    },
    {
      "kind": "Service",
      "apiVersion": "v1",
      "metadata": {
        "name": "${NAME}",
        "annotations": {
          "description": "Exposes and load balances the application pods",
          "service.alpha.openshift.io/dependencies": "[{\"name\": \"${DATABASE_SERVICE_NAME}\", \"kind\": \"Service\"}]"
        }
      },
      "spec": {
        "ports": [
          {
            "name": "web",
            "port": 8080,
            "targetPort": 8080
          }
        ],
        "selector": {
          "name": "${NAME}"
        }
      }
    },
    {
      "kind": "Route",
      "apiVersion": "v1",
      "metadata": {
        "name": "${NAME}"
      },
      "spec": {
        "host": "${APPLICATION_DOMAIN}",
        "to": {
          "kind": "Service",
          "name": "${NAME}"
        }
      }
    },
    {
      "kind": "ImageStream",
      "apiVersion": "v1",
      "metadata": {
        "name": "${NAME}",
        "annotations": {
          "description": "Keeps track of changes in the application image"
        }
      }
    },
    {
      "kind": "BuildConfig",
      "apiVersion": "v1",
      "metadata": {
        "name": "${NAME}",
        "annotations": {
          "description": "Defines how to build the application"
        }
      },
      "spec": {
        "source": {
          "type": "Git",
          "git": {
            "uri": "${SOURCE_REPOSITORY_URL}",
            "ref": "${SOURCE_REPOSITORY_REF}"
          },
          "contextDir": "${CONTEXT_DIR}"
        },
        "strategy": {
          "type": "Source",
          "sourceStrategy": {
            "from": {
              "kind": "ImageStreamTag",
              "namespace": "${NAMESPACE}",
              "name": "php:7.0"
            },
            "env":  [
              {
                "name": "COMPOSER_MIRROR",
                "value": "${COMPOSER_MIRROR}"
              }
            ]
          }
        },
        "output": {
          "to": {
            "kind": "ImageStreamTag",
            "name": "${NAME}:latest"
          }
        },
        "triggers": [
          {
            "type": "ImageChange"
          },
          {
            "type": "ConfigChange"
          },
          {
            "type": "GitHub",
            "github": {
              "secret": "${GITHUB_WEBHOOK_SECRET}"
            }
          }
        ],
        "postCommit": {
          "script": "./lib/Cake/Console/cake test app AllTests"
        }
      }
    },
    {
      "kind": "DeploymentConfig",
      "apiVersion": "v1",
      "metadata": {
        "name": "${NAME}",
        "annotations": {
          "description": "Defines how to deploy the application server"
        }
      },
      "spec": {
        "strategy": {
          "type": "Recreate",
          "recreateParams": {
            "pre": {
              "failurePolicy": "Retry",
              "execNewPod": {
                "command": [
                  "./migrate-database.sh"
                ],
                "containerName": "cakephp-mysql-example"
              }
            }
          }
        },
        "triggers": [
          {
            "type": "ImageChange",
            "imageChangeParams": {
              "automatic": true,
              "containerNames": [
                "cakephp-mysql-example"
              ],
              "from": {
                "kind": "ImageStreamTag",
                "name": "${NAME}:latest"
              }
            }
          },
          {
            "type": "ConfigChange"
          }
        ],
        "replicas": 1,
        "selector": {
          "name": "${NAME}"
        },
        "template": {
          "metadata": {
            "name": "${NAME}",
            "labels": {
              "name": "${NAME}"
            }
          },
          "spec": {
            "containers": [
              {
                "name": "cakephp-mysql-example",
                "image": " ",
                "ports": [
                  {
                    "containerPort": 8080
                  }
                ],
                "readinessProbe": {
                  "timeoutSeconds": 3,
                  "initialDelaySeconds": 3,
                  "httpGet": {
                    "path": "/health.php",
                    "port": 8080
                  }
                },
                "livenessProbe": {
                  "timeoutSeconds": 3,
                  "initialDelaySeconds": 30,
                  "httpGet": {
                    "path": "/",
                    "port": 8080
                  }
                },
                "env": [
                  {
                    "name": "DATABASE_SERVICE_NAME",
                    "value": "${DATABASE_SERVICE_NAME}"
                  },
                  {
                    "name": "DATABASE_ENGINE",
                    "value": "${DATABASE_ENGINE}"
                  },
                  {
                    "name": "DATABASE_NAME",
                    "value": "${DATABASE_NAME}"
                  },
                  {
                    "name": "DATABASE_USER",
                    "valueFrom": {
                      "secretKeyRef" : {
                        "name" : "${NAME}",
                        "key" : "database-user"
                      }
                    }
                  },
                  {
                    "name": "DATABASE_PASSWORD",
                    "valueFrom": {
                      "secretKeyRef" : {
                        "name" : "${NAME}",
                        "key" : "database-password"
                      }
                    }
                  },
                  {
                    "name": "CAKEPHP_SECRET_TOKEN",
                    "valueFrom": {
                      "secretKeyRef" : {
                        "name" : "${NAME}",
                        "key" : "cakephp-secret-token"
                      }
                    }
                  },
                  {
                    "name": "CAKEPHP_SECURITY_SALT",
                    "valueFrom": {
                      "secretKeyRef" : {
                        "name" : "${NAME}",
                        "key" : "cakephp-security-salt"
                      }
                    }
                  },
                  {
                    "name": "CAKEPHP_SECURITY_CIPHER_SEED",
                    "valueFrom": {
                      "secretKeyRef" : {
                        "name" : "${NAME}",
                        "key" : "cakephp-security-cipher-seed"
                      }
                    }
                  },
                  {
                    "name": "OPCACHE_REVALIDATE_FREQ",
                    "value": "${OPCACHE_REVALIDATE_FREQ}"
                  }
                ],
                "resources": {
                  "limits": {
                    "memory": "${MEMORY_LIMIT}"
                  }
                }
              }
            ]
          }
        }
      }
    },
    {
      "kind": "Service",
      "apiVersion": "v1",
      "metadata": {
        "name": "${DATABASE_SERVICE_NAME}",
        "annotations": {
          "description": "Exposes the database server"
        }
      },
      "spec": {
        "ports": [
          {
            "name": "mysql",
            "port": 3306,
            "targetPort": 3306
          }
        ],
        "selector": {
          "name": "${DATABASE_SERVICE_NAME}"
        }
      }
    },
    {
      "kind": "DeploymentConfig",
      "apiVersion": "v1",
      "metadata": {
        "name": "${DATABASE_SERVICE_NAME}",
        "annotations": {
          "description": "Defines how to deploy the database"
        }
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
                "mysql"
              ],
              "from": {
                "kind": "ImageStreamTag",
                "namespace": "${NAMESPACE}",
                "name": "mysql:5.7"
              }
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
            "name": "${DATABASE_SERVICE_NAME}",
            "labels": {
              "name": "${DATABASE_SERVICE_NAME}"
            }
          },
          "spec": {
            "volumes": [
              {
                "name": "data",
                "emptyDir": {}
              }
            ],
            "containers": [
              {
                "name": "mysql",
                "image": " ",
                "ports": [
                  {
                    "containerPort": 3306
                  }
                ],
                "volumeMounts": [
                  {
                    "name": "data",
                    "mountPath": "/var/lib/mysql/data"
                  }
                ],
                "readinessProbe": {
                  "timeoutSeconds": 1,
                  "initialDelaySeconds": 5,
                  "exec": {
                    "command": [ "/bin/sh", "-i", "-c", "MYSQL_PWD='${DATABASE_PASSWORD}' mysql -h 127.0.0.1 -u ${DATABASE_USER} -D ${DATABASE_NAME} -e 'SELECT 1'" ]
                  }
                },
                "livenessProbe": {
                  "timeoutSeconds": 1,
                  "initialDelaySeconds": 30,
                  "tcpSocket": {
                    "port": 3306
                  }
                },
                "env": [
                  {
                    "name": "MYSQL_USER",
                    "valueFrom": {
                      "secretKeyRef" : {
                        "name" : "${NAME}",
                        "key" : "database-user"
                      }
                    }
                  },
                  {
                    "name": "MYSQL_PASSWORD",
                    "valueFrom": {
                      "secretKeyRef" : {
                        "name" : "${NAME}",
                        "key" : "database-password"
                      }
                    }
                  },
                  {
                    "name": "MYSQL_DATABASE",
                    "value": "${DATABASE_NAME}"
                  }
                ],
                "resources": {
                  "limits": {
                    "memory": "${MEMORY_MYSQL_LIMIT}"
                  }
                }
              }
            ]
          }
        }
      }
    }
  ],
  "parameters": [
    {
      "name": "NAME",
      "displayName": "Name",
      "description": "The name assigned to all of the frontend objects defined in this template.",
      "required": true,
      "value": "cakephp-mysql-example"
    },
    {
      "name": "NAMESPACE",
      "displayName": "Namespace",
      "description": "The OpenShift Namespace where the ImageStream resides.",
      "required": true,
      "value": "openshift"
    },
    {
      "name": "MEMORY_LIMIT",
      "displayName": "Memory Limit",
      "description": "Maximum amount of memory the CakePHP container can use.",
      "required": true,
      "value": "512Mi"
    },
    {
      "name": "MEMORY_MYSQL_LIMIT",
      "displayName": "Memory Limit (MySQL)",
      "description": "Maximum amount of memory the MySQL container can use.",
      "required": true,
      "value": "512Mi"
    },
    {
      "name": "SOURCE_REPOSITORY_URL",
      "displayName": "Git Repository URL",
      "description": "The URL of the repository with your application source code.",
      "required": true,
      "value": "https://github.com/openshift/cakephp-ex.git"
    },
    {
      "name": "SOURCE_REPOSITORY_REF",
      "displayName": "Git Reference",
      "description": "Set this to a branch name, tag or other ref of your repository if you are not using the default branch."
    },
    {
      "name": "CONTEXT_DIR",
      "displayName": "Context Directory",
      "description": "Set this to the relative path to your project if it is not in the root of your repository."
    },
    {
      "name": "APPLICATION_DOMAIN",
      "displayName": "Application Hostname",
      "description": "The exposed hostname that will route to the CakePHP service, if left blank a value will be defaulted.",
      "value": ""
    },
    {
      "name": "GITHUB_WEBHOOK_SECRET",
      "displayName": "GitHub Webhook Secret",
      "description": "A secret string used to configure the GitHub webhook.",
      "generate": "expression",
      "from": "[a-zA-Z0-9]{40}"
    },
    {
      "name": "DATABASE_SERVICE_NAME",
      "displayName": "Database Service Name",
      "required": true,
      "value": "mysql"
    },
    {
      "name": "DATABASE_ENGINE",
      "displayName": "Database Engine",
      "description": "Database engine: postgresql, mysql or sqlite (default).",
      "required": true,
      "value": "mysql"
    },
    {
      "name": "DATABASE_NAME",
      "displayName": "Database Name",
      "required": true,
      "value": "default"
    },
    {
      "name": "DATABASE_USER",
      "displayName": "Database User",
      "required": true,
      "value": "cakephp"
    },
    {
      "name": "DATABASE_PASSWORD",
      "displayName": "Database Password",
      "generate": "expression",
      "from": "[a-zA-Z0-9]{16}"
    },
    {
      "name": "CAKEPHP_SECRET_TOKEN",
      "displayName": "CakePHP secret token",
      "description": "Set this to a long random string.",
      "generate": "expression",
      "from": "[\\w]{50}"
    },
    {
      "name": "CAKEPHP_SECURITY_SALT",
      "displayName": "CakePHP Security Salt",
      "description": "Security salt for session hash.",
      "generate": "expression",
      "from": "[a-zA-Z0-9]{40}"
    },
    {
      "name": "CAKEPHP_SECURITY_CIPHER_SEED",
      "displayName": "CakePHP Security Cipher Seed",
      "description": "Security cipher seed for session hash.",
      "generate": "expression",
      "from": "[0-9]{30}"
    },
    {
      "name": "OPCACHE_REVALIDATE_FREQ",
      "displayName": "OPcache Revalidation Frequency",
      "description": "How often to check script timestamps for updates, in seconds. 0 will result in OPcache checking for updates on every request.",
      "value": "2"
    },
    {
      "name": "COMPOSER_MIRROR",
      "displayName": "Custom Composer Mirror URL",
      "description": "The custom Composer mirror URL",
      "value": ""
    }
  ]
}
```

### Custom Template

Say you want to use your OWN template, then write it similar to the example above and import it in OpenShift when adding to the project (using the tab 'Import YAML/JSON'). The above example is in the JSON format.

***openshift/templates/cakephp-mysql-custom.json***

```javascript
{
  "kind": "Template",
  "apiVersion": "v1",
  "metadata": {
    "name": "cakephp-mysql-custom",
    // etcetera
  }
}  
```