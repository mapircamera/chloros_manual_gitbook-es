# Chloros+ Inicio de sesión

## Chloros y Chloros (navegador) Inicio de sesión

El <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> barra lateral del usuario le permite iniciar sesión en su cuenta Chloros+ y desbloquear funciones adicionales.

Al iniciar sesión, se mostrarán los detalles de su cuenta:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Iniciar sesión

Inicie sesión con sus credenciales de Chloros+ para habilitar el procesamiento de CLI.

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

**Salida:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Caducidad del plan

La caducidad del plan en la interfaz gráfica de usuario muestra cuándo dejará de ser válida su licencia. En el caso de las suscripciones mensuales recurrentes, la caducidad es al final del mes. En el caso de las suscripciones anuales, es un año después de haber iniciado la suscripción. La comprobación de la licencia requiere una conexión a Internet mensual para verificarla, con un periodo de gracia de 30 días.

### Límite de dispositivos

Cada plan Chloros+ ofrece un número diferente de dispositivos registrados. Cada dispositivo en el que inicie sesión con una cuenta Chloros+ contará para su número de dispositivos registrados. Puede cambiar el nombre y eliminar un dispositivo en la página de su cuenta MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plan Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">PLATA</th><th align="center">ORO</th></tr></thead><tbody><tr><td align="right">Dispositivos compatibles</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
