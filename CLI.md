# CLI: Línea de comandos

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** proporciona un potente acceso por línea de comandos al motor de procesamiento de imágenes Chloros, lo que permite la automatización, la creación de scripts y el funcionamiento sin interfaz gráfica para sus flujos de trabajo de imágenes.

### Características principales

* 🚀 **Automatización**: procesamiento por lotes mediante scripts de múltiples conjuntos de datos
* 🔗 **Integración**: se integra en flujos de trabajo y procesos existentes
* 💻 **Funcionamiento sin interfaz gráfica**: se ejecuta sin GUI
* 🌍 **Multilingüe**: compatible con 38 idiomas
* ⚡ **Procesamiento paralelo**: se adapta dinámicamente a su CPU (hasta 16 trabajadores paralelos).

### Requisitos

| Requisito          | Detalles                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Sistema operativo** | Windows 10/11 (64 bits)                                              |
| **Licencia**          | Chloros+ ([se requiere un plan de pago](https://cloud.mapir.camera/pricing)) |
| **Memoria**           | 8 GB de RAM como mínimo (se recomiendan 16 GB)                                  |
| **Internet**         | Necesario para la activación de la licencia                                     |
| **Espacio en disco**       | Varía según el tamaño del proyecto                                              |

{% hint style=&quot;warning&quot; %}
**Requisitos de licencia**: CLI requiere una suscripción de pago a Chloros+. Los planes estándar (gratuitos) no tienen acceso a CLI. Visite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) para actualizar.
{% endhint %}

## Inicio rápido

### Instalación

El CLI se incluye automáticamente con el instalador Chloros:

1. Descargue y ejecute **Chloros Installer.exe**
2. Complete el asistente de instalación
3. CLI instalado en: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style=&quot;success&quot; %}
El instalador añade automáticamente `chloros-cli` a la ruta PATH de su sistema. Reinicie su terminal después de la instalación.
{% endhint %}

### Configuración inicial

Antes de utilizar CLI, active su licencia Chloros+:

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

### Uso básico

Procese una carpeta con la configuración predeterminada:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Referencia de comandos

### Sintaxis general

```
chloros-cli [global-options] <command> [command-options]
```

***

## Comandos

### `process`: procesar imágenes

Procesa imágenes en una carpeta con calibración.

**Sintaxis:**

```bash
chloros-cli process <input-folder> [options]
```

**Ejemplo:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Opciones del comando de procesamiento

| Opción                | Tipo    | Predeterminado        | Descripción                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Ruta    | _Requerido_     | Carpeta que contiene imágenes multiespectrales RAW/JPG                                         |
| `-o, --output`        | Ruta    | Igual que la entrada  | Carpeta de salida para las imágenes procesadas                                                     |
| `-n, --project-name`  | Cadena  | Generado automáticamente | Nombre del proyecto personalizado                                                                    |
| `--vignette`          | Indicador    | Habilitado        | Habilitar corrección de viñeta                                                             |
| `--no-vignette`       | Indicador    | -              | Deshabilitar corrección de viñeta                                                            |
| `--reflectance`       | Indicador    | Habilitado        | Habilitar calibración de reflectancia                                                         |
| `--no-reflectance`    | Indicador    | -              | Deshabilitar calibración de reflectancia                                                        |
| `--ppk`               | Indicador    | Desactivado       | Aplicar correcciones PPK a partir de datos del sensor de luz .daq                                      |
| `--format`            | Opción  | TIFF (16 bits)  | Formato de salida: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Entero | Automático           | Tamaño mínimo del objetivo en píxeles para la detección del panel de calibración                          |
| `--target-clustering` | Entero | Automático           | Umbral de agrupación de objetivos (0-100)                                                    |
| `--exposure-pin-1`    | Cadena  | Ninguno           | Bloquear exposición para modelo de cámara (Pin 1)                                                 |
| `--exposure-pin-2`    | Cadena  | Ninguno           | Bloquear exposición para modelo de cámara (Pin 2)                                                 |
| `--recal-interval`    | Entero | Auto           | Intervalo de recalibración en segundos                                                      |
| `--timezone-offset`   | Entero | 0              | Desviación horaria en horas                                                               |

***

### `login` - Autenticar cuenta

Inicie sesión con sus credenciales Chloros+ para habilitar el procesamiento CLI.

**Sintaxis:**

```bash
chloros-cli login <email> <password>
```

**Ejemplo:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Caracteres especiales**: utilice comillas simples alrededor de las contraseñas que contengan caracteres como `$`, `!` o espacios.
{% endhint %}

**Resultado:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout`: borrar credenciales

Borra las credenciales almacenadas y cierra la sesión de tu cuenta.

**Sintaxis:**

```bash
chloros-cli logout
```

**Ejemplo:**

```powershell
chloros-cli logout
```

**Salida:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

***

### `status` - Comprobar el estado de la licencia

Muestra el estado actual de la licencia y la autenticación.

**Sintaxis:**

```bash
chloros-cli status
```

**Ejemplo:**

```powershell
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

### `export-status`: comprobar el progreso de la exportación

Supervisa el progreso de la exportación del subproceso 4 durante o después del procesamiento.

**Sintaxis:**

```bash
chloros-cli export-status
```

**Ejemplo:**

```powershell
chloros-cli export-status
```

**Caso de uso:** Llame a este comando mientras se ejecuta el procesamiento para comprobar el progreso de la exportación.

***

### `language`: gestionar el idioma de la interfaz

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

```powershell
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Idiomas compatibles (38 en total)

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
| `nl`    | Neerlandés                 | Nederlands       |
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
| `cs`    | Checo                 | Čeština          |
| `hu`    | Húngaro             | Magyar           |
| `ro`    | Rumano              | Română           |
| `uk`    | Ucraniano             | Українська       |
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

{% hint style=&quot;success&quot; %}
**Persistencia automática**: Su preferencia de idioma se guarda en `~/.chloros/cli_language.json` y se mantiene en todas las sesiones.
{% endhint %}

***

### `set-project-folder`: establecer la carpeta predeterminada del proyecto

Cambia la ubicación predeterminada de la carpeta del proyecto (compartida con la GUI).

**Sintaxis:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Ejemplo:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder`: mostrar carpeta del proyecto

Muestra la ubicación predeterminada actual de la carpeta del proyecto.

**Sintaxis:**

```bash
chloros-cli get-project-folder
```

**Ejemplo:**

```powershell
chloros-cli get-project-folder
```

**Salida:**

```
ℹ Current project folder: C:\Projects\2025
```

***

### `reset-project-folder`: restablecer a los valores predeterminados

Restablece la carpeta del proyecto a la ubicación predeterminada.

**Sintaxis:**

```bash
chloros-cli reset-project-folder
```

***

## Opciones globales

Estas opciones se aplican a todos los comandos:

| Opción          | Tipo    | Predeterminado       | Descripción                                      |
| --------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe` | Ruta    | Detectada automáticamente | Ruta al ejecutable del backend                       |
| `--port`        | Entero | 5000          | Número de puerto del backend API                          |
| `--restart`     | Indicador    | -             | Forzar reinicio del backend (elimina los procesos existentes) |
| `--version`     | Indicador    | -             | Mostrar información de la versión y salir                |
| `--help`        | Indicador    | -             | Mostrar información de ayuda y salir                   |

**Ejemplo con opciones globales:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Guía de configuración del procesamiento

### Procesamiento paralelo

Chloros+ CLI **escala automáticamente** el procesamiento paralelo para adaptarse a las capacidades de su ordenador:

**Cómo funciona:**

* Detecta los núcleos de la CPU y la RAM.
* Asigna trabajadores: **2× núcleos de CPU** (utiliza hiperprocesamiento).
* **Máximo: 16 trabajadores en paralelo** (para mayor estabilidad).

**Niveles del sistema:**

| Tipo de sistema   | CPU        | RAM      | Trabajadores  | Rendimiento     |
| ------------- | ---------- | -------- | -------- | --------------- |
| **Gama alta**  | 16+ núcleos  | 32+ GB   | Hasta 16 | Velocidad máxima   |
| **Gama media** | 8-15 núcleos | 16-31 GB | 8-16     | Excelente velocidad |
| **Gama baja**   | 4-7 núcleos  | 8-15 GB  | 4-8      | Buena velocidad      |

{% hint style=&quot;success&quot; %}
**Optimización automática**: El CLI detecta automáticamente las especificaciones de su sistema y configura el procesamiento paralelo óptimo. ¡No es necesaria ninguna configuración manual!
{% endhint %}

### Métodos Debayer

El CLI utiliza **Alta calidad (más rápido)** como algoritmo Debayer predeterminado y recomendado:

| Método                      | Calidad | Velocidad | Descripción                                 |
| --------------------------- | ------- | ----- | ------------------------------------------- |
| **Alta calidad (más rápido)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡   | Algoritmo sensible a los bordes (predeterminado, recomendado) |

### Corrección de viñeta

**Qué hace:** corrige la pérdida de luz en los bordes de la imagen (esquinas más oscuras comunes en las imágenes de cámara).

* **Habilitado de forma predeterminada**: la mayoría de los usuarios deben mantener esta opción habilitada.
* Utilice `--no-vignette` para deshabilitarla.

{% hint style=&quot;success&quot; %}
**Recomendación**: active siempre la corrección de viñeta para garantizar un brillo uniforme en todo el fotograma.
{% endhint %}

### Calibración de reflectancia

Convierte los valores brutos del sensor en porcentajes de reflectancia estandarizados utilizando paneles de calibración.

* **Habilitado de forma predeterminada**: esencial para el análisis de la vegetación.
* Requiere paneles de calibración en las imágenes.
* Utilice `--no-reflectance` para deshabilitarlo.

{% hint style=&quot;info&quot; %}
**Requisitos**: Asegúrese de que los paneles de calibración estén correctamente expuestos y visibles en sus imágenes para una conversión precisa de la reflectancia.
{% endhint %}

### Correcciones PPK

**Qué hace:** Aplica correcciones cinemáticas posprocesadas utilizando datos de registro DAQ-A-SD para mejorar la precisión del GPS.

* **Desactivado por defecto**
* Utilice `--ppk` para activarlo
* Requiere archivos .daq en la carpeta del proyecto del sensor de luz MAPIR DAQ-A-SD.

### Formatos de salida

<table><thead><tr><th width="197">Formato</th><th width="130.20001220703125">Profundidad de bits</th><th width="116.5999755859375">Tamaño del archivo</th><th>Ideal para</th></tr></thead><tbody><tr><td><strong>TIFF (16 bits)</strong> ⭐</td><td>Entero de 16 bits</td><td>Grande</td><td>Análisis GIS, fotogrametría (recomendado)</td></tr><tr><td><strong>TIFF (32 bits, porcentaje)</strong></td><td>Floating point de 32 bits</td><td>Muy grande</td><td>Análisis científico, investigación</td></tr><tr><td><strong>PNG (8 bits)</strong></td><td>Entero de 8 bits</td><td>Medio</td><td>Inspección visual, intercambio web</td></tr><tr><td><strong>JPG (8 bits)</strong></td><td>Entero de 8 bits</td><td>Pequeño</td><td>Vista previa rápida, salida comprimida</td></tr></tbody></table>***

## Automatización y secuencias de comandos

### Procesamiento por lotes de PowerShell

Procese automáticamente varias carpetas de conjuntos de datos:

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

### Windows Secuencia de comandos por lotes

Bucle simple para el procesamiento por lotes:

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

### Python Script de automatización

Automatización avanzada con gestión de errores:

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

| Modo              | Tiempo      | Hardware                                     |
| ----------------- | --------- | -------------------------------------------- |
| **Modo paralelo** | 5-10 min  | i7/Ryzen 7, 16 GB de RAM, SSD (hasta 16 trabajadores) |
| **Modo paralelo** | 10-15 min | i5/Ryzen 5, 8 GB de RAM, HDD (hasta 8 trabajadores)   |

{% hint style=&quot;info&quot; %}
**Consejo de rendimiento**: El tiempo de procesamiento varía en función del número de imágenes, la resolución y las especificaciones del ordenador.
{% endhint %}

***

## Solución de problemas

### CLI No encontrado

**Error:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Soluciones:**

1. Verifique la ubicación de la instalación:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Utilice la ruta completa si no está en PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Añádalo a PATH manualmente:
   * Abra Propiedades del sistema → Variables de entorno.
   * Edite la variable PATH.
   * Añada: `C:\Program Files\Chloros\resources\cli`
   * Reinicie el terminal.

***

### Error al iniciar el backend.

**Error:**

```
Backend failed to start within 30 seconds
```

**Soluciones:**

1. Compruebe si el backend ya se está ejecutando (ciérrelo primero).
2. Compruebe que el cortafuegos Windows no lo está bloqueando.
3. Pruebe con un puerto diferente:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Forzar el reinicio del backend:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Problemas de licencia/autenticación

**Error:**

```
Chloros+ license required for CLI access
```

**Soluciones:**

1. Compruebe que tiene una suscripción activa a Chloros+.
2. Inicie sesión con sus credenciales:

```powershell
chloros-cli login user@example.com 'password'
```

3. Compruebe el estado de la licencia:

```powershell
chloros-cli status
```

4. Póngase en contacto con el servicio de asistencia: info@mapir.camera

***

### No se han encontrado imágenes.

**Error:**

```
No images found in the specified folder
```

**Soluciones:**

1. Compruebe que la carpeta contiene formatos compatibles (.RAW, .TIF, .JPG).
2. Compruebe que la ruta de la carpeta es correcta (utilice comillas para las rutas con espacios).
3. Asegúrese de tener permisos de lectura para la carpeta.
4. Compruebe que las extensiones de los archivos sean correctas.

***

### El procesamiento se detiene o se cuelga

**Soluciones:**

1. Compruebe el espacio disponible en el disco (asegúrese de que haya suficiente para la salida).
2. Cierre otras aplicaciones para liberar memoria.
3. Reduzca el número de imágenes (procese por lotes).

***

### Puerto ya en uso

**Error:**

```
Port 5000 is already in use
```

**Solución:**

Especifique un puerto diferente:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Preguntas frecuentes

### P: ¿Necesito una licencia para el CLI?

**R:** ¡Sí! El CLI requiere una **licencia Chloros+ de pago**.

* ❌ Plan estándar (gratuito): CLI desactivado
* ✅ Planes Chloros+ (de pago): CLI totalmente habilitado

Suscríbase en: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### P: ¿Puedo utilizar CLI en un servidor sin GUI?

**R:** ¡Sí! CLI funciona completamente sin interfaz gráfica. Requisitos:

* Windows Server 2016 o posterior
* Visual C++ Redistributable instalado
* RAM suficiente (mínimo 8 GB, recomendado 16 GB)
* Activación única de la licencia GUI en cualquier máquina

***

### P: ¿Dónde se guardan las imágenes procesadas?

**R:** Por defecto, las imágenes procesadas se guardan en la **misma carpeta que la entrada** en subcarpetas del modelo de cámara (por ejemplo, `Survey3N_RGN/`).

Utilice la opción `-o` para especificar una carpeta de salida diferente:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### P: ¿Puedo procesar varias carpetas a la vez?

**R:** No directamente con un solo comando, pero puede utilizar scripts para procesar carpetas de forma secuencial. Consulte la sección [Automatización y scripts](CLI.md#automation--scripting).

***

### P: ¿Cómo guardo la salida CLI en un archivo de registro?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Lote:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

***

### P: ¿Qué ocurre si pulso Ctrl+C durante el procesamiento?

**R:** CLI hará lo siguiente:

1. Detendrá el procesamiento de forma ordenada.
2. Apagará el backend.
3. Saldrá con el código 130.

Es posible que queden imágenes parcialmente procesadas en la carpeta de salida.

***

### P: ¿Puedo automatizar el procesamiento de CLI?

**R:** ¡Por supuesto! CLI está diseñado para la automatización. Consulte [Automatización y secuencias de comandos](CLI.md#automation--scripting) para ver ejemplos de PowerShell, Batch y Python.

***

### P: ¿Cómo puedo comprobar la versión de CLI?

**R:**

```powershell
chloros-cli --version
```

**Salida:**

```
Chloros CLI 1.0.2
```

***

## Obtener ayuda

### Ayuda de la línea de comandos

Vea la información de ayuda directamente en CLI:

```powershell
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
* **Precios**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Ejemplos completos

### Ejemplo 1: Procesamiento básico

Procesamiento con la configuración predeterminada (viñeta, reflectancia):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Ejemplo 2: Resultado científico de alta calidad

32 bits flotantes TIFF:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Ejemplo 3: Procesamiento rápido de vista previa

8 bits PNG sin calibración para una revisión rápida:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Ejemplo 4: Procesamiento corregido con PPK

Aplicar correcciones PPK con reflectancia:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Ejemplo 5: Ubicación de salida personalizada

Procesar en una unidad diferente con un formato específico:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Ejemplo 6: Flujo de trabajo de autenticación

Flujo de autenticación completo:

```powershell
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
chloros-cli process "C:\Datasets\Field_A"

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Ejemplo 7: Uso multilingüe

Cambiar el idioma de la interfaz:

```powershell
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
chloros-cli process "C:\Vuelos\Campo_A"

# Change back to English
chloros-cli language en
```
