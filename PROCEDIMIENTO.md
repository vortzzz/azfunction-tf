# Procedimiento: Despliegue de Azure Function con Terraform

Documentación del proceso realizado para provisionar una **Azure Function App (Windows / Node.js ~18)** usando Terraform y Azure CLI.

---

## Recursos creados

| Recurso | Tipo Terraform | Nombre |
|---------|----------------|--------|
| Resource Group | `azurerm_resource_group` | `otroxd` |
| Storage Account | `azurerm_storage_account` | `otroxd` |
| Service Plan (Consumo Y1) | `azurerm_service_plan` | `otroxd` |
| Windows Function App | `azurerm_windows_function_app` | `otroxd` |
| Function (HTTP trigger) | `azurerm_function_app_function` | `otroxd` |

- **Región:** `eastus2`
- **URL de invocación:** `https://otroxd.azurewebsites.net/api/otroxd`
- **Provider:** `hashicorp/azurerm` v5.4.0

---


---

## Inicialización del proyecto (`terraform init`)

Desde el directorio del proyecto (`azfunction-tf`):

```bash
terraform init
```

Este comando descarga el provider `azurerm`, crea `.terraform/` y genera el lock file `.terraform.lock.hcl`.


---

## 3. Formateo del código (`terraform fmt`)

```bash
terraform fmt
```


---


## 6. Plan de ejecución (`terraform plan`)

```bash
terraform plan
```

Terraform solicita las variables definidas en `variables.tf`:

- `name_function` → valor usado: `otroxd`
- `location` → por defecto `West Europe` (luego se cambió en el código a `eastus2` por políticas de la suscripción)

El plan muestra los recursos que se crearán (Resource Group, Storage Account, Service Plan, Function App y Function).

> **Salida de `terraform plan`**
>
> ![](https://cdn.phototourl.com/free/2026-09-06-e8af721f-83e6-49d1-9353-44a13d2a8f01.png)

---


## Aplicación exitosa (`terraform apply`)

```bash
terraform apply
```

Confirmar con `yes` cuando Terraform pida aprobación.

Recursos provisionados correctamente:

1. Resource Group `otroxd`
2. Storage Account `otroxd`
3. Service Plan `otroxd` (SKU `Y1` — consumo)
4. Windows Function App `otroxd` (Node.js `~18`)
5. Function HTTP `otroxd` (código desde `example/index.js`)

> **`terraform apply` completado (Apply complete!)**
>
> 
>
> ![](https://cdn.phototourl.com/free/2026-09-06-d2567450-cab6-4e93-a296-86cfd1522f78.png)
>
> ![](https://cdn.phototourl.com/free/2026-09-06-522966d9-111d-4bc8-9815-308aed947a2e.png)
---

Salida obtenida:

```text
url = "https://otroxd.azurewebsites.net/api/otroxd"
```
![](https://cdn.phototourl.com/free/2026-09-06-637bb21b-9a09-49eb-b6c2-5b74c9adf599.png)
---

## 11. Prueba de la Function

### Por navegador o curl

```bash
curl "https://otroxd.azurewebsites.net/api/otroxd?name=Azure"
```

---

## Resumen de comandos 

```bash
# 1. Instalar Terraform
sudo apt update && sudo apt install terraform

# 2. Inicializar
terraform init

# 3. Formatear
terraform fmt

# 4. Instalar Azure CLI (Para Ubuntu)
curl -fsSL 'https://azurecliprod.blob.core.windows.net/$root/deb_install.sh' | sudo bash 

# 5. Autenticarse
az login

# 6. Planificar
terraform plan

# 7. (Si falla por ubicación) consultar políticas y ajustar location a eastus2 en main.tf
az policy assignment list --query "[?contains(displayName, 'Allowed locations') || contains(displayName, 'regions')].parameters"

# 8. (Si falla por provider) registrar Microsoft.Storage
az provider register --namespace Microsoft.Storage
az provider show -n Microsoft.Storage --query "registrationState"

# 9. Aplicar
terraform apply

# 10. Ver URL
terraform output

# 11. Probar
curl "https://otroxd.azurewebsites.net/api/otroxd?name=Azure"
```

---

## Destrucción (opcional)

Para eliminar toda la infraestructura creada:

```bash
terraform destroy
```


## Estructura del proyecto

```text
azfunction-tf/
├── main.tf                 # Recursos Azure (RG, Storage, Plan, Function App, Function)
├── variables.tf            # Variables: name_function, location
├── outputs.tf              # Output: URL de invocación
├── example/
│   └── index.js            # Código de la Function (HTTP trigger)
├── .terraform/             # Provider descargado (generado por init)
├── .terraform.lock.hcl     # Lock de versiones del provider
├── terraform.tfstate       # Estado actual de la infraestructura
└── PROCEDIMIENTO.md        # Este documento
```

---

## Notas

- El nombre `name_function` debe ser único a nivel global en Azure (Storage Account y Function App).
- `sku_name = "Y1"` corresponde al plan de **consumo**.
- El trigger HTTP está configurado con `authLevel: anonymous` (GET y POST).
- Si la suscripción restringe regiones, usar una ubicación permitida (en este caso `eastus2`).

