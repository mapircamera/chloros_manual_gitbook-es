# Inicio de sesión en Chloros+

## Inicio de sesión en Chloros y Chloros (navegador)

El <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> menú lateral te permite iniciar sesión en tu cuenta Chloros+ y desbloquear funciones adicionales.

Una vez iniciada la sesión, se mostrarán los datos de tu cuenta:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## Inicio de sesión en CLI

Inicie sesión con sus credenciales de Chloros+ para habilitar el procesamiento de CLI. En Linux (sin interfaz gráfica), esta es la única forma de activar su licencia.

**Sintaxis:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Usuarios de SDK**: El Python SDK también proporciona un método programático `logout()` para borrar las credenciales almacenadas en caché. Consulte la [documentación de Python SDK](api-python-sdk.md#logout) para obtener más detalles.
{% endhint %}

**Ejemplo:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caracteres especiales**: Utilice comillas simples alrededor de las contraseñas que contengan caracteres como `$`, `!` o espacios.
{% endhint %}

**Salida:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Almacenamiento de credenciales

Las credenciales almacenadas en caché se guardan en una ubicación específica de la plataforma:

| Plataforma | Ruta de la caché de credenciales |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Caducidad del plan

La fecha de caducidad del plan que aparece en la interfaz gráfica de usuario indica cuándo dejará de ser válida su licencia. En el caso de las suscripciones mensuales recurrentes, la caducidad es a final de mes. Para las suscripciones anuales, es un año después de haber iniciado la suscripción. La comprobación de la licencia requiere una conexión a Internet mensual para verificarla, con un periodo de gracia de 30 días.

### Límite de dispositivos

Cada plan Chloros+ ofrece un número diferente de dispositivos registrados. Cada dispositivo en el que inicie sesión con una cuenta Chloros+ contará para su número de dispositivos registrados. Puede renombrar y eliminar un dispositivo en la página de su cuenta MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plan Chloros+</th><th align="center">COPPER</th><th align="center">BRONCE</th><th align="center">PLATA</th><th align="center">ORO</th></tr></thead><tbody><tr><td align="right">Dispositivos compatibles</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
