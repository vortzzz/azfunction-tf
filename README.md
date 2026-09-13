### **Infraestructura como código**

- **Utilizar archivos de definición**: Todas las herramientas de infraestructura como código tienen un formato propio para definir la infraestructura.
- **Autodocumentación de procesos y sistemas**: Al utilizar el enfoque de infraestructura como código, podemos reutilizar el código. Es importante que este esté documentado adecuadamente para que otros usuarios comprendan el propósito y funcionamiento del módulo.
- **Versionar todo**: Esto nos permite rastrear los cambios realizados. Si se comete un error, podemos retroceder a una versión estable.
- **Preferir cambios pequeños**: Realizar cambios pequeños para evitar grandes impactos.
- **Mantener los servicios continuamente disponibles**: Garantizar la disponibilidad continua es clave en la infraestructura.

### **Beneficios de la infraestructura como código**

- **Creación rápida y bajo demanda**: Con un único archivo de definición de infraestructura que almacena todas nuestras configuraciones, podemos crear múltiples veces la infraestructura sin necesidad de rehacer todo desde el principio.
- **Automatización**: Una vez creado el archivo de definición, podemos usar herramientas de **continuous integration** para automatizar la infraestructura.
- **Visibilidad y trazabilidad**: El versionamiento de la infraestructura como código permite una mayor visibilidad y trazabilidad, ya que todos los cambios quedan registrados.
- **Ambientes homogéneos**: Podemos crear varios ambientes a partir del mismo archivo de definición, cambiando únicamente algunos parámetros.

---

### **Mejores prácticas**

- **Modularidad**: Es recomendable dividir la infraestructura en módulos reutilizables para facilitar su mantenimiento y escalabilidad.
- **Mantener las configuraciones centralizadas**: Utilizar variables y archivos de configuración para gestionar parámetros y evitar valores "hardcoded".
- **Manejo seguro del estado**: Almacenar el archivo `terraform.tfstate` de manera remota (por ejemplo, en un bucket S3 con bloqueo de versión) para evitar problemas en equipos distribuidos.
- **Revisiones de código y pull requests**: Antes de aplicar cambios importantes en la infraestructura, hacer revisiones mediante pull requests para asegurar que los cambios han sido revisados por otros.

### **Ambientes**

Terraform permite la creación de múltiples ambientes (dev, stage, prod) con diferentes configuraciones. Puedes gestionar estos ambientes utilizando archivos `.tfvars` específicos para cada entorno.

- **Ambiente de desarrollo (dev)**: Se recomienda utilizar recursos más pequeños y económicos en este ambiente para reducir costos.
- **Ambiente de producción (prod)**: Aquí es importante configurar instancias y recursos con redundancia y alta disponibilidad.
  
Ejemplo de estructura para gestionar ambientes:

```bash
├── main.tf
├── variables.tf
├── dev.tfvars
├── prod.tfvars
```

Al aplicar los cambios para un ambiente en específico, puedes ejecutar:

```bash
terraform apply --var-file="dev.tfvars"
```

### **Automatización con CI/CD**

Integrar Terraform en un flujo de CI/CD es una excelente práctica para automatizar la gestión de la infraestructura. Puedes utilizar herramientas como Jenkins, GitLab CI, o GitHub Actions para automatizar el proceso de despliegue y validación.

Ejemplo de un pipeline básico en GitLab CI:

```yaml
stages:
  - validate
  - plan
  - apply

validate:
  script:
    - terraform init
    - terraform validate

plan:
  script:
    - terraform plan

apply:
  script:
    - terraform apply --auto-approve
```

Este pipeline primero inicializa el entorno, luego valida la configuración, y finalmente aplica los cambios automáticamente.

### **Seguridad**

- **Manejo seguro de credenciales**: Nunca almacenar credenciales en el código fuente. Utilizar herramientas como **AWS Secrets Manager** o **HashiCorp Vault** para gestionar los secretos de manera segura.
- **Control de acceso basado en roles (IAM)**: Asignar roles y permisos específicos a los recursos de Terraform mediante políticas de IAM para restringir el acceso según sea necesario.
- **Cifrado de datos**: Utilizar cifrado en reposo y en tránsito para proteger los datos sensibles, como el uso de **KMS (Key Management Service)** de AWS.
- **Seguridad en el estado**: Si almacenas el archivo `terraform.tfstate` en un bucket S3, asegúrate de habilitar el cifrado y el control de versiones para evitar modificaciones no autorizadas.

---

### **Manejo de variables en Terraform**

Para hacer escalable y reutilizable el archivo de definición de infraestructura, se recomienda no usar valores "hardcoded". Terraform permite crear variables de los siguientes tipos:

- **string**
- **number**
- **boolean**
- **map**
- **list**

Si no se declara un tipo, el valor por defecto será `string`. Sin embargo, es una buena práctica especificar el tipo de la variable.

Ejemplo de definición de variables:

```terraform
variable "ami_id" {
  type        = string
  description = "ID de la AMI"
}

variable "instance_type" {
  type        = string
  description = "Tipo de instancia"
}

variable "tags" {
  type        = map
  description = "Etiquetas para la instancia"
}
```

### **Asignar valores a las variables**

Los valores de las variables se pueden asignar de tres maneras:

1. Utilizando variables de entorno.
2. Pasándolos como argumentos en la línea de comandos.
3. Mediante un archivo `.tfvars` con formato `key = value`.

Ejemplo de archivo `.tfvars`:

```terraform
ami_id        = "ami-0ca0c67309196175e"
instance_type = "t2.micro"
tags = {
  Name       = "devops-tf"
  Environment = "Dev"
}
```

Para usar este archivo con variables:

```bash
terraform apply --var-file="dev.tfvars"
```

### **Destruir la infraestructura**

Para eliminar la infraestructura creada, se puede utilizar:

```bash
terraform destroy --var-file="dev.tfvars" -auto-approve
```
---
# Procedure: Azure Function Deployment with Terraform

Documentation of the process performed to provision an **Azure Function App (Windows / Node.js ~18)** using Terraform and Azure CLI.

---

## Resources created

| Resource | Terraform Type | Name |
|---------|----------------|--------|
| Resource Group | `azurerm_resource_group` | `otroxd` |
| Storage Account | `azurerm_storage_account` | `otroxd` |
| Service Plan (Consumption Y1) | `azurerm_service_plan` | `otroxd` |
| Windows Function App | `azurerm_windows_function_app` | `otroxd` |
| Function (HTTP trigger) | `azurerm_function_app_function` | `otroxd` |

- **Region:** `eastus2`
- **Invocation URL:** `https://otroxd.azurewebsites.net/api/otroxd`
- **Provider:** `hashicorp/azurerm` v5.4.0

---


---

## Project initialization (`terraform init`)

From the project directory (`azfunction-tf`):

```bash
terraform init
```

This command downloads the `azurerm` provider, creates `.terraform/`, and generates the lock file `.terraform.lock.hcl`.


---

## 3. Code formatting (`terraform fmt`)

```bash
terraform fmt
```


---


## 6. Execution plan (`terraform plan`)

```bash
terraform plan
```

Terraform prompts for the variables defined in `variables.tf`:

- `name_function` → value used: `otroxd`
- `location` → defaults to `West Europe` (later changed in the code to `eastus2` due to subscription policies)

The plan shows the resources that will be created (Resource Group, Storage Account, Service Plan, Function App, and Function).

> **`terraform plan` output**
>
> ![](https://cdn.phototourl.com/free/2026-09-06-e8af721f-83e6-49d1-9353-44a13d2a8f01.png)

---


## Successful apply (`terraform apply`)

```bash
terraform apply
```

Confirm with `yes` when Terraform asks for approval.

Resources provisioned successfully:

1. Resource Group `otroxd`
2. Storage Account `otroxd`
3. Service Plan `otroxd` (SKU `Y1` — consumption)
4. Windows Function App `otroxd` (Node.js `~18`)
5. HTTP Function `otroxd` (code from `example/index.js`)

> **`terraform apply` completed (Apply complete!)**
>
> 
>
> ![](https://cdn.phototourl.com/free/2026-09-06-d2567450-cab6-4e93-a296-86cfd1522f78.png)
>
> ![](https://cdn.phototourl.com/free/2026-09-06-522966d9-111d-4bc8-9815-308aed947a2e.png)
---

Output obtained:

```text
url = "https://otroxd.azurewebsites.net/api/otroxd"
```
![](https://cdn.phototourl.com/free/2026-09-06-637bb21b-9a09-49eb-b6c2-5b74c9adf599.png)
---

## 11. Testing the Function

### Via browser or curl

```bash
curl "https://otroxd.azurewebsites.net/api/otroxd?name=Azure"
```

---

## Command summary

```bash
# 1. Install Terraform
sudo apt update && sudo apt install terraform

# 2. Initialize
terraform init

# 3. Format
terraform fmt

# 4. Install Azure CLI (for Ubuntu)
curl -fsSL 'https://azurecliprod.blob.core.windows.net/$root/deb_install.sh' | sudo bash 

# 5. Authenticate
az login

# 6. Plan
terraform plan

# 7. (If it fails due to location) check policies and set location to eastus2 in main.tf
az policy assignment list --query "[?contains(displayName, 'Allowed locations') || contains(displayName, 'regions')].parameters"

# 8. (If it fails due to provider) register Microsoft.Storage
az provider register --namespace Microsoft.Storage
az provider show -n Microsoft.Storage --query "registrationState"

# 9. Apply
terraform apply

# 10. View URL
terraform output

# 11. Test
curl "https://otroxd.azurewebsites.net/api/otroxd?name=Azure"
```

---

## Destroy (optional)

To remove all the infrastructure created:

```bash
terraform destroy
```


## Project structure

```text
azfunction-tf/
├── main.tf                 # Azure resources (RG, Storage, Plan, Function App, Function)
├── variables.tf            # Variables: name_function, location
├── outputs.tf              # Output: invocation URL
├── example/
│   └── index.js            # Function code (HTTP trigger)
├── .terraform/             # Downloaded provider (generated by init)
├── .terraform.lock.hcl     # Provider version lock
├── terraform.tfstate       # Current state of the infrastructure
└── PROCEDIMIENTO.md        # This document
```

---

## Notes

- The `name_function` name must be globally unique in Azure (Storage Account and Function App).
- `sku_name = "Y1"` corresponds to the **consumption** plan.
- The HTTP trigger is configured with `authLevel: anonymous` (GET and POST).
- If the subscription restricts regions, use an allowed location (in this case `eastus2`).
