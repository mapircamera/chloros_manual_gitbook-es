# CLI : Línea de Comandos

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>

El **CLI de Chloros** proporciona potente acceso de línea de comandos al motor de procesamiento de imágenes de Chloros, permitiendo automatización, scripting y operación sin interfaz gráfica para sus flujos de trabajo de imágenes.

### Características Clave

* 🚀 **Automatización** - Script de procesamiento por lotes de múltiples conjuntos de datos
* 🔗 **Integración** - Incrustar en flujos de trabajo y pipelines existentes
* 💻 **Operación Sin Interfaz** - Ejecutar sin GUI
* 🌍 **Multi-Idioma** - Soporte para 38 idiomas
* ⚡ **Procesamiento Paralelo** - Se escala dinámicamente a su CPU (hasta 16 workers paralelos)

### Requisitos

| Requisito              | Detalles                                                              |
| ---------------------- | --------------------------------------------------------------------- |
| **Sistema Operativo**  | Windows 10/11 (64-bit)                                                |
| **Licencia**           | Chloros+ ([plan pago requerido](https://cloud.mapir.camera/pricing)) |
| **Memoria**            | 8GB RAM mínimo (16GB recomendado)                                     |
| **Internet**           | Requerido para activación de licencia                                 |
| **Espacio en Disco**   | Varía según el tamaño del proyecto                                    |

{% hint style="warning" %}
**Requisito de Licencia**: El CLI requiere una suscripción paga de Chloros+. Los planes estándar (gratuitos) no tienen acceso CLI. Visite [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) para actualizar.
{% endhint %}

## Inicio Rápido

### Instalación

El CLI se incluye automáticamente con el instalador de Chloros:

1. Descargue y ejecute **Chloros Installer.exe**
2. Complete el asistente de instalación
3. CLI instalado en: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
El instalador agrega automáticamente `chloros-cli` a su PATH del sistema. Reinicie su terminal después de la instalación.
{% endhint %}

### Configuración Inicial

Antes de usar el CLI, active su licencia Chloros+:

```bash
# Inicie sesión con su cuenta Chloros+
chloros-cli login usuario@ejemplo.com 'su_contraseña'

# Verifique el estado de la licencia
chloros-cli status

# Procese su primer proyecto
chloros-cli process "C:\Images\Dataset001"
```

### Uso Básico

Procese una carpeta con configuración predeterminada:

```powershell
chloros-cli process "C:\Images\Dataset001"
```

***

## Referencia de Comandos

### Sintaxis General

```
chloros-cli [opciones-globales] <comando> [opciones-comando]
```

***

## Comandos

### `process` - Procesar Imágenes

Procese imágenes en una carpeta con calibración.

**Sintaxis:**

```bash
chloros-cli process <carpeta-entrada> [opciones]
```

**Ejemplo:**

```powershell
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance
```

#### Opciones del Comando Process

| Opción                | Tipo    | Predeterminado  | Descripción                                                                               |
| --------------------- | ------- | --------------- | ----------------------------------------------------------------------------------------- |
| `<carpeta-entrada>`   | Path    | _Requerido_     | Carpeta que contiene imágenes multiespectrales RAW/JPG                                    |
| `-o, --output`        | Path    | Mismo que entry | Carpeta de salida para imágenes procesadas                                                |
| `-n, --project-name`  | String  | Auto-generado   | Nombre de proyecto personalizado                                                          |
| `--vignette`          | Flag    | Habilitado      | Habilitar corrección de viñeta                                                            |
| `--no-vignette`       | Flag    | -               | Deshabilitar corrección de viñeta                                                         |
| `--reflectance`       | Flag    | Habilitado      | Habilitar calibración de reflectancia                                                     |
| `--no-reflectance`    | Flag    | -               | Deshabilitar calibración de reflectancia                                                  |
| `--ppk`               | Flag    | Deshabilitado   | Aplicar correcciones PPK desde datos del sensor de luz .daq                               |
| `--format`            | Choice  | TIFF (16-bit)   | Formato de salida: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Integer | Auto            | Tamaño mínimo de objetivo en píxeles para detección de panel de calibración              |
| `--target-clustering` | Integer | Auto            | Umbral de agrupación de objetivo (0-100)                                                  |
| `--exposure-pin-1`    | String  | None            | Bloquear exposición para modelo de cámara (Pin 1)                                         |
| `--exposure-pin-2`    | String  | None            | Bloquear exposición para modelo de cámara (Pin 2)                                         |
| `--recal-interval`    | Integer | Auto            | Intervalo de recalibración en segundos                                                    |
| `--timezone-offset`   | Integer | 0               | Desplazamiento de zona horaria en horas                                                   |

***

### `login` - Autenticar Cuenta

Inicie sesión con sus credenciales de Chloros+ para habilitar el procesamiento CLI.

**Sintaxis:**

```bash
chloros-cli login <correo> <contraseña>
```

**Ejemplo:**

```powershell
chloros-cli login usuario@ejemplo.com 'MiC0ntraseñ@123'
```

{% hint style="warning" %}
**Caracteres Especiales**: Use comillas simples alrededor de contraseñas que contengan caracteres como `$`, `!`, o espacios.
{% endhint %}

**Salida:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>

***

### `logout` - Borrar Credenciales

Borre las credenciales almacenadas y cierre sesión de su cuenta.

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
✓ Cierre de sesión exitoso
ℹ Credenciales borradas de la caché
```

***

### `status` - Verificar Estado de Licencia

Mostrar el estado actual de licencia y autenticación.

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
║  INFORMACIÓN DE LICENCIA Y CUENTA    ║
╚══════════════════════════════════════╝

📧 Correo: usuario@ejemplo.com
📋 Plan: Chloros+ Profesional
🔓 Acceso API/CLI: Habilitado
✓ Estado: Activo
```

***

### `export-status` - Verificar Progreso de Exportación

Monitorear el progreso de exportación del Hilo 4 durante o después del procesamiento.

**Sintaxis:**

```bash
chloros-cli export-status
```

**Ejemplo:**

```powershell
chloros-cli export-status
```

**Caso de Uso:** Llame a este comando mientras el procesamiento se está ejecutando para verificar el progreso de exportación.

***

### `language` - Gestionar Idioma de Interfaz

Ver o cambiar el idioma de interfaz del CLI.

**Sintaxis:**

```bash
# Mostrar idioma actual
chloros-cli language

# Listar todos los idiomas disponibles
chloros-cli language --list

# Establecer un idioma específico
chloros-cli language <código-idioma>
```

**Ejemplos:**

```powershell
# Ver idioma actual
chloros-cli language

# Listar los 38 idiomas compatibles
chloros-cli language --list

# Cambiar a español
chloros-cli language es

# Cambiar a japonés
chloros-cli language ja
```

#### Idiomas Compatibles (38 en Total)

| Código  | Idioma                | Nombre Nativo        |
| ------- | --------------------- | -------------------- |
| `en`    | English               | English              |
| `es`    | Spanish               | Español              |
| `pt`    | Portuguese            | Português            |
| `fr`    | French                | Français             |
| `de`    | German                | Deutsch              |
| `it`    | Italian               | Italiano             |
| `ja`    | Japanese              | 日本語                  |
| `ko`    | Korean                | 한국어                  |
| `zh`    | Chinese (Simplified)  | 简体中文                 |
| `zh-TW` | Chinese (Traditional) | 繁體中文                 |
| `ru`    | Russian               | Русский              |
| `nl`    | Dutch                 | Nederlands           |
| `ar`    | Arabic                | العربية              |
| `pl`    | Polish                | Polski               |
| `tr`    | Turkish               | Türkçe               |
| `hi`    | Hindi                 | हिंदी                |
| `id`    | Indonesian            | Bahasa Indonesia     |
| `vi`    | Vietnamese            | Tiếng Việt           |
| `th`    | Thai                  | ไทย                  |
| `sv`    | Swedish               | Svenska              |
| `da`    | Danish                | Dansk                |
| `no`    | Norwegian             | Norsk                |
| `fi`    | Finnish               | Suomi                |
| `el`    | Greek                 | Ελληνικά             |
| `cs`    | Czech                 | Čeština              |
| `hu`    | Hungarian             | Magyar               |
| `ro`    | Romanian              | Română               |
| `uk`    | Ukrainian             | Українська           |
| `pt-BR` | Brazilian Portuguese  | Português Brasileiro |
| `zh-HK` | Cantonese             | 粵語                   |
| `ms`    | Malay                 | Bahasa Melayu        |
| `sk`    | Slovak                | Slovenčina           |
| `bg`    | Bulgarian             | Български            |
| `hr`    | Croatian              | Hrvatski             |
| `lt`    | Lithuanian            | Lietuvių             |
| `lv`    | Latvian               | Latviešu             |
| `et`    | Estonian              | Eesti                |
| `sl`    | Slovenian             | Slovenščina          |

{% hint style="success" %}
**Persistencia Automática**: Su preferencia de idioma se guarda en `~/.chloros/cli_language.json` y persiste en todas las sesiones.
{% endhint %}

***

### `set-project-folder` - Establecer Carpeta de Proyecto Predeterminada

Cambiar la ubicación predeterminada de la carpeta de proyecto (compartida con GUI).

**Sintaxis:**

```bash
chloros-cli set-project-folder <ruta-carpeta>
```

**Ejemplo:**

```powershell
chloros-cli set-project-folder "C:\Projects\2025"
```

***

### `get-project-folder` - Mostrar Carpeta de Proyecto

Mostrar la ubicación actual de la carpeta de proyecto predeterminada.

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
ℹ Carpeta de proyecto actual: C:\Projects\2025
```

***

### `reset-project-folder` - Restablecer a Predeterminado

Restablecer la carpeta de proyecto a la ubicación predeterminada.

**Sintaxis:**

```bash
chloros-cli reset-project-folder
```

***

## Opciones Globales

Estas opciones se aplican a todos los comandos:

| Opción          | Tipo    | Predeterminado | Descripción                                                   |
| --------------- | ------- | -------------- | ------------------------------------------------------------- |
| `--backend-exe` | Path    | Auto-detectado | Ruta al ejecutable backend                                    |
| `--port`        | Integer | 5000           | Número de puerto API backend                                  |
| `--restart`     | Flag    | -              | Forzar reinicio del backend (mata procesos existentes)        |
| `--version`     | Flag    | -              | Mostrar información de versión y salir                        |
| `--help`        | Flag    | -              | Mostrar información de ayuda y salir                          |

**Ejemplo con Opciones Globales:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

***

## Guía de Configuración de Procesamiento

### Procesamiento Paralelo

El CLI de Chloros+ **escala automáticamente** el procesamiento paralelo para coincidir con las capacidades de su computadora:

**Cómo Funciona:**

* Detecta sus núcleos de CPU y RAM
* Asigna workers: **2× núcleos de CPU** (usa hyperthreading)
* **Máximo: 16 workers paralelos** (para estabilidad)

**Niveles de Sistema:**

| Tipo de Sistema  | CPU        | RAM      | Workers  | Rendimiento         |
| ---------------- | ---------- | -------- | -------- | ------------------- |
| **Gama Alta**    | 16+ núcleos | 32+ GB   | Hasta 16 | Velocidad máxima    |
| **Gama Media**   | 8-15 núcleos | 16-31 GB | 8-16     | Excelente velocidad |
| **Gama Baja**    | 4-7 núcleos  | 8-15 GB  | 4-8      | Buena velocidad     |

{% hint style="success" %}
**Optimización Automática**: El CLI detecta automáticamente las especificaciones de su sistema y configura el procesamiento paralelo óptimo. ¡No se necesita configuración manual!
{% endhint %}

### Métodos de Debayer

El CLI usa **Alta Calidad (Más Rápido)** como algoritmo de debayer predeterminado y recomendado:

| Método                         | Calidad | Velocidad | Descripción                                             |
| ------------------------------ | ------- | --------- | ------------------------------------------------------- |
| **Alta Calidad (Más Rápido)** ⭐ | ⭐⭐⭐⭐    | ⚡⚡⚡       | Algoritmo con detección de bordes (predeterminado, recomendado) |

### Corrección de Viñeta

**Qué hace:** Corrige la caída de luz en los bordes de la imagen (esquinas más oscuras comunes en imágenes de cámara).

* **Habilitado por defecto** - La mayoría de los usuarios deben mantener esto habilitado
* Use `--no-vignette` para deshabilitar

{% hint style="success" %}
**Recomendación**: Siempre habilite la corrección de viñeta para garantizar un brillo uniforme en todo el cuadro.
{% endhint %}

### Calibración de Reflectancia

Convierte valores crudos del sensor a porcentajes de reflectancia estandarizados usando paneles de calibración.

* **Habilitado por defecto** - Esencial para análisis de vegetación
* Requiere paneles objetivo de calibración en las imágenes
* Use `--no-reflectance` para deshabilitar

{% hint style="info" %}
**Requisitos**: Asegúrese de que los paneles de calibración estén correctamente expuestos y visibles en sus imágenes para una conversión de reflectancia precisa.
{% endhint %}

### Correcciones PPK

**Qué hace:** Aplica correcciones Cinemáticas Post-Procesadas usando datos de registro DAQ-A-SD para mejorar la precisión del GPS.

* **Deshabilitado por defecto**
* Use `--ppk` para habilitar
* Requiere archivos .daq en la carpeta del proyecto del sensor de luz MAPIR DAQ-A-SD.

### Formatos de Salida

<table><thead><tr><th width="197">Formato</th><th width="130.20001220703125">Profundidad de Bits</th><th width="116.5999755859375">Tamaño de Archivo</th><th>Mejor Para</th></tr></thead><tbody><tr><td><strong>TIFF (16-bit)</strong> ⭐</td><td>16-bit integer</td><td>Grande</td><td>Análisis GIS, fotogrametría (recomendado)</td></tr><tr><td><strong>TIFF (32-bit, Porcentaje)</strong></td><td>32-bit float</td><td>Muy Grande</td><td>Análisis científico, investigación</td></tr><tr><td><strong>PNG (8-bit)</strong></td><td>8-bit integer</td><td>Medio</td><td>Inspección visual, compartir en web</td></tr><tr><td><strong>JPG (8-bit)</strong></td><td>8-bit integer</td><td>Pequeño</td><td>Vista previa rápida, salida comprimida</td></tr></tbody></table>

***

## Automatización y Scripting

### Procesamiento por Lotes en PowerShell

Procese múltiples carpetas de conjuntos de datos automáticamente:

```powershell
# procesar_todos_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Procesando $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) completo" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) falló" -ForegroundColor Red
    }
}

Write-Host "¡Todos los conjuntos de datos procesados!" -ForegroundColor Green
```

### Script de Lote de Windows

Bucle simple para procesamiento por lotes:

```batch
@echo off
echo Iniciando procesamiento por lotes...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Procesando: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo ÉXITO: %%i procesado
    ) else (
        echo ERROR: %%i falló
    )
)

echo.
echo ¡Todos los conjuntos de datos procesados!
pause
```

### Script de Automatización en Python

Automatización avanzada con manejo de errores:

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def procesar_dataset(carpeta_entrada):
    """Procesar una carpeta usando Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(carpeta_entrada)]
    
    # Ejecutar comando
    resultado = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return resultado.returncode == 0, resultado.stdout, resultado.stderr

def main():
    """Procesar todos los conjuntos de datos en un directorio"""
    dir_datasets = Path('C:/Datasets/2025')
    archivo_log = Path('registro_procesamiento.txt')
    
    exitosos = []
    fallidos = []
    
    # Iniciar procesamiento
    print(f"Iniciando procesamiento por lotes: {datetime.now()}")
    print(f"Escaneando: {dir_datasets}")
    print("=" * 60)
    
    for carpeta_dataset in sorted(dir_datasets.iterdir()):
        if not carpeta_dataset.is_dir():
            continue
        
        print(f"\nProcesando: {carpeta_dataset.name}")
        
        exito, stdout, stderr = procesar_dataset(carpeta_dataset)
        
        if exito:
            print(f"✓ {carpeta_dataset.name} - ÉXITO")
            exitosos.append(carpeta_dataset.name)
        else:
            print(f"✗ {carpeta_dataset.name} - FALLÓ")
            fallidos.append(carpeta_dataset.name)
            
            # Registrar detalles de error
            with open(archivo_log, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {carpeta_dataset.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Imprimir resumen
    print("\n" + "=" * 60)
    print(f"RESUMEN - Completado: {datetime.now()}")
    print(f"  Exitosos: {len(exitosos)}")
    print(f"  Fallidos: {len(fallidos)}")
    
    if fallidos:
        print(f"\nCarpetas fallidas:")
        for carpeta in fallidos:
            print(f"  - {carpeta}")
        print(f"\nRevise {archivo_log} para detalles de errores")
        sys.exit(1)
    else:
        print("\n¡Todos los conjuntos de datos procesados con éxito!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Flujo de Trabajo de Procesamiento

### Flujo de Trabajo Estándar

1. **Entrada**: Carpeta que contiene pares de imágenes RAW/JPG
2. **Descubrimiento**: El CLI escanea automáticamente archivos de imagen compatibles
3. **Procesamiento**: El modo paralelo se escala a sus núcleos de CPU (Chloros+)
4. **Salida**: Crea subcarpetas de modelo de cámara con imágenes procesadas

### Ejemplo de Estructura de Salida

```
MiProyecto/
├── project.json                             # Metadatos del proyecto
├── 2025_0203_193056_008.JPG                # JPG original
├── 2025_0203_193055_007.RAW                # RAW original
└── Survey3N_RGN/                           # Salidas procesadas ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Reflectancia calibrada
    ├── 2025_0203_193056_008_Target.tif        # Detección de objetivo
    └── ...
```

### Estimaciones de Tiempo de Procesamiento

Tiempos de procesamiento típicos para 100 imágenes (12MP cada una):

| Modo              | Tiempo     | Hardware                                     |
| ----------------- | ---------- | -------------------------------------------- |
| **Modo Paralelo** | 5-10 min   | i7/Ryzen 7, 16GB RAM, SSD (hasta 16 workers) |
| **Modo Paralelo** | 10-15 min  | i5/Ryzen 5, 8GB RAM, HDD (hasta 8 workers)   |

{% hint style="info" %}
**Consejo de Rendimiento**: El tiempo de procesamiento varía según el conteo de imágenes, resolución y especificaciones de la computadora.
{% endhint %}

***

## Solución de Problemas

### CLI No Encontrado

**Error:**

```
'chloros-cli' no se reconoce como un comando interno o externo
```

**Soluciones:**

1. Verifique la ubicación de instalación:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Use la ruta completa si no está en PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Agregue a PATH manualmente:
   * Abra Propiedades del Sistema → Variables de Entorno
   * Edite la variable PATH
   * Agregue: `C:\Program Files\Chloros\resources\cli`
   * Reinicie la terminal

***

### Fallo al Iniciar Backend

**Error:**

```
El backend no pudo iniciarse en 30 segundos
```

**Soluciones:**

1. Verifique si el backend ya está en ejecución (ciérrelo primero)
2. Verifique que el Firewall de Windows no esté bloqueando
3. Pruebe con un puerto diferente:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

4. Fuerce el reinicio del backend:

```powershell
chloros-cli --restart process "C:\Datasets\Field_A"
```

***

### Problemas de Licencia / Autenticación

**Error:**

```
Se requiere licencia Chloros+ para acceso CLI
```

**Soluciones:**

1. Verifique que tiene una suscripción activa de Chloros+
2. Inicie sesión con sus credenciales:

```powershell
chloros-cli login usuario@ejemplo.com 'contraseña'
```

3. Verifique el estado de la licencia:

```powershell
chloros-cli status
```

4. Contacte soporte: info@mapir.camera

***

### No Se Encontraron Imágenes

**Error:**

```
No se encontraron imágenes en la carpeta especificada
```

**Soluciones:**

1. Verifique que la carpeta contiene formatos compatibles (.RAW, .TIF, .JPG)
2. Verifique que la ruta de la carpeta es correcta (use comillas para rutas con espacios)
3. Asegúrese de tener permisos de lectura para la carpeta
4. Verifique que las extensiones de archivo son correctas

***

### El Procesamiento se Estanca o Cuelga

**Soluciones:**

1. Verifique el espacio disponible en disco (asegure suficiente para salida)
2. Cierre otras aplicaciones para liberar memoria
3. Reduzca el conteo de imágenes (procese en lotes)

***

### Puerto Ya en Uso

**Error:**

```
El puerto 5000 ya está en uso
```

**Solución:**

Especifique un puerto diferente:

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

***

## Preguntas Frecuentes

### P: ¿Necesito una licencia para el CLI?

**R:** ¡Sí! El CLI requiere una **licencia Chloros+ pagada**.

* ❌ Plan estándar (gratuito): CLI deshabilitado
* ✅ Planes Chloros+ (pagados): CLI completamente habilitado

Suscríbase en: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### P: ¿Puedo usar el CLI en un servidor sin GUI?

**R:** ¡Sí! El CLI se ejecuta completamente sin interfaz. Requisitos:

* Windows Server 2016 o posterior
* Visual C++ Redistributable instalado
* RAM suficiente (8GB mínimo, 16GB recomendado)
* Activación de licencia GUI única en cualquier máquina

***

### P: ¿Dónde se guardan las imágenes procesadas?

**R:** Por defecto, las imágenes procesadas se guardan en **la misma carpeta que la entrada** en subcarpetas de modelo de cámara (ej., `Survey3N_RGN/`).

Use la opción `-o` para especificar una carpeta de salida diferente:

```powershell
chloros-cli process "C:\Input" -o "D:\Output"
```

***

### P: ¿Puedo procesar múltiples carpetas a la vez?

**R:** No directamente en un comando, pero puede usar scripting para procesar carpetas secuencialmente. Consulte la sección [Automatización y Scripting](CLI.md#automatización-y-scripting).

***

### P: ¿Cómo guardo la salida del CLI en un archivo de registro?

**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "procesamiento.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > procesamiento.log 2>&1
```

***

### P: ¿Qué sucede si presiono Ctrl+C durante el procesamiento?

**R:** El CLI:

1. Detendrá el procesamiento gradualmente
2. Apagará el backend
3. Saldrá con código 130

Las imágenes parcialmente procesadas pueden permanecer en la carpeta de salida.

***

### P: ¿Puedo automatizar el procesamiento CLI?

**R:** ¡Absolutamente! El CLI está diseñado para automatización. Consulte [Automatización y Scripting](CLI.md#automatización-y-scripting) para ejemplos de PowerShell, Batch y Python.

***

### P: ¿Cómo verifico la versión del CLI?

**R:**

```powershell
chloros-cli --version
```

**Salida:**

```
Chloros CLI 1.0.2
```

***

## Obtener Ayuda

### Ayuda de Línea de Comandos

Ver información de ayuda directamente en el CLI:

```powershell
# Ayuda general
chloros-cli --help

# Ayuda específica del comando
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Canales de Soporte

* **Correo**: info@mapir.camera
* **Sitio web**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Precios**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

## Ejemplos Completos

### Ejemplo 1: Procesamiento Básico

Procese con configuración predeterminada (viñeta, reflectancia):

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

***

### Ejemplo 2: Salida Científica de Alta Calidad

TIFF flotante de 32 bits:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

***

### Ejemplo 3: Procesamiento de Vista Previa Rápida

PNG de 8 bits sin calibración para revisión rápida:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

***

### Ejemplo 4: Procesamiento Corregido con PPK

Aplicar correcciones PPK con reflectancia:

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

***

### Ejemplo 5: Ubicación de Salida Personalizada

Procese a una unidad diferente con formato específico:

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

***

### Ejemplo 6: Flujo de Trabajo de Autenticación

Flujo de autenticación completo:

```powershell
# Paso 1: Inicio de sesión
chloros-cli login usuario@ejemplo.com 'MiC0ntraseñ@'

# Paso 2: Verificar estado
chloros-cli status

# Paso 3: Procesar imágenes
chloros-cli process "C:\Datasets\Field_A"

# Paso 4: Cierre de sesión (opcional, al cambiar cuentas)
chloros-cli logout
```

***

### Ejemplo 7: Uso Multi-Idioma

Cambiar idioma de interfaz:

```powershell
# Listar idiomas disponibles
chloros-cli language --list

# Cambiar a español
chloros-cli language es

# Procesar con interfaz en español
chloros-cli process "C:\Vuelos\Campo_A"

# Cambiar de vuelta a inglés
chloros-cli language en
```
