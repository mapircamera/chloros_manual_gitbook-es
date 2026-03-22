# CLI: Línea de comandos

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** proporciona un potente acceso mediante línea de comandos al motor de procesamiento de imágenes Chloros, lo que permite la automatización, la creación de scripts y el funcionamiento sin interfaz gráfica para sus flujos de trabajo de imágenes.

### Características principales

* 🚀 **Automatización**: procesamiento por lotes mediante scripts de múltiples conjuntos de datos
* 🔗 **Integración**: se integra en flujos de trabajo y procesos existentes
* 💻 **Funcionamiento sin interfaz gráfica**: se ejecuta sin interfaz gráfica de usuario
* 🌍 **Multilingüe**: compatible con 38 idiomas
* ⚡ **Procesamiento paralelo**: la [adaptación dinámica de la computación](processing-architecture/dynamic-compute-adaptation.md) se optimiza automáticamente para su hardware

### Requisitos

| Requisito          | Detalles                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Sistema operativo** | Windows 10/11 (64 bits), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licencia**          | Chloros+ ([se requiere un plan de pago](https://cloud.mapir.camera/pricing)) |
| **Memoria**           | 8 GB de RAM como mínimo (se recomiendan 16 GB)                                  |
| **Internet**         | Necesario para la activación de la licencia                                     |
| **Espacio en disco**       | Varía según el tamaño del proyecto                                              |

{% hint style="warning" %}
**Requisitos de licencia**: CLI requiere una suscripción de pago a Chloros+. Los planes estándar (gratuitos) no tienen acceso a CLI. Visite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) para actualizar.
{% endhint %}

## Inicio rápido

### Instalación

#### Windows

El CLI se incluye automáticamente con el instalador de Chloros:

1. Descargue y ejecute **Chloros Installer.exe**

2. Complete el asistente de instalación
3. CLI instalado en: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
El instalador añade automáticamente `chloros-cli` a la ruta PATH de su sistema. Reinicie su terminal tras la instalación.
{% endhint %}

#### Linux

Instale el paquete `.deb` para su arquitectura:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Para obtener información detallada sobre la configuración de Linux, consulte [Instalación de Linux](linux/linux-installation.md).

### Configuración inicial

Antes de utilizar CLI, active su licencia Chloros+:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Uso básico

Procesar una carpeta con la configuración predeterminada:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Referencia de comandos

### Sintaxis general

```
chloros-cli [global-options] <command> [command-options]
```

***

## Comandos

### `process` - Procesar imágenes

Procesa las imágenes de una carpeta con calibración.

**Sintaxis:**

```bash
chloros-cli process <input-folder> [options]
```

**Ejemplos:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Opciones del comando de procesamiento

| Opción                | Tipo    | Predeterminado        | Descripción                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Ruta    | _Obligatorio_     | Carpeta que contiene imágenes multiespectrales RAW/JPG                                         |
| `-o, --output`        | Ruta    | Igual que la de entrada  | Carpeta de salida para las imágenes procesadas                                                     |
| `-n, --project-name`  | Cadena  | Generado automáticamente | Nombre de proyecto personalizado                                                                    |
| `--vignette`          | Indicador    | Activado        | Activar corrección de viñeteado                                                             |
| `--no-vignette`       | Indicador    | -              | Desactivar corrección de viñeteado                                                            |
| `--reflectance`       | Indicador    | Activado        | Activar calibración de reflectancia                                                         |
| `--no-reflectance`    | Indicador    | -              | Desactivar calibración de reflectancia                                                        |
| `--ppk`               | Indicador    | Desactivado       | Aplicar correcciones PPK a partir de los datos del sensor de luz .daq                                      |
| `--format`            | Opción  | TIFF (16 bits)  | Formato de salida: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Entero | Automático           | Tamaño mínimo del objetivo en píxeles para la detección del panel de calibración                          |
| `--target-clustering` | Entero | Automático           | Umbral de agrupamiento de objetivos (0-100)                                                    |
| `--debayer`           | Opción  | `standard`     | Método de despixelado: `standard` o `texture-aware` (solo Chloros+)                          |
| `--target`, `--targets` | Indicador  | Desactivado       | Buscar objetivos de calibración únicamente en una subcarpeta «target» o «targets» (acelera el procesamiento) |
| `--indices`           | Lista    | Ninguna           | Índices de vegetación a calcular (p. ej., `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Cadena  | Ninguna           | Bloquear la exposición para el modelo de cámara (Pin 1)                                                 |
| `--exposure-pin-2`    | Cadena  | Ninguno           | Bloqueo de exposición para el modelo de cámara (Pin 2)                                                 |
| `--recal-interval`    | Entero | Auto           | Intervalo de recalibración en segundos                                                      |
| `--timezone-offset`   | Entero | 0              | Desfase horario en horas                                                               |

***

### `login` - Autenticar cuenta

Inicie sesión con sus credenciales Chloros+ para habilitar el procesamiento CLI.

**Sintaxis:**

```bash
chloros-cli login <email> <password>
```

**Ejemplo:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caracteres especiales**: Utilice comillas simples alrededor de las contraseñas que contengan caracteres como `$`, `!` o espacios.
{% endhint %}

**Resultado:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Borrar credenciales

Borra las credenciales almacenadas y cierra sesión en tu cuenta.

**Sintaxis:**

```bash
chloros-cli logout
```

**Ejemplo:**

```bash
chloros-cli logout
```

**Salida:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Usuarios de SDK**: Python SDK también proporciona un método programático `logout()` para borrar credenciales dentro de scripts Python. Consulte la [documentación de Python SDK](api-python-sdk.md#logout) para obtener más detalles.
{% endhint %}

***

### `status` - Comprobar el estado de la licencia

Muestra el estado actual de la licencia y la autenticación.

**Sintaxis:**

```bash
chloros-cli status
```

**Ejemplo:**

```bash
chloros-cli status
```

**Salida:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Comprobar el progreso de la exportación

Supervisa el progreso de la exportación del hilo 4 durante o después del procesamiento.

**Sintaxis:**

```bash
chloros-cli export-status
```

**Ejemplo:**

```bash
chloros-cli export-status
```

**Caso de uso:** Ejecute este comando mientras se está ejecutando el procesamiento para comprobar el progreso de la exportación.***

### `language` - Gestionar el idioma de la interfaz

Ver o cambiar el idioma de la interfaz CLI.

**Sintaxis:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Ejemplos:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Idiomas admitidos (38 en total)

| Código    | Idioma              | Nombre nativo      |
| ------- | --------------------- | ---------------- |
| `en`    | Inglés               | English          |
| `es`    | Español               | Español          |
| `pt`    | Portugués            | Português        |
| `fr`    | Francés                | Français         |
| `de`    | Alemán                | Deutsch          |
| `it`    | Italiano               | Italiano         |
| `ja`    | Japonés              | 日本語              |
| `ko`    | Coreano                | 한국어              |
| `zh`    | Chino (simplificado)  | 简体中文             |
| `zh-TW` | Chino (tradicional) | 繁體中文             |
| `ru`    | Ruso               | Русский          |
| `nl`    | Neerlandés                | Nederlands       |
| `ar`    | Árabe                | العربية          |
| `pl`    | Polaco                | Polski           |
| `tr`    | Turco               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonesio            | Bahasa Indonesia |
| `vi`    | Vietnamita            | Tiếng Việt       |
| `th`    | Tailandés                  | ไทย              |
| `sv`    | Sueco               | Svenska          |
| `da`    | Danés                | Dansk            |
| `no`    | Noruego             | Norsk            |
| `fi`    | Finlandés               | Suomi            |
| `el`    | Griego                 | Ελληνικά         |
| `cs`    | Checo                | Čeština          |
| `hu`    | Húngaro             | Magyar           |
| `ro`    | Rumano              | Română           |
| `uk`    | Ucraniano            | Українська       |
| `pt-BR` | Portugués brasileño  | Português Brasileiro |
| `zh-HK` | Cantonés             | 粵語             |
| `ms`    | Malayo                 | Bahasa Melayu    |
| `sk`    | Eslovaco                | Slovenčina       |
| `bg`    | Búlgaro             | Български        |
| `hr`    | Croata              | Hrvatski         |
| `lt`    | Lituano            | Lietuvių         |
| `lv`    | Letón               | Latviešu         |
| `et`    | Estonio              | Eesti            |
| `sl`    | Esloveno             | Slovenščina      |

{% hint style="success" %}
**Persistencia automática**: Su preferencia de idioma se guarda en `~/.chloros/cli_language.json` y se mantiene en todas las sesiones.
{% endhint %}

***

### `set-project-folder` - Establecer carpeta de proyecto predeterminada

Cambia la ubicación de la carpeta de proyecto predeterminada (compartida con la interfaz gráfica de usuario en Windows).

**Sintaxis:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Ejemplos:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Mostrar carpeta de proyectos

Muestra la ubicación actual de la carpeta de proyectos predeterminada.

**Sintaxis:**

```bash
chloros-cli get-project-folder
```

**Ejemplo:**

```bash
chloros-cli get-project-folder
```

**Salida:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Restablecer valores predeterminados

Restablece la carpeta de proyectos a la ubicación predeterminada.

**Sintaxis:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Ejecutar diagnósticos del sistema

Ejecuta 7 comprobaciones de diagnóstico para verificar la configuración del sistema.

**Sintaxis:**

```bash
chloros-cli selftest
```

**Diagnósticos realizados:**

1. Comprobación de la versión
2. Disponibilidad del puerto (5000)
3. Inicio del backend
4. Prueba de conectividad API
5. Información del sistema y detección de la GPU
6. Verificación de los modelos del denoiser
7. Comprobación de la disponibilidad de CUDA

{% hint style="info" %}
**Útil para la resolución de problemas**: Ejecute `selftest` después de la instalación para verificar que su sistema está configurado correctamente, especialmente en Linux/Jetson, donde puede ser necesario verificar la configuración de la GPU y CUDA.
{% endhint %}

***

### `update` - Comprobar si hay actualizaciones (solo Linux)

Comprueba e instala las actualizaciones de CLI en los sistemas Linux.

**Sintaxis:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Opción    | Descripción                        |
| --------- | ---------------------------------- |
| `--check` | Solo buscar actualizaciones, no instalar |

{% hint style="info" %}
Este comando solo está disponible en Linux. En Windows, las actualizaciones se distribuyen a través del instalador.
{% endhint %}

***

## Opciones globales

Estas opciones se aplican a todos los comandos:

| Opción            | Tipo    | Predeterminado       | Descripción                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Ruta    | Detectada automáticamente | Ruta al ejecutable del backend                       |
| `--port`          | Entero | 5000          | Número de puerto del backend API                          |
| `--restart`       | Indicador    | -             | Forzar reinicio del backend (termina los procesos existentes) |
| `--version`       | Indicador    | -             | Mostrar información de la versión y salir                |
| `--help`          | Indicador    | -             | Mostrar información de ayuda y salir                   |

{% hint style="info" %}
**Detección automática del backend**: La ruta `--backend-exe` se detecta automáticamente según la plataforma:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manual)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Ejemplo con opciones globales:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Guía de configuración de procesamiento

### Procesamiento paralelo y adaptación dinámica de computación

Chloros 1.1.0 incluye [Adaptación dinámica de computación](processing-architecture/dynamic-compute-adaptation.md): el motor de procesamiento **detecta automáticamente su hardware** y selecciona la estrategia óptima:

| Plataforma | Estrategia | Trabajadores | Canalización | Notas |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Eficiente en memoria, serializado |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Procesamiento simultáneo con GPU |
| **Ordenador de sobremesa con GPU de 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Buen rendimiento de sobremesa |
| **Ordenador de sobremesa con GPU de 12 GB o más** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Rendimiento óptimo del ordenador de sobremesa |
| **Sistema solo con CPU** | `CPU_PARALLEL` | núcleos - 1 | `cpu_fallback` | No se requiere GPU |

{% hint style="success" %}
**¡No se necesita configuración manual!** Chloros detecta automáticamente la CPU, la GPU, la RAM y (en Jetson) los sensores térmicos, y luego configura automáticamente el flujo de procesamiento óptimo.
{% endhint %}

### Métodos de debayer

| Método | Indicador CLI | Calidad | Velocidad | Licencia |
| --- | --- | --- | --- | --- |
| **Estándar (rápido, calidad media)** | `--debayer standard` | Buena | Rápido | Gratis / Chloros+ |
| **Sensible a la textura (lento, máxima calidad)** | `--debayer texture-aware` | Máxima | Lento | Solo Chloros+ |

El método de debayer predeterminado es **Estándar**. El método**Sensible a la textura** utiliza un modelo de eliminación de ruido basado en IA/ML para obtener resultados de la máxima calidad, pero requiere una licencia Chloros+ y una GPU NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Corrección de viñeteado

**Qué hace:** Corrige la pérdida de luz en los bordes de la imagen (las esquinas más oscuras habituales en las imágenes de cámara).

* **Habilitado por defecto**: la mayoría de los usuarios deberían mantenerlo habilitado
* Utilice `--no-vignette` para deshabilitarlo

{% hint style="success" %}
**Recomendación**: Habilite siempre la corrección de viñeteado para garantizar un brillo uniforme en todo el fotograma.
{% endhint %}

### Calibración de reflectancia

Convierte los valores brutos del sensor en porcentajes de reflectancia estandarizados utilizando paneles de calibración.

* **Habilitado por defecto**: esencial para el análisis de la vegetación
* Requiere paneles de calibración en las imágenes
* Utilice `--no-reflectance` para deshabilitarlo

{% hint style="info" %}
**Requisitos**: Asegúrese de que los paneles de calibración estén correctamente expuestos y sean visibles en sus imágenes para una conversión precisa de la reflectancia.
{% endhint %}

### Correcciones PPK

**Qué hace:** Aplica correcciones cinemáticas posprocesadas utilizando datos de registro DAQ-A-SD para mejorar la precisión del GPS.

* **Desactivado por defecto**
* Utilice `--ppk` para activarlo
* Requiere archivos .daq en la carpeta del proyecto procedentes del sensor de luz DAQ-A-SD MAPIR.

### Formatos de salida

<table><thead><tr><th width="197">Formato</th><th width="130.20001220703125">Profundidad de bits</th><th width="116.5999755859375">Tamaño del archivo</th><th>Ideal para</th></tr></thead><tbody><tr><td><strong>TIFF (16 bits)</strong> ⭐</td><td>Entero de 16 bits</td><td>Grande</td><td>Análisis SIG, fotogrametría (recomendado)</td></tr><tr><td><strong>TIFF (32 bits, porcentaje)</strong></td><td>Número flotante de 32 bits</td><td>Muy grande</td><td>Análisis científico, investigación</td></tr><tr><td><strong>PNG (8 bits)</strong></td><td>Entero de 8 bits</td><td>Medio</td><td>Inspección visual, uso compartido en la web</td></tr><tr><td><strong>JPG (8 bits)</strong></td><td>Entero de 8 bits</td><td>Pequeño</td><td>Vista previa rápida, salida comprimida</td></tr></tbody></table>***

## Automatización y scripts

### Procesamiento por lotes con PowerShell (Windows)

Procesa automáticamente varias carpetas de conjuntos de datos en Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Script por lotes Windows (Windows)

Bucle simple para el procesamiento por lotes en Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Procesamiento por lotes en Bash (Linux)

Procesar varias carpetas de conjuntos de datos en Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Script de automatización Python (multipropósito)

Automatización avanzada con gestión de errores (funciona en Windows y Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Flujo de trabajo de procesamiento

### Flujo de trabajo estándar

1. **Entrada**: Carpeta que contiene pares de imágenes RAW/JPG
2. **Detección**: CLI busca automáticamente los archivos de imagen compatibles
3. **Procesamiento**: El modo paralelo se adapta a los núcleos de su CPU (Chloros+)
4. **Salida**: Crea subcarpetas por modelo de cámara con las imágenes procesadas

### Ejemplo de estructura de salida

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Estimaciones del tiempo de procesamiento

Tiempos de procesamiento típicos para 100 imágenes (12 MP cada una):

| Plataforma | Modo | Tiempo estimado | Notas |
| --- | --- | --- | --- |
| **Ordenador de sobremesa con GPU de 12 GB o más** | `GPU_PARALLEL` | 5-10 min | Opción más rápida |
| **Ordenador de sobremesa con GPU de 8 GB** | `GPU_SINGLE` | 10-15 min | Buen rendimiento |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15-25 min | Computación periférica |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30-60 min | Memoria limitada |
| **Solo CPU** | `CPU_PARALLEL` | 20-40 min | No requiere GPU |

{% hint style="info" %}
**Consejo de rendimiento**: El tiempo de procesamiento varía en función del número de imágenes, la resolución, el método de debayer y el hardware. El debayer con reconocimiento de texturas tarda considerablemente más que el estándar. Consulte [Adaptación dinámica de computación](processing-architecture/dynamic-compute-adaptation.md) para obtener más detalles.
{% endhint %}

***

## Solución de problemas

### CLI no encontrado

**Error Windows:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Soluciones para Windows:**

1. Compruebe la ubicación de la instalación:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Utilice la ruta completa si no está en PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Añádalo a PATH manualmente:
   * Abra Propiedades del sistema → Variables de entorno
   * Edite la variable PATH
   * Añada: `C:\Program Files\Chloros\resources\cli`
   * Reinicie el terminal

**Linux Error:**

```
chloros-cli: command not found
```

**Linux Soluciones:**

1. Verifique la instalación:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Vuelva a cargar su shell:

```bash
source ~/.bashrc
```

3. Compruebe los permisos:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Fallo al iniciar el backend**Error:**

```

Backend failed to start within 30 seconds
```

**Soluciones:**

1. Compruebe si el backend ya se está ejecutando (ciérrelo primero)
2. Compruebe que el cortafuegos no lo esté bloqueando (Windows) o compruebe la disponibilidad del puerto (Linux: `lsof -i :5000`)
3. Pruebe con un puerto diferente:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Force el reinicio del backend:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. En el caso de Linux, compruebe que el ejecutable del backend existe:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Problemas de licencia/autenticación**Error:**

```

Chloros+ license required for CLI access
```

**Soluciones:**

1. Comprueba que tienes una suscripción activa a Chloros+
2. Inicia sesión con tus credenciales:

```bash
chloros-cli login user@example.com 'password'
```

3. Compruebe el estado de la licencia:

```bash
chloros-cli status
```

4. Póngase en contacto con el servicio de asistencia: info@mapir.camera

***

### No se han encontrado imágenes**Error:**

```

No images found in the specified folder
```

**Soluciones:**

1. Comprueba que la carpeta contenga formatos compatibles (.RAW, .TIF, .JPG)
2. Comprueba que la ruta de la carpeta sea correcta (utiliza comillas para las rutas con espacios)
3. Asegúrate de tener permisos de lectura para la carpeta
4. Comprueba que las extensiones de los archivos sean correctas

***

### El procesamiento se detiene o se bloquea**Soluciones:**

1. Comprueba el espacio disponible en disco (asegúrese de que haya suficiente para la salida)
2. Cierre otras aplicaciones para liberar memoria
3. Reduzca el número de imágenes (procéselas por lotes)

***

### Puerto ya en uso**Error:**

```

Port 5000 is already in use
```

**Soluciones:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Preguntas frecuentes

### P: ¿Necesito una licencia para el CLI?

**R:**¡Sí! El CLI requiere una**licencia Chloros+** de pago.

* ❌ Plan Estándar (gratuito): CLI desactivado
* ✅ Planes Chloros+ (de pago): CLI totalmente habilitado

Suscríbete en: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### P: ¿Puedo utilizar el CLI en un servidor sin interfaz gráfica de usuario?**R:** ¡Sí! El CLI funciona completamente sin interfaz gráfica. Este es el caso de uso principal en Linux.**Servidor Windows:**
* Servidor Windows 2016 o posterior
* Visual C++ Redistributable instalado

**Servidor Linux:**
* Ubuntu 20.04+ / Debian 11+ (amd64) o JetPack 6 (arm64)
* Instalar mediante el paquete `.deb`

**Ambas plataformas:**
* 8 GB de RAM como mínimo (se recomiendan 16 GB)
* Activación de licencia única: `chloros-cli login user@example.com 'password'`

***

### P: ¿Dónde se guardan las imágenes procesadas?**R:**De forma predeterminada, las imágenes procesadas se guardan en la**misma carpeta que las de entrada**, en subcarpetas de modelos de cámara (por ejemplo, `Survey3N_RGN/`).

Utilice la opción `-o` para especificar una carpeta de salida diferente:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### P: ¿Puedo procesar varias carpetas a la vez?**R:** No directamente con un solo comando, pero puede utilizar scripts para procesar las carpetas de forma secuencial. Consulte la sección [Automatización y scripts](CLI.md#automation--scripting).***

### P: ¿Cómo guardo la salida de CLI en un archivo de registro?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### P: ¿Qué ocurre si pulso Ctrl+C durante el procesamiento?**R:** CLI hará lo siguiente:

1. Detendrá el procesamiento de forma controlada
2. Apagará el backend
3. Saldrá con el código 130

Es posible que queden imágenes procesadas parcialmente en la carpeta de salida.

***

### P: ¿Puedo automatizar el procesamiento de CLI?**R:** ¡Por supuesto! El CLI está diseñado para la automatización. Consulte [Automatización y scripts](CLI.md#automation--scripting) para ver ejemplos de PowerShell (Windows), Batch (Windows), Bash (Linux) y Python (multipropósito).***

### P: ¿Cómo puedo comprobar la versión de CLI?**R:**

```bash
chloros-cli --version
```

**Salida:**

```

Chloros CLI 1.1.0
```

***

## Obtener ayuda

### Ayuda de la línea de comandos

Ver la información de ayuda directamente en CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Canales de asistencia

* **Correo electrónico**: info@mapir.camera
* **Sitio web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Precios**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Ejemplos completos

### Ejemplo 1: Procesamiento básico

Proceso con ajustes predeterminados (viñeta, reflectancia):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Ejemplo 2: Resultados científicos de alta calidad

32 bits en coma flotante TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Ejemplo 3: Procesamiento rápido de vista previa

PNG de 8 bits sin calibración para una revisión rápida:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Ejemplo 4: Procesamiento con corrección PPK

Aplicar correcciones PPK con reflectancia:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Ejemplo 5: Ubicación de salida personalizada

Procesar en una ubicación diferente con un formato específico:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Ejemplo 6: Flujo de trabajo de autenticación

Flujo de autenticación completo (igual en todas las plataformas):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Ejemplo 7: Uso multilingüe

Cambiar el idioma de la interfaz (igual en todas las plataformas):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
