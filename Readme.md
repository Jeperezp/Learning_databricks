# Data Lake Setup and Utilities

Este repositorio contiene funciones y configuraciones para facilitar la gestión de un Data Lake utilizando contenedores organizados en niveles **bronze**, **silver** y **gold**. Está diseñado para entornos como **Azure Databricks**, aunque puede adaptarse a otros entornos Sp

## 📁 Estructura del repositorio

```text
Includes/
├── common_functions/
│   └── Funciones comunes como la creación de columnas environment e ingestion_date.
├── config/
│   └── Variables de configuración para rutas de contenedores: bronze, silver, gold.

set_up/
└── Mount_containers/
    └── Función para el montaje de contenedores en el entorno de ejecución.
```

---

## ⚙️ Contenido del repositorio

### Includes/common_functions/
Contiene funciones reutilizables aplicables a los DataFrames, como:

- `add_environment_column(df)`
- `add_ingestion_date_column(df)`

Estas funciones son útiles para mantener trazabilidad y control de datos.

### Includes/config/
Archivo de configuración con las rutas de los contenedores de almacenamiento:

```
python
BRONZE_PATH = "/mnt/bronze"
SILVER_PATH = "/mnt/silver"
GOLD_PATH   = "/mnt/gold"
```

Esto permite centralizar y estandarizar las ubicaciones de almacenamiento en distintos scripts.

### set_up/

Contiene la lógica necesaria para preparar el entorno de ejecución en Azure Databricks:

- **Mount_containers/**  
  Incluye funciones para montar los contenedores del Data Lake (`bronze`, `silver`, `gold`) en Databricks.  
  Este montaje se realiza utilizando los secretos almacenados en **Azure Key Vau(accedidos de forma segura desde Databricks).  
  Los secretos necesarios para autenticarse contra Azure AD son:

  - `tenant-id`
  - `client-id`
  - `client-secret`

  Estos mismos secretos también son utilizados para configurar las credenciales necesarias para montar los contenedores con `dbutils.fs.mount(...)`.

## 🚀 Ejemplo de uso

```python
from Includes.common_functions.create_columns import add_environment_column, add_ingestion_date_column
from Includes.config.containers import BRONZE_PATH
from set_up.Mount_containers.mount import mount_containers

# Montaje de contenedores
mount_containers()

# Aplicación de funciones a un DataFrame
df = spark.read.parquet(f"{BRONZE_PATH}/example_data")
df = add_environment_column(df)
df = add_ingestion_date_column(df)
```

- **secrets_utils**  
  Notebook auxiliar que accede a los mismos secretos utilizando un **Databricks Secret Scope** ya creado.  
  Este notebook es útil para cargar los valores de autenticación (`tenant-id`, `client-id`, `client-secret`) directamente desde el scope y preparar variables globales que puedan ser reutilizadas.

  ```python
  tenant_id = dbutils.secrets.get(scope="project-scope", key="tenant-id")
  client_id = dbutils.secrets.get(scope="project-scope", key="client-id")
  client_secret = dbutils.secrets.get(scope="project-scope", key="client-secret")
  ```

  Aunque se accede desde diferentes mecanismos (Key Vault vs. Scope), los secretos utilizados son los mismos para garantizar consistencia.

> ⚠️ **Importante:**  
> Las rutas de los contenedores (`/mnt/bronze`, `/mnt/silver`, `/mnt/gold`) **no están codificadas directamente en los notebooks**, sino que se encuentran centralizadas en el módulo `Includes/config/containers.py`. Esto permite una mayor reutilización y mantenibilidad del código.



---


## ✅ Requisitos

Antes de ejecutar este proyecto, asegúrate de tener los siguientes servicios creados y configurados en tu suscripción de Azure:

### Servicios de Azure necesarios:

- **Azure Data Lake Storage Gen2 (ADLS)**: para el almacenamiento de los datos en contenedores `bronze`, `silver` y `gold`.
- **Azure Databricks**: para ejecutar notebooks o scripts PySpark y montar los contenedores del Data Lake.
- **Azure Key Vault**: para almacenar y acceder de forma segura a credenciales y secretos.

### Secretos que deben estar configurados en Key Vault (o en el scope de secretos de Databricks):

- `tenant-id`: ID del inquilino (Azure Active Directory).
- `client-id`: ID de la aplicación registrada en Azure AD.
- `client-secret`: Secreto de autenticación de la apl

