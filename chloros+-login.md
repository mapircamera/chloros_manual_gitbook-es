# Inicio de sesión en Chloros+

## Inicio de sesión en la interfaz gráfica de usuario (GUI)

El menú lateral de la página <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> te permite iniciar sesión en tu cuenta de Chloros+ y acceder a funciones adicionales.

**Solo es necesario iniciar sesión una vez por equipo.** La interfaz gráfica de usuario, CLI y Python SDK comparten la misma sesión almacenada en caché ; al iniciar sesión a través de la interfaz gráfica de usuario de escritorio, también se activan CLI y SDK en ese equipo (y viceversa a través de `chloros-cli login`).

Una vez iniciada la sesión, se mostrarán los datos de tu cuenta:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.2.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->
## Niveles de planes

| Plan | `plan_id` | Tipo |
| --- | --- | --- |
| Iron | `0` | Gratuito |
| Copper | `1` | De pago (Chloros+) |
| Bronce | `2` | De pago (Chloros+) |
| Plata | `3` | De pago (Chloros+) |
| Oro | `4` | De pago (Chloros+) |

Consulta [los planes y precios](https://cloud.mapir.camera/pricing) para saber qué incluye cada nivel de pago.

### El acceso a CLI / SDK requiere un nivel de pago

El acceso a CLI, Python y SDK requiere **cualquier nivel de pago Chloros+ (Copper o superior)**. Esto se aplica**del lado del servidor**: cada solicitud CLI/SDK debe incluir tanto una sesión activa como un plan de pago:

| Estado HTTP | `error_code` | Significado | Solución |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | No ha iniciado sesión en este equipo | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Ha iniciado sesión, pero el nivel del plan es demasiado bajo (nivel gratuito «Iron») | Actualizar a cualquier plan Chloros+ de pago |

`chloros-cli status` sigue estando disponible en el nivel gratuito, por lo que siempre podrás ver tu plan actual y el motivo por el que se te deniega el acceso.

### Límites de hardware conectado por plan

Cada plan limita el número de cámaras LATTICE y sensores de luz DAQ que se pueden conectar en directo a la vez:

| Plan | Cámaras LATTICE | Sensores de luz DAQ |
| --- | --- | --- |
| Iron (gratuito / sin iniciar sesión) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Inicio de sesión en CLI

Inicia sesión con tus credenciales de Chloros+ para habilitar el procesamiento de CLI. En Linux (sin interfaz gráfica), esta es la única forma de activar su licencia.

**Sintaxis:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Usuarios de SDK**: Python SDK también ofrece un método programático `logout()` para borrar las credenciales almacenadas en caché. Consulta la [Referencia de SDK](reference/sdk-reference.md) para obtener más detalles.
{% endhint %}

**Ejemplo:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caracteres especiales**: Utiliza comillas simples para encerrar las contraseñas que contengan caracteres como `$`, `!` o espacios.
{% endhint %}

**Salida:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.2.0"; capture a successful login with the current output format. -->
### Almacenamiento de credenciales

Las credenciales y la configuración almacenadas en caché se guardan en la carpeta `.chloros` del directorio de inicio de usuario en **todas las plataformas**:

| Plataforma | Ruta de la caché de credenciales |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

### Caducidad del plan y periodo de gracia sin conexión

La fecha de caducidad del plan que aparece en la interfaz gráfica de usuario indica cuándo dejará de ser válida tu licencia. En el caso de las suscripciones mensuales recurrentes, la caducidad se produce a final de mes; en el caso de las suscripciones anuales, un año después de haber iniciado la suscripción.

Chloros valida tu licencia en línea, pero permite trabajar sin conexión durante un periodo de gracia:

* Las validaciones correctas del servidor se almacenan en caché durante **5 minutos**, por lo que el uso normal genera muy pocas solicitudes de licencia.
* Una caché de licencias firmadas y vinculadas al equipo cubre periodos sin conexión más largos: **30 días para los planes mensuales**y**hasta la fecha de vencimiento de tu suscripción (como máximo 365 días) para los planes anuales**.
* Cuando vence el periodo de gracia, el plan pasa al nivel gratuito «Iron» hasta que el equipo pueda conectarse al servidor de licencias una vez; el acceso se reanuda tras la siguiente comprobación satisfactoria.

### Límite de dispositivos

Cada plan Chloros+ ofrece un número diferente de dispositivos registrados. Cada dispositivo en el que inicies sesión con una cuenta Chloros+ cuenta para tu número de dispositivos registrados. Puedes cambiar el nombre y eliminar un dispositivo en la página de tu cuenta de MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plan Chloros+</th><th align="center">COBRE</th><th align="center">BRONCE</th><th align="center">PLATA</th><th align="center">ORO</th></tr></thead><tbody><tr><td align="right">Dispositivos compatibles</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>El número exacto de dispositivos permitidos en tu cuenta aparece en la página de tu cuenta de MAPIR Cloud. Al cerrar sesión en un dispositivo, se libera su plaza de forma definitiva, y un dispositivo que ya esté registrado siempre puede volver a iniciar sesión, incluso cuando la cuenta haya alcanzado su límite de dispositivos.
