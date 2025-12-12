# Inicio de Sesión Chloros+

## Inicio de Sesión en Chloros y Chloros (Navegador)

El menú lateral de usuario <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> le permite iniciar sesión en su cuenta Chloros+ y desbloquear funciones adicionales.

Cuando inicie sesión, se mostrarán los detalles de su cuenta:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>

## Inicio de Sesión CLI

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

### Expiración del Plan

La expiración del plan en la GUI muestra cuándo su licencia dejará de ser válida. Para suscripciones mensuales recurrentes, la expiración es al final del mes. Para suscripciones anuales, es un año después de que comenzó la suscripción. La verificación de licencia requiere una conexión a internet mensual para verificar, con un período de gracia de 30 días.

### Límite de Dispositivos

Cada plan Chloros+ ofrece un número diferente de dispositivos registrados. Cada dispositivo en el que inicie sesión con una cuenta Chloros+ contará para su número de dispositivos registrados. Puede renombrar y eliminar un dispositivo en la página de su cuenta MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plan Chloros+</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Dispositivos Soportados</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
