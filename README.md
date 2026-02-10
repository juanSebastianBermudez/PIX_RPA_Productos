[README.md](https://github.com/user-attachments/files/25217478/README.md)
# Automatización RPA - Gestión de Productos con PIX RPA

## 📋 Descripción General

Este proyecto implementa un **proceso de Automatización Robótica de Procesos (RPA)** completo utilizando la plantilla universal de **PIX RPA**. El sistema automatiza el análisis diario de productos desde una tienda online, integrando consumo de APIs, almacenamiento en base de datos, generación de reportes en Excel, automatización web y sincronización con OneDrive.



---

## 🎯 Objetivo General

Desarrollar un robot RPA que ejecute de forma automatizada las siguientes operaciones:

1. ✅ Obtener productos desde una API pública
2. ✅ Respaldar datos originales en formato JSON
3. ✅ Almacenar información estructurada en base de datos
4. ✅ Generar reportes analíticos en Excel
5. ✅ Subir reportes a OneDrive automáticamente
6. ✅ Enviar reporte a través de formulario web
7. ✅ Almaceba las evidencias del proceso

---

## 📦 Requisitos Previos

### Software Requerido
- **PIX Studio** (versión demo o superior) - [Descargar](https://es.pixrobotics.com/download/)
- **SQL Server** / **SQLite** / **PostgreSQL** 




### Credenciales Requeridas
- **Tenant ID de Microsoft Azure**
- **Client ID y Client Secret** (Azure Application)
- **Acceso a OneDrive** con permisos de API Graph




---

## 1️⃣ Consumo de API Pública

### Descripción
El primer paso del proceso es obtener los datos de productos desde **Fake Store API**, una API pública que simula una tienda online real.

### Endpoint y Configuración
- **Fuente:** Fake Store API
- **Endpoint:** `https://fakestoreapi.com/products`
- **Método:** GET
- **Documentación:** [Fake Store API Docs](https://fakestoreapi.com/docs#tag/Products)

### Campos Extraídos
```json
{
  "id": 1,
  "title": "Nombre del Producto",
  "price": 109.95,
  "category": "Categoría",
  "description": "Descripción completa del producto"
}
```

### Tareas Implementadas

#### A. Solicitud HTTP GET
Se realiza una solicitud HTTP GET al endpoint indicado con manejo robusto de errores:
- Validación de conexión
- Manejo de timeouts
- Reintentos en caso de fallo temporal
- Logging detallado de la solicitud

#### B. Almacenamiento de Respuesta JSON
La respuesta completa se guarda como respaldo en archivo JSON:
- **Ubicación OneDrive:** `/RPA/Logs/Productos_YYYY-MM-DD.json`


#### C. Extracción de Datos
Se procesan únicamente los campos requeridos:
- `id`: Identificador único del producto
- `title`: Nombre del producto
- `price`: Precio en formato decimal
- `category`: Categoría del producto
- `description`: Descripción detallada

### Manejo de Errores
- Validación de respuesta HTTP válida (código 200)
- Parsing correcto de JSON


---

## 2️⃣ Almacenamiento en Base de Datos

### Descripción
Los datos extraídos se almacenan de forma estructurada y consultable en una base de datos relacional, permitiendo consultas, análisis y reportes posteriores.

### Tecnología Utilizada
Se utiliza **SQL Server** con el script de creación provided en `PIX_RPA.sql`.

**Puede adaptarse a:**
- ✓ SQLite
- ✓ PostgreSQL
- ✓ MySQL
- ✓ Otra tecnología relacional equivalente

### Estructura de la Tabla Productos

```sql
CREATE TABLE [dbo].[Productos](
    [id] [int] NOT NULL,                          -- Identificador único
    [title] NVARCHAR(50) NULL,                    -- Nombre del producto
    [price] REAL NULL,                            -- Precio (formato decimal)
    [category] NVARCHAR(50) NULL,                 -- Categoría
    [description] NVARCHAR(250) NULL,             -- Descripción
    [fecha_insercion] [datetime2](0) NULL,        -- Timestamp de inserción
    CONSTRAINT [PK_Productos] PRIMARY KEY CLUSTERED ([id] ASC)
)
```

### Características de la Implementación

#### Clave Primaria
- **Campo:** `id` (entero)
- **Función:** Garantiza unicidad de registros
- **Beneficio:** Previene duplicados automáticamente

#### Timestamp Automático
```sql
ALTER TABLE [dbo].[Productos]
ADD CONSTRAINT [DF_Productos_FechaInsercion]
DEFAULT (sysdatetime())
FOR [fecha_insercion]
```
- Registra automáticamente la fecha/hora de inserción
- Permite auditoría temporal del proceso
- Útil para análisis de cambios en el tiempo

#### Validación de Duplicados
El proceso implementa validación antes de insertar:

```sql
-- Pseudocódigo del validación
IF NOT EXISTS (SELECT 1 FROM Productos WHERE id = @productId)
BEGIN
    INSERT INTO Productos (id, title, price, category, description)
    VALUES (@id, @title, @price, @category, @description)
END
ELSE
BEGIN
    -- Log: Producto ya existe, no se inserta duplicado
END
```

### Tipos de Datos
| Campo | Tipo | Longitud | Descripción |
|-------|------|----------|-------------|
| `id` | int | - | Clave primaria, identificador único |
| `title` | NVARCHAR | 50 | Nombre del producto |
| `price` | REAL | - | Precio con decimales |
| `category` | NVARCHAR | 50 | Categoría del producto |
| `description` | NVARCHAR | 250 | Descripción detallada |
| `fecha_insercion` | datetime2 | 0 | Timestamp de la inserción |

### Creación de la Base de Datos

Ejecutar el script `PIX_RPA.sql`:

```bash
# SQL Server
sqlcmd -S servidor -U usuario -P contraseña -i "PIX_RPA.sql"

# O directamente en SQL Server Management Studio
# File → Open → PIX_RPA.sql → Execute
```

El script automáticamente:
1. Crea la base de datos `PIX_PRA` si no existe
2. Define la estructura de tabla `Productos`
3. Establece restricciones y defaults
4. Configura índices para optimización

### Validación de la Instalación

```sql
-- Verificar creación de base de datos
SELECT * FROM sys.databases WHERE name = 'PIX_PRA'

-- Verificar estructura de tabla
SELECT * FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'Productos'

-- Verificar datos insertados
SELECT COUNT(*) as total_productos FROM [PIX_PRA].[dbo].[Productos]
```

---

## 3️⃣ Generación de Reportes en Excel

### Descripción
Una vez la información está en la base de datos, el proceso genera un reporte analítico en Excel (.xlsx) con dos hojas: una con el listado completo y otra con estadísticas resumidas.

### Especificaciones del Reporte

#### Nombre del Archivo
```
Reporte_YYYY-MM-DD.xlsx
```
- **Ejemplo:** `Reporte_2026-02-10.xlsx`
- **Formato de fechas:** Año-Mes-Día

#### Hoja 1: Productos
Contiene el **listado completo** de todos los productos registrados en la base de datos.

| Columna | Descripción | Ancho |
|---------|-------------|-------|
| ID | Identificador único del producto | 5 |
| Nombre | Título del producto | 30 |
| Precio | Precio unitario | 10 |
| Categoría | Categoría comercial | 15 |
| Descripción | Descripción detallada | 40 |
| Fecha Inserción | Fecha y hora de registro | 20 |

**Formato:**
- Encabezados en negrita con fondo azul
- Bordes en todas las celdas
- Alineación automática de ancho de columnas
- Formato moneda para precios

#### Hoja 2: Resumen
Contiene **estadísticas consolidadas** del análisis de productos.

**Estadísticas Incluidas:**

1. **Total de Productos**
   - Cantidad total de registros en la base de datos
   
2. **Precio Promedio General**
   - Promedio aritmético de todos los precios
  

3. **Precio Promedio por Categoría**
   - Desglose de promedio por cada categoría
   - Estructura tabla dinámica
   - Formato: Tabla con bordes

4. **Cantidad de Productos por Categoría**
   - Conteo de productos agrupados por categoría
   - Visualización en tabla con histograma
   - Ordenado alfabéticamente

### Ubicación y Sincronización

#### Almacenamiento Local
```
/Reportes/
├── Reporte_2026-02-10.xlsx
├── Reporte_2026-02-09.xlsx
└── Reporte_2026-02-08.xlsx
```

#### Sincronización OneDrive
```
/RPA/Reportes/Reporte_YYYY-MM-DD.xlsx
```
- Carga automática vía API Microsoft Graph
- Validación de integridad post-carga
- Logging del resultado (exitoso/fallido)

### Flujo de Generación

```
1. Conectar a base de datos
   ↓
2. Consultar tabla Productos
   ↓
3. Crear workbook en Excel
   ↓
4. Exportar datos a Hoja 1
   ↓
5. Calcular estadísticas
   ↓
6. Formatear Hoja 2 (Resumen)
   ↓
7. Guardar archivo localmente
   ↓
8. Cargar a OneDrive
   ↓
9. Registrar en log
```

### Validación de Reporte

- ✓ Dos hojas presentes y con datos
- ✓ Estadísticas calculadas correctamente
- ✓ Formato visual profesional
- ✓ Cargado en OneDrive

---

## 4️⃣ Automatización Web - Envío de Formulario

### Descripción
El paso final del proceso consiste en entregar el reporte automáticamente a través de un formulario web, simulando un flujo de negocio real.

### Plataformas Permitidas
Se puede utilizar cualquiera de las siguientes plataformas para crear el formulario:
- **Google Forms** - Fácil de usar, integración con Google Drive
- **Jotform** - Robusta, muchas opciones de integración
- **Typeform** - Diseño moderno, API disponible

### Estructura del Formulario

El formulario debe incluir los siguientes campos:

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| Nombre del Colaborador | Texto corto | Sí | Nombre de la persona que envía |
| Fecha de Generación | Fecha | Sí | Fecha del reporte |
| Comentarios | Texto largo | No | Observaciones adicionales (opcional) |
| Archivo de Reporte | Archivo | Sí | Cargar el Excel generado |

### Tareas del Robot

#### 1. Acceso al Formulario
- Navegar a la URL del formulario web
- Esperar carga completa de página
- Validar disponibilidad del formulario

#### 2. Autenticación (si aplica)
- Verificar si se requiere login
- Completar credenciales si es necesario
- Mantener sesión de usuario

#### 3. Rellenado de Campos

```
Nombre del Colaborador: [Sistema RPA - Automated]
Fecha de Generación: [YYYY-MM-DD actual]
Comentarios: [Reporte Diario - Análisis de Productos]
Archivo: [Ruta del Excel generado]
```

#### 4. Subida de Archivo
- Seleccionar archivo `Reporte_YYYY-MM-DD.xlsx`
- Validar carga del archivo
- Esperar confirmación de carga

#### 5. Envío del Formulario
- Hacer clic en botón de envío
- Esperar respuesta del servidor

#### 6. Captura de Evidencia
- Tomar screenshot de confirmación exitosa
- Guardar la captura de la pantalla


---

## 5️⃣ Integración con OneDrive - Microsoft Graph API

### Descripción
Los archivos generados (JSON con respaldo de API y Excel con reportes) se sincronizan automáticamente a OneDrive utilizando **Microsoft Graph API**, permitiendo acceso centralizado y auditoría.

### Pre-requisitos de Azure

#### A. Creación de Aplicación en Azure AD
1. Ir a [Azure Portal](https://portal.azure.com)
2. Navegar a **Azure Active Directory** → **App registrations**
3. Crear nueva aplicación con:
   - **Nombre:** PIX RPA Bot
   - **Tipo:** Public client/native
   - **URI de redirección:** (no requerida para client credentials)

#### B. Otorgación de Permisos
En la aplicación creada, ir a **API permissions**:

| Permiso | Tipo | Justificación |
|---------|------|---------------|
| `Files.ReadWrite.All` | Aplicación | Leer/escribir archivos en OneDrive |
| `Calendars.Read` | Delegado | Auditoría de cambios |

#### C. Creación de Client Secret
1. En la aplicación, ir a **Certificates & secrets**
2. Crear nuevo **Client secret**
3. Copiar el valor (se muestra una sola vez)
4. Guardar en variable de entorno segura



⚠️ **NUNCA** commitear credenciales en repositorio. Usar variables de entorno.

### Autenticación Sin Interacción Servidor

Se utiliza el flujo **Client Credentials** para obtener token de acceso:

```python
# Código en script.py
import requests

tenant = "2748ad8c-96c7-4573-aa50-********"
url = f"https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token"

headers = {
    "Content-Type": "application/x-www-form-urlencoded"
}

data = {
    "client_id": "f7fc6ed2-fea1-****************",
    "scope": "https://graph.microsoft.com/.default",
    "client_secret": "L6Y8Q~KG7fhmE56on**************",
    "grant_type": "client_credentials"
}

response = requests.post(url, headers=headers, data=data)
response.raise_for_status()
token_response = response.json()

access_token = token_response['access_token']
```

**Ventajas:**
- ✓ No requiere interacción del usuario
- ✓ Ideal para automatizaciones sin supervisor
- ✓ Token válido por 60 minutos
- ✓ Renovación automática de tokens

### Rutas en OneDrive

Estructura sugerida de carpetas:

```
/RPA/
├── Logs/
│   ├── Productos_2026-02-10.json
│   ├── Productos_2026-02-09.json
│   └── Productos_2026-02-08.json
└── Reportes/
    ├── Reportes_2026-02-10.xlsx
    ├── Reportes_2026-02-09.xlsx
    └── Reportes_2026-02-08.xlsx
```



#### Sobrescritura o Versionado
- **Sobrescribir:** Usar mismo nombre de archivo
- **Versionado:** Incluir timestamp adicional en nombre
- **Recomendación:** Sobrescribir para mantener limpieza de almacenamiento




## 📚 Recursos Adicionales

### Documentación Oficial
- **PIX RPA Academy:** [https://academy.es.pixrobotics.com/course/index.php](https://academy.es.pixrobotics.com/course/index.php)
- **PIX RPA Docs:** [https://docs.pixrobotics.com/articles/#!rpa-es/welcome](https://docs.pixrobotics.com/articles/#!rpa-es/welcome)
- **Fake Store API:** [https://fakestoreapi.com/docs](https://fakestoreapi.com/docs)
- **Microsoft Graph API:** [https://docs.microsoft.com/en-us/graph/api/](https://docs.microsoft.com/en-us/graph/api/)

### Enlaces de Referencia
- **Azure Portal:** [https://portal.azure.com](https://portal.azure.com)
- **Azure AD App Registrations:** [https://entra.microsoft.com/](https://entra.microsoft.com/)
- **OneDrive for Business:** [https://www.office.com](https://www.office.com)

### Documentación de Tecnologías
- **Python Requests:** [https://requests.readthedocs.io/](https://requests.readthedocs.io/)
- **Pandas:** [https://pandas.pydata.org/docs/](https://pandas.pydata.org/docs/)
- **OpenPyXL:** [https://openpyxl.readthedocs.io/](https://openpyxl.readthedocs.io/)
- **SQL Server T-SQL:** [https://learn.microsoft.com/en-us/sql/t-sql/](https://learn.microsoft.com/en-us/sql/t-sql/)

---

## 📝 Criterios de Evaluación

El proyecto será evaluado según los siguientes criterios:

| Criterio | Descripción | Peso |
|----------|-------------|------|
| **Estructura PIX** | Organización, nombres, claridad | 10% |
| **Consumo API** | Solicitud, parseo, respaldo JSON | 15% |
| **Base de Datos** | Inserción limpia, validación duplicados, timestamp | 15% |
| **Reporte Excel** | Datos organizados, estadísticas correctas | 15% |
| **Automatización Web** | Rellenado preciso, carga archivo, evidencia | 15% |
| **OneDrive API** | Carga exitosa, control de errores | 10% |
| **Logs y Errores** | Registro completo, logs informativos | 10% |
| **Documentación README** | Claridad, completitud, instrucciones | 10% |

---


*Última actualización: 2026-02-10*
