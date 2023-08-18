# Use cases

This document describes the functionalities of the entire application.

## Authentication

|  Documentation  |                             URL                              |
| :-------------: | :----------------------------------------------------------: |
|   Repository    | [authentication-go](https://github.com/hawks-atlanta/authentication-go) |
| Database Models |             [Models](Database.md#Authentication)             |
|     OpenAPI     | [Specification](https://github.com/hawks-atlanta/authentication-go/docs/spec.openapi.yaml) |

### Authorization

<p style="text-align: center"><img src="assets/use-case-user-auth.svg" alt="use-case-user-auth" /></p>
<p style="text-align: center"><img src="assets/use-case-gw-ch.svg" alt="use-case-gw-ch" /></p>

### Account

<p style="text-align: center"><img src="assets/use-case-user-acc.svg" alt="use-case-user-acc" /></p>


## Metadata

|  Documentation  |                             URL                              |
| :-------------: | :----------------------------------------------------------: |
|   Repository    | [metadata-scala](https://github.com/hawks-atlanta/metadata-scala) |
| Database Models |                [Models](Database.md#Metadata)                |
|      Logic      |                   [docs](docs/Metadata.md)                   |
|     OpenAPI     | [Specification](https://github.com/hawks-atlanta/metadata-scala/docs/spec.openapi.yaml) |

### Files index

<p style="text-align: center"><img src="assets/use-case-meta-user.svg" alt="use-case-meta-user" /></p>

### Worker uploads

<p style="text-align: center"><img src="assets/use-case-worker-upload.svg" alt="use-case-worker-upload" /></p>


## Worker

|  Documentation  |                             URL                             |
| :-------------: | :---------------------------------------------------------: |
|   Repository    | [worker-java](https://github.com/hawks-atlanta/worker-java) |
| Database Models |                [Models](Database.md#Worker)                 |
|      Logic      |                   [docs](docs/Worker.md)                    |

### Filesystem

<p style="text-align: center"><img src="assets/use-case-worker-fs.svg" alt="use-case-worker-fs" /></p>


### Gateway

The gateway by itself is a proxy that manages the access to the rest of the microservices running in the cluster. So its functionality is the **UNION** of all **Use cases** mentioned before plus the combination of the AUTH check.

Read [Gateway.md](docs/Gateway.md) for more information.
