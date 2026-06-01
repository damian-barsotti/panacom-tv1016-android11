# Guía: Actualizar firmware y configurar el control remoto del Panacom TV-1016

Tutorial para reemplazar el firmware del **Panacom TV-1016** (SoC **Amlogic S905W2**, 2 GB RAM, 16 GB eMMC) por **Android 11**, y dejar funcionando el **control remoto IR original** con todas sus teclas, incluido el botón de modo mouse.

> **Aviso**
> Este procedimiento borra el contenido del dispositivo y modifica particiones del sistema. Hay riesgo de dejar el equipo inoperativo si algo sale mal. Hacelo bajo tu responsabilidad. Conservá siempre la imagen de boot original y el firmware completo para poder recuperar con el USB Burning Tool.

---

## Tabla de contenidos

1. [Requisitos](#1-requisitos)
2. [Flashear Android 11 (firmware X98Q)](#2-flashear-android-11-firmware-x98q)
3. [Primer arranque y WiFi](#3-primer-arranque-y-wifi)
   - [Opcional: ejecutar los comandos por SSH desde la PC](#opcional-ejecutar-los-comandos-por-ssh-desde-la-pc)
4. [Decodificar los códigos del control remoto IR](#4-decodificar-los-códigos-del-control-remoto-ir)
5. [Instalar Magisk (root)](#5-instalar-magisk-root)
6. [Cargar el control remoto de forma persistente](#6-cargar-el-control-remoto-de-forma-persistente)
7. [Modo mouse por software (opcional)](#7-modo-mouse-por-software-opcional)
8. [Notas finales](#8-notas-finales)

---

## 1. Requisitos

- Una **PC con Windows** (para el USB Burning Tool).
- **Amlogic USB Burning Tool** v3.x — descarga: <https://androidmtk.com/download-amlogic-usb-burning-tool>
- **Firmware Android 11 del X98Q** (mismo hardware: S905W2, 2 GB / 16 GB) — descarga: <https://androidpctv.com/firmware-x98q-amlogic/>
- Un **cable USB-A macho a macho** (USB-A a USB-A). No es el típico de carga; se consigue en casas de informática.
- Un **clip o palillo fino** para presionar el botón de reset dentro del puerto AV.
- En el dispositivo: la app **Termux** (para usar la terminal con root).

> El firmware del X98Q sirve porque comparte exactamente el SoC y la configuración de memoria del Panacom TV-1016. Otros firmwares de S905W2 (X98 Mini, etc.) pueden tener drivers de WiFi incompatibles.

### Verificar el chip (opcional)

Para confirmar que tu unidad realmente lleva el **Amlogic S905W2**, podés abrir el TV box:

1. Dale vuelta el equipo. Vas a ver **dos regatones (patas de goma) pegados** en la base; debajo de cada uno hay **un tornillo**. Despegá los regatones y sacá los dos tornillos.
2. Desatornillá y separá la placa de la carcasa.
3. El SoC es el chip más grande y tiene un **disipador** encima. **Despegalo con cuidado** para leer la leyenda del chip, que debe decir **Amlogic S905W2**.

> **Al volver a armar:** si la **pasta térmica** entre el chip y el disipador quedó seca o se desprendió, **limpiá los restos viejos** (del chip y del disipador) y **aplicá pasta térmica nueva** antes de recolocar el disipador. Esto mantiene la refrigeración correcta del SoC.

---

## 2. Flashear Android 11 (firmware X98Q)

### 2.1. Preparar la herramienta

1. Instalá el **Amlogic USB Burning Tool** (<https://androidmtk.com/download-amlogic-usb-burning-tool>) en Windows. Durante la instalación, aceptá instalar también los **drivers USB de Amlogic**.
2. Abrí la herramienta y cargá el firmware: **File → Import Image** y seleccioná el archivo `.img` del X98Q.

### 2.2. Configurar las opciones

- En **Erase Flash**, elegí **Normal Erase**.
- Tildá **Reset After Success**.
- Hacé clic en **Start**. La herramienta queda esperando que conectes el dispositivo.

### 2.3. Entrar en modo de grabación (burn mode)

El TV-1016 tiene el botón de reset **dentro del puerto AV** (el conector redondo de video).

1. Con el dispositivo **apagado y desconectado**, insertá el clip en el puerto AV y **mantené presionado** el botón interno.
2. Sin soltar, conectá el **cable USB-A a USB-A** entre el dispositivo y la PC.
3. Soltá el botón después de 3–5 segundos.
4. La herramienta detecta el dispositivo y comienza a grabar.

> Si no lo detecta, probá con el otro puerto USB del equipo.

### 2.4. Finalizar

- La grabación tarda entre 5 y 15 minutos. **No desconectes la alimentación durante el proceso.**
- Al terminar muestra **"Burn successful"** y el dispositivo se reinicia solo.
- El primer arranque puede tardar varios minutos. Es normal.

---

## 3. Primer arranque y WiFi

1. Conectá el HDMI al televisor (o al amplificador/receiver).
2. Completá la configuración inicial de Android.
3. Conectate al WiFi.

> **Nota sobre el WiFi:** si el WiFi no conecta o anda lento, suele ser el **cable de la antena** mal ubicado dentro de la carcasa. Al cerrar el equipo, verificá que el cable de antena no quede aplastado ni doblado bruscamente. Para medir la velocidad real, usá una herramienta confiable (por ejemplo `speedtest-cli` desde Termux); los tests del navegador en estos boxes subestiman la velocidad.

### Actualizar a versiones más nuevas (FOTA)

Este firmware incluye una **app de actualización FOTA preinstalada**, que permite actualizar a versiones más recientes del firmware directamente desde el dispositivo (por aire, sin volver a flashear con el USB Burning Tool). Conviene buscar actualizaciones desde esa app después del primer arranque.

> **Importante:** si vas a aplicar las configuraciones de root/control remoto de los pasos siguientes, hacé primero las actualizaciones FOTA y recién después instalá Magisk. Una actualización FOTA posterior puede sobrescribir el boot y desactivar Magisk (ver [Notas finales](#8-notas-finales)).

### Instalar las apps

Las apps que no estén en la tienda se instalan como **APK** (descargándolas desde el navegador del dispositivo o pasándolas por USB), por ejemplo Stremio y Tidal. Netflix se instala desde la tienda incluida.

---

## Opcional: ejecutar los comandos por SSH desde la PC

Los comandos de los pasos siguientes se ejecutan en **Termux**. Tipearlos en el TV box es incómodo, así que conviene levantar un servidor **SSH** en Termux y conectarse desde la PC (que esté en la misma red WiFi). Así se copian y pegan los comandos cómodamente desde la terminal de la PC.

### Pasos en Termux (en el dispositivo)

1. Instalar el servidor OpenSSH:

   ```sh
   pkg install openssh
   ```

2. Ponerle una contraseña al usuario de Termux (sin esto, el login por contraseña no funciona):

   ```sh
   passwd
   ```

   Ingresá y confirmá una contraseña.

3. Averiguar el **nombre de usuario** de Termux (lo vas a necesitar para conectarte):

   ```sh
   whoami
   ```

   Devuelve algo como `u0_a51`.

4. Averiguar la **IP del dispositivo** en la red:

   ```sh
   ifconfig wlan0
   ```

   Buscá la dirección en la línea `inet` (por ejemplo `192.168.0.42`). También podés verla en **Ajustes → WiFi → red conectada**.

5. Iniciar el servidor SSH:

   ```sh
   sshd
   ```

   > Termux escucha en el **puerto 8022** (no el 22 estándar). Hay que ejecutar `sshd` cada vez que reinicies, salvo que lo automatices aparte.

### Conectarse desde la PC

Con el usuario (paso 3) y la IP (paso 4), desde una terminal en la PC (Linux/macOS, o Windows con OpenSSH):

```sh
ssh u0_a51@192.168.0.42 -p 8022
```

Reemplazá `u0_a51` por tu usuario y `192.168.0.42` por la IP de tu dispositivo. Ingresá la contraseña que pusiste en el paso 2. Una vez dentro, podés ejecutar `su` y seguir con el resto de la guía desde la PC.

---

## 4. Decodificar los códigos del control remoto IR

El control remoto del Panacom usa un **custom code** que el firmware del X98Q no reconoce de fábrica. Hay que leer los códigos reales del control y armar una tabla.

### 4.1. Leer los códigos crudos

1. Abrí **Termux** en el dispositivo.
2. Apuntá el control al equipo y ejecutá, apretando cada tecla justo antes:

   ```sh
   dmesg | grep -i "meson-ir"
   ```

   Por cada tecla aparece una línea como:

   ```
   meson-ir fe084040.ir: invalid custom:0xec137f80
   ```

   (Dice "invalid" porque el código todavía no está configurado; eso es lo que queremos en esta etapa.)

### 4.2. Interpretar el código (little-endian)

El valor `0xAABBCCDD` se lee **invirtiendo los bytes**:

- **custom_code** = los **últimos 4 dígitos** del valor (los mismos para todas las teclas).
- **scancode** = el **segundo byte** (dígitos 3.º y 4.º desde la izquierda), distinto por tecla.

Ejemplo con la tecla OK = `0xec137f80`:

- custom_code = `0x7f80`
- scancode = `0x13`

### 4.3. Tabla del control Panacom

Estos son los valores leídos para este control (los tuyos pueden variar si tenés otra unidad — repetí el paso 4.1 para cada botón):

| Botón  | Valor crudo  | scancode | keycode (Linux) |
|--------|--------------|----------|------------------|
| Power  | 0x7e81**7f80** | 0x81   | 116 (POWER)      |
| Menu   | 0x7c83**7f80** | 0x83   | 125 (MENU)       |
| Home   | 0x8c73**7f80** | 0x73   | 102 (HOME)       |
| OK     | 0xec13**7f80** | 0x13   | 28 (ENTER/OK)    |
| Up     | 0xc738**7f80** | 0x38   | 103 (UP)         |
| Down   | 0xbf40**7f80** | 0x40   | 108 (DOWN)       |
| Left   | 0xc837**7f80** | 0x37   | 105 (LEFT)       |
| Right  | 0xc639**7f80** | 0x39   | 106 (RIGHT)      |
| Back   | 0xd827**7f80** | 0x27   | 158 (BACK)       |
| Vol−   | 0x7689**7f80** | 0x89   | 114 (VOL DOWN)   |
| Vol+   | 0x7887**7f80** | 0x87   | 115 (VOL UP)     |
| Mouse  | 0xb748**7f80** | 0x48   | 133 (INFO)       |

**custom_code del control: `0x7f80`**

> Los keycodes Linux se eligen de modo que el archivo de keylayout (`.kl`) del dispositivo los traduzca a teclas válidas de Android. El botón Mouse se mapea a **133 (INFO)** porque ese keycode sí está mapeado en el `.kl` y sirve de activador para la app de mouse por software (ver paso 7).

---

## 5. Instalar Magisk (root)

Se usa Magisk para cargar la tabla del control en cada arranque de forma "systemless" (sin modificar la partición de sistema real).

> Estos pasos asumen que el firmware provee acceso root (`su`). Se usa **Termux** para los comandos.

### 5.1. Instalar la app Magisk

1. Descargá el APK oficial desde **github.com/topjohnwu/Magisk/releases**.
2. Instalalo en el dispositivo (permitiendo "instalar apps de origen desconocido").

### 5.2. Volcar la imagen de boot

En Termux:

```sh
su
SLOT=$(getprop ro.boot.slot_suffix)
dd if=/dev/block/by-name/boot$SLOT of=/sdcard/boot.img
```

> **Guardá una copia de `/sdcard/boot.img` en la PC.** Es tu respaldo para revertir Magisk con un `dd` si hiciera falta.

### 5.3. Parchear el boot por línea de comandos

(En este hardware el parcheo desde la app puede dar "Process error"; el método por CLI usa los mismos binarios de la app y es confiable.)

```sh
su
APK=$(ls -d /data/app/*/com.topjohnwu.magisk*/ | head -1)
LIB=$(ls -d ${APK}lib/* | head -1)   # carpeta de arquitectura (arm o arm64)

mkdir -p /data/local/tmp/mp && cd /data/local/tmp/mp
cp /sdcard/boot.img .

cp "$LIB/libmagiskboot.so"   ./magiskboot
cp "$LIB/libmagiskinit.so"   ./magiskinit
cp "$LIB/libmagiskpolicy.so" ./magiskpolicy
cp "$LIB/libbusybox.so"      ./busybox
cp "$LIB/libinit-ld.so"      ./init-ld   2>/dev/null
cp "$LIB/libmagisk.so"       ./magisk32  2>/dev/null
cp "$LIB/libmagisk.so"       ./magisk    2>/dev/null
chmod 755 ./*

./busybox unzip -o "${APK}base.apk" "assets/*" -d .
cp assets/boot_patch.sh assets/util_functions.sh assets/stub.apk .
chmod 755 boot_patch.sh

export KEEPVERITY=true KEEPFORCEENCRYPT=true RECOVERYMODE=false
sh boot_patch.sh boot.img
```

Al terminar, genera **`new-boot.img`** en `/data/local/tmp/mp`.

### 5.4. Escribir el boot parcheado

```sh
su
SLOT=$(getprop ro.boot.slot_suffix)
dd if=/data/local/tmp/mp/new-boot.img of=/dev/block/by-name/boot$SLOT
sync
reboot
```

### 5.5. Completar la instalación (Direct Install)

Tras reiniciar, el daemon de Magisk corre (`su` funciona), pero falta completar la integración con el init para que se ejecuten los scripts de arranque:

1. Abrí la app **Magisk**.
2. Tocá **Install → Direct Install (Recommended)**.
   - **No** elijas "Install to inactive slot".
3. Esperá a "All done!" y **reiniciá**.

Verificá que Magisk quedó activo:

```sh
su
magisk -c        # debe imprimir la versión, p. ej. 30.7:MAGISK:R
```

---

## 6. Cargar el control remoto de forma persistente

El sistema carga las tablas IR con el binario `/vendor/bin/remotecfg`. Vamos a crear un módulo de Magisk con nuestra tabla y un script que la cargue en cada arranque.

### 6.1. Crear el módulo y el archivo de tabla

```sh
su
MOD=/data/adb/modules/panacom_remote
mkdir -p $MOD/system/vendor/etc

cat > $MOD/module.prop <<'EOF'
id=panacom_remote
name=Panacom IR Remote
version=1.0
versionCode=1
author=user
description=Carga el control Panacom (custom_code 0x7f80)
EOF

cat > $MOD/system/vendor/etc/remote.tab3 <<'EOF'
custom_name = amlogic-remote-3
custom_code = 0x7f80
release_delay = 80
fn_key_scancode = 0x00
cursor_left_scancode = 0x37
cursor_right_scancode = 0x39
cursor_up_scancode = 0x38
cursor_down_scancode = 0x40
cursor_ok_scancode = 0x13
key_begin
0x81 116
0x83 125
0x73 102
0x13 28
0x38 103
0x40 108
0x37 105
0x39 106
0x27 158
0x89 114
0x87 115
0x48 133
key_end
EOF

chmod 644 $MOD/module.prop $MOD/system/vendor/etc/remote.tab3
```

### 6.2. Crear el script de carga en el arranque

```sh
su
cat > /data/adb/service.d/panacom_remote.sh <<'EOF'
#!/system/bin/sh
until [ "$(getprop sys.boot_completed)" = "1" ]; do sleep 2; done
sleep 5
/vendor/bin/remotecfg -t /data/adb/modules/panacom_remote/system/vendor/etc/remote.tab3
EOF

chmod 755 /data/adb/service.d/panacom_remote.sh
```

### 6.3. Probar

Cargá la tabla en caliente (o reiniciá, que la carga sola):

```sh
su
/vendor/bin/remotecfg -t /data/adb/modules/panacom_remote/system/vendor/etc/remote.tab3
cat /sys/class/remote/amremote/map_tables
```

Deberías ver el `0x7f80` en la lista. Probá todas las teclas del control. Después reiniciá una vez para confirmar que el control responde solo, sin ejecutar ningún comando.

---

## 7. Modo mouse por software (opcional)

El kernel de este firmware no tiene el modo mouse por hardware del control, pero se puede lograr el mismo comportamiento (puntero movido con las flechas) mediante una app de accesibilidad. El botón **Mouse** ya quedó mapeado al keycode **INFO (165 en Android)** en el paso 6, listo para usarse como activador.

1. Descargá **MATVT** (Mouse for Android TV Toggle) desde <https://github.com/virresh/matvt/releases>.
   - Usá una **pre-release reciente**; las versiones estables viejas tienen un bug que impide el clic en algunos dispositivos.
2. Instalala y habilitala en **Ajustes → Accesibilidad**, otorgando todos los permisos (incluido **"Mostrar sobre otras apps"**).
3. En la configuración de MATVT, usá **"Automatically detect boss key code"** y apretá el botón **Mouse** del control. Debe detectar **Key 165** (INFO).
4. Probá: el botón **Mouse** activa/desactiva el cursor, las flechas lo mueven y **OK** clickea.

> Tené un mouse USB a mano durante la configuración por si el servicio de accesibilidad queda mal habilitado.

---

## 8. Notas finales

- **Respaldo:** guardá el `boot.img` original (paso 5.2) en un lugar seguro. Permite revertir Magisk con `dd` sin recurrir al USB Burning Tool.
- **Actualizaciones OTA:** una OTA puede sobrescribir el boot y perder Magisk (y con él, la carga automática del control). El módulo y el script en `/data` sobreviven; solo hay que rehacer el **Direct Install** de Magisk (paso 5.5) para reactivar todo.
- **Limpieza:** podés borrar los temporales cuando termines:

  ```sh
  su
  rm -rf /data/local/tmp/mp
  rm -f /sdcard/boot.img
  ```

- **Personalizar teclas:** para cambiar el mapeo de algún botón, editá el archivo
  `/data/adb/modules/panacom_remote/system/vendor/etc/remote.tab3`
  y reiniciá (o recargá con `remotecfg`).

---

*Valores como el `custom_code` (`0x7f80`) y los scancodes son específicos de este control remoto. Si tu unidad tiene otro control, repetí el paso 4 para obtener los tuyos.*
