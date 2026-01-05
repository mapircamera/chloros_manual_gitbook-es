# API : Python SDK

El **Chloros Python SDK** proporciona acceso programático al motor de procesamiento de imágenes Chloros, lo que permite la automatización, los flujos de trabajo personalizados y la integración perfecta con sus aplicaciones Python y sus procesos de investigación.

### Características principales

* 🐍 **Python nativo** - API limpio y pythónico para el procesamiento de imágenes
* 🔧 **Acceso completo a API** - Control total sobre el procesamiento Chloros
* 🚀 **Automatización** - Cree flujos de trabajo de procesamiento por lotes personalizados
* 🔗 **Integración**: incruste Chloros en aplicaciones Python existentes.
* 📊 **Listo para la investigación**: perfecto para procesos de análisis científico.
* ⚡ **Procesamiento paralelo**: se adapta a los núcleos de su CPU (Chloros+).

### Requisitos

| Requisito          | Detalles                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Debe estar instalado localmente                                           |
| **Licencia**          | Chloros+ ([se requiere plan de pago](https://cloud.mapir.camera/pricing)) |
| **Sistema operativo** | Windows 10/11 (64 bits)                                              |
| **Python**           | Python 3.7 o superior                                                |
| **Memoria**           | 8 GB de RAM como mínimo (se recomiendan 16 GB)                                  |
| **Internet**         | Necesario para la activación de la licencia                                     |

{% hint style=&quot;warning&quot; %}
**Requisitos de licencia**: Python SDK requiere una suscripción de pago a Chloros+ para acceder a API. Los planes estándar (gratuitos) no tienen acceso a API/SDK. Visite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) para actualizar.
{% endhint %}

## Inicio rápido

### Instalación

Instalar mediante pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Configuración inicial**: Antes de utilizar SDK, active su licencia Chloros+ abriendo Chloros, Chloros (navegador) o Chloros CLI e iniciando sesión con sus credenciales. Esto solo es necesario hacerlo una vez.
{% endhint %}

### Uso básico

Procese una carpeta con solo unas pocas líneas:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Control total

Para flujos de trabajo avanzados:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Guía de instalación

### Requisitos previos

Antes de instalar SDK, asegúrese de tener:

1. **Chloros Desktop** instalado ([descargar](download.md))
2. **Python 3.7+** instalado ([python.org](https://www.python.org))
3. **Licencia Chloros+ activa** ([actualizar](https://cloud.mapir.camera/pricing))

### Instalar mediante pip

**Instalación estándar:**

```bash
pip install chloros-sdk
```

**Con soporte para supervisión del progreso:**

```bash
pip install chloros-sdk[progress]
```

**Instalación de desarrollo:**

```bash
pip install chloros-sdk[dev]
```

### Verificar la instalación

Compruebe que SDK está instalado correctamente:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Configuración inicial

### Activación de la licencia

El SDK utiliza la misma licencia que el Chloros, el Chloros (navegador) y el Chloros CLI. Actívela una vez a través de la GUI o CLI:

1. Abra **Chloros o Chloros (navegador)**e inicie sesión en la pestaña Usuario <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . O bien, abra**CLI**.
2. Introduzca sus credenciales de Chloros+ e inicie sesión
3. La licencia se almacena en caché localmente (persiste tras los reinicios)

{% hint style=&quot;success&quot; %}
**Configuración única**: después de iniciar sesión a través de la GUI o CLI, SDK utiliza automáticamente la licencia almacenada en caché. ¡No se necesita autenticación adicional!
{% endhint %}

{% hint style=&quot;info&quot; %}
**Cerrar sesión**: Los usuarios de SDK pueden borrar las credenciales almacenadas en caché mediante programación utilizando el método `logout()`. Consulte el método [logout()](#logout) en la referencia API.
{% endhint %}

### Probar conexión

Verifique que SDK puede conectarse a Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## Referencia API

### Clase ChlorosLocal

Clase principal para el procesamiento de imágenes locales Chloros.

#### Constructor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parámetros:**

| Parámetro                 | Tipo | Predeterminado                   | Descripción                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL del backend local Chloros          |
| `auto_start_backend`      | bool | `True`                    | Iniciar automáticamente el backend si es necesario |
| `backend_exe`             | str  | `None` (detección automática)      | Ruta al ejecutable del backend            |
| `timeout`                 | int  | `30`                      | Tiempo de espera de la solicitud en segundos            |
| `backend_startup_timeout` | int  | `60`                      | Tiempo de espera para el inicio del backend (segundos) |

**Ejemplos:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Métodos

#### `create_project(project_name, camera=None)`

Crear un nuevo proyecto Chloros.

**Parámetros:**

| Parámetro      | Tipo | Obligatorio | Descripción                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Sí      | Nombre del proyecto                                     |
| `camera`       | str  | No       | Plantilla de cámara (por ejemplo, «Survey3N\_RGN», «Survey3W\_OCN») |

**Devuelve:** `dict` - Respuesta de creación del proyecto.**Ejemplo:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importar imágenes desde una carpeta.

**Parámetros:**

| Parámetro     | Tipo     | Obligatorio | Descripción                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Sí      | Ruta a la carpeta con imágenes         |
| `recursive`   | bool     | No       | Buscar subcarpetas (predeterminado: Falso) |

**Devuelve:** `dict`: resultados de la importación con recuento de archivos.**Ejemplo:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Configurar los ajustes de procesamiento.

**Parámetros:**

| Parámetro                 | Tipo | Predeterminado                 | Descripción                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | «Alta calidad (más rápido)» | Método Debayer                  |
| `vignette_correction`     | bool | `True`                  | Habilitar corrección de viñeta      |
| `reflectance_calibration` | bool | `True`                  | Habilitar calibración de reflectancia  |
| `indices`                 | lista | `None`                  | Índices de vegetación para calcular |
| `export_format`           | str  | «TIFF (16 bits)»         | Formato de salida                   |
| `ppk`                     | bool | `False`                 | Habilitar correcciones PPK          |
| `custom_settings`         | dict | `None`                  | Ajustes personalizados avanzados        |

**Formatos de exportación:**

* `"TIFF (16-bit)"`: recomendado para SIG/fotogrametría
* `"TIFF (32-bit, Percent)"`: análisis científico
* `"PNG (8-bit)"`: inspección visual
* `"JPG (8-bit)"`: salida comprimida

**Índices disponibles:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 y más.**Ejemplo:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Procesar las imágenes del proyecto.

**Parámetros:**

| Parámetro           | Tipo     | Predeterminado      | Descripción                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Modo de procesamiento: «paralelo» o «serie»   |
| `wait`              | bool     | `True`       | Esperar a que finalice                       |
| `progress_callback` | callable | `None`       | Función de devolución de llamada de progreso (progreso, mensaje) |
| `poll_interval`     | float    | `2.0`        | Intervalo de sondeo para el progreso (segundos)   |

**Devuelve:** `dict` - Resultados del procesamiento

{% hint style=&quot;warning&quot; %}
**Modo paralelo**: Requiere la licencia Chloros+. Se adapta automáticamente a los núcleos de la CPU (hasta 16 trabajadores).
{% endhint %}

**Ejemplo:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Obtiene la configuración actual del proyecto.

**Devuelve:** `dict`: configuración actual del proyecto.**Ejemplo:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Obtener información sobre el estado del backend.

**Devuelve:** `dict` - Estado del backend.**Ejemplo:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Apaga el backend (si se ha iniciado con SDK).

**Ejemplo:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Borra las credenciales almacenadas en la caché del sistema local.

**Descripción:**

Cierra sesión de forma programada eliminando las credenciales de autenticación almacenadas en caché. Esto es útil para:
* Cambiar entre diferentes cuentas Chloros+.
* Borrar credenciales en entornos automatizados.
* Fines de seguridad (por ejemplo, eliminar credenciales antes de desinstalar).

**Devuelve:** `dict`: resultado de la operación de cierre de sesión.**Ejemplo:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style=&quot;info&quot; %}
**Se requiere reautenticación**: Después de llamar a `logout()`, debe volver a iniciar sesión a través de Chloros, Chloros (navegador) o Chloros CLI antes de utilizar SDK.
{% endhint %}

***

### Funciones prácticas

#### `process_folder(folder_path, **options)`

Función práctica de una línea para procesar una carpeta.

**Parámetros:**

| Parámetro                 | Tipo     | Predeterminado         | Descripción                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Obligatorio        | Ruta a la carpeta con imágenes     |
| `project_name`            | str      | Generado automáticamente  | Nombre del proyecto                   |
| `camera`                  | str      | `None`          | Plantilla de cámara                |
| `indices`                 | list     | `["NDVI"]`      | Índices para calcular           |
| `vignette_correction`     | bool     | `True`          | Habilitar corrección de viñeta     |
| `reflectance_calibration` | bool     | `True`          | Habilitar calibración de reflectancia |
| `export_format`           | str      | «TIFF (16 bits)» | Formato de salida                  |
| `mode`                    | str      | `"parallel"`    | Modo de procesamiento                |
| `progress_callback`       | callable | `None`          | Retorno de llamada de progreso              |

**Devuelve:** `dict` - Resultados del procesamiento.**Ejemplo:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Compatibilidad con el administrador de contexto

SDK es compatible con los administradores de contexto para la limpieza automática:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Ejemplos completos

### Ejemplo 1: Procesamiento básico

Procesar una carpeta con la configuración predeterminada:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Ejemplo 2: flujo de trabajo personalizado

Control total sobre el proceso de procesamiento:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="High Quality (Faster)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Ejemplo 3: procesamiento por lotes de varias carpetas

Procesar varios conjuntos de datos de vuelos:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Ejemplo 4: integración del proceso de investigación

Integración de Chloros con el análisis de datos:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Ejemplo 5: Supervisión personalizada del progreso

Seguimiento avanzado del progreso con registro:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Ejemplo 6: Gestión de errores

Gestión de errores robusta para uso en producción:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Ejemplo 7: Gestión de cuentas y cierre de sesión

Gestionar credenciales mediante programación:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Ejemplo 8: Herramienta de línea de comandos

Cree una herramienta CLI personalizada con SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Uso:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Gestión de excepciones

SDK proporciona clases de excepciones específicas para diferentes tipos de errores:

### Jerarquía de excepciones

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Ejemplos de excepciones

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Temas avanzados

### Configuración personalizada del backend

Utilice una ubicación o configuración de backend personalizada:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Procesamiento sin bloqueo

Inicie el procesamiento y continúe con otras tareas:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Gestión de la memoria

Para conjuntos de datos grandes, procese por lotes:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Solución de problemas

### El backend no se inicia

**Problema:** SDK no inicia el backend.**Soluciones:**

1. Compruebe que Chloros Desktop está instalado:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Compruebe que Windows Firewall no lo está bloqueando.
3. Pruebe la ruta manual del backend:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### No se detecta la licencia**Problema:** SDK advierte de que falta la licencia.**Soluciones:**

1. Abra Chloros, Chloros (navegador) o Chloros CLI e inicie sesión.
2. Compruebe que la licencia está almacenada en la caché:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Si tiene problemas con las credenciales, borre las credenciales almacenadas en la caché y vuelva a iniciar sesión:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Póngase en contacto con el servicio de asistencia: info@mapir.camera

***

### Errores de importación**Problema:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Soluciones:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Tiempo de espera de procesamiento**Problema:** Se agota el tiempo de espera de procesamiento.**Soluciones:**

1. Aumente el tiempo de espera:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Procesar lotes más pequeños
3. Comprobar el espacio disponible en disco
4. Supervisar los recursos del sistema

***

### Puerto ya en uso**Problema:** Puerto 5000 del backend ocupado**Soluciones:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

O buscar y cerrar el proceso conflictivo:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Consejos de rendimiento

### Optimizar la velocidad de procesamiento

1. **Utilizar el modo paralelo** (requiere Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Reduzca la resolución de salida** (si es aceptable)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Desactive los índices innecesarios**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Procese en SSD** (no en HDD)***

### Optimización de la memoria

Para conjuntos de datos grandes:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Procesamiento en segundo plano

Liberar Python para otras tareas:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Ejemplos de integración

### Integración con Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Preguntas frecuentes

### P: ¿SDK requiere conexión a Internet?

**R:** Solo para la activación inicial de la licencia. Después de iniciar sesión a través de Chloros, Chloros (navegador) o Chloros CLI, la licencia se almacena en la caché local y funciona sin conexión durante 30 días.***

### P: ¿Puedo utilizar SDK en un servidor sin GUI?**R:** ¡Sí! Requisitos:

* Windows Server 2016 o posterior
* Chloros instalado (una sola vez)
* Licencia activada en cualquier máquina (licencia almacenada en caché copiada al servidor)

***

### P: ¿Cuál es la diferencia entre Desktop, CLI y SDK?

| Característica         | GUI de escritorio | CLI Línea de comandos | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Interfaz**   | Puntar y hacer clic | Comando          | Python API  |
| **Ideal para**    | Trabajo visual | Scripting        | Integración |
| **Automatización**  | Limitada     | Buena             | Excelente   |
| **Flexibilidad** | Básica       | Buena             | Máxima     |
| **Licencia**     | Chloros+    | Chloros+         | Chloros+    |***

### P: ¿Puedo distribuir aplicaciones creadas con SDK?**R:** El código SDK se puede integrar en sus aplicaciones, pero:

* Los usuarios finales necesitan tener instalado Chloros.
* Los usuarios finales necesitan tener licencias Chloros+ activas.
* La distribución comercial requiere una licencia OEM.

Póngase en contacto con info@mapir.camera para consultas sobre OEM.

***

### P: ¿Cómo actualizo SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### P: ¿Dónde se guardan las imágenes procesadas?

De forma predeterminada, en la ruta del proyecto:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### P: ¿Puedo procesar imágenes desde scripts Python que se ejecutan según lo programado?**R:** ¡Sí! Utilice el Programador de tareas Windows con scripts Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Programe a través del Programador de tareas para que se ejecute diariamente.

***

### P: ¿SDK es compatible con async/await?**R:** La versión actual es sincrónica. Para un comportamiento asíncrono, utilice `wait=False` o ejecute en un subproceso separado:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### P: ¿Cómo puedo cambiar entre diferentes cuentas Chloros+?**R:** Utilice el método `logout()` para borrar las credenciales almacenadas en la caché y, a continuación, vuelva a iniciar sesión con la nueva cuenta:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Después de cerrar la sesión, autentíquese con la nueva cuenta a través de la GUI, el navegador o CLI antes de volver a utilizar SDK.

***

## Obtener ayuda

### Documentación

* **Referencia API**: esta página

### Canales de asistencia

* **Correo electrónico**: info@mapir.camera
* **Sitio web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Precios**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Código de ejemplo

Todos los ejemplos que se incluyen aquí se han probado y están listos para su uso en producción. Cópielos y adáptelos a su caso de uso.

***

## Licencia**Software propietario** - Copyright (c) 2025 MAPIR Inc.

SDK requiere una suscripción activa a Chloros+. Se prohíbe el uso, la distribución o la modificación no autorizados.
