# 📱 Android SDK — Comandos útiles

> Sin Android Studio. Solo SDK + emulador desde terminal o VS Code.

---

## 🔧 Variables de entorno requeridas

```
ANDROID_HOME    = C:\Users\Dyno\AppData\Local\Android\Sdk
ANDROID_SDK_ROOT = C:\Users\Dyno\AppData\Local\Android\Sdk

PATH debe incluir:
  %ANDROID_HOME%\platform-tools
  %ANDROID_HOME%\emulator
```

---

## ✅ Verificar instalación

```bash
# Verificar ADB
adb version

# Listar emuladores disponibles
emulator -list-avds
```

---

## 🚀 Emulador

```bash
# Levantar un emulador
emulator -avd NOMBRE_DEL_EMULADOR

# Levantar sin ventana (headless)
emulator -avd NOMBRE_DEL_EMULADOR -no-window

# Levantar con GPU acelerada
emulator -avd NOMBRE_DEL_EMULADOR -gpu host
```

---

## 📡 ADB — Android Debug Bridge

```bash
# Ver dispositivos conectados (físicos + emuladores activos)
adb devices

# Instalar un APK en el dispositivo/emulador
adb install ruta\al\archivo.apk

# Desinstalar una app
adb uninstall com.paquete.app

# Abrir shell en el dispositivo
adb shell

# Copiar archivo del PC al dispositivo
adb push archivo.txt /sdcard/archivo.txt

# Copiar archivo del dispositivo al PC
adb pull /sdcard/archivo.txt archivo.txt

# Ver logs en tiempo real
adb logcat

# Filtrar logs por tag
adb logcat -s "MiApp"

# Reiniciar ADB si no detecta dispositivos
adb kill-server
adb start-server
```

---

## 📦 SDK Manager (sin Android Studio)

```bash
# Listar paquetes instalados
sdkmanager --list_installed

# Listar paquetes disponibles
sdkmanager --list

# Instalar un paquete
sdkmanager "platform-tools"
sdkmanager "platforms;android-34"
sdkmanager "system-images;android-34;google_apis;x86_64"

# Actualizar todo
sdkmanager --update
```

---

## 🖥️ AVD Manager (gestionar emuladores)

```bash
# Listar emuladores
avdmanager list avd

# Crear un emulador nuevo
avdmanager create avd -n MiEmulador -k "system-images;android-34;google_apis;x86_64"

# Eliminar un emulador
avdmanager delete avd -n MiEmulador
```

---

## 💡 Tips

- Los emuladores pesan mucho en `C:\Users\Dyno\.android\avd\` — borra los que no uses
- `adb devices` debe mostrar el emulador como `emulator-5554 device` cuando está corriendo
- Si el emulador no aparece en `adb devices`, reinicia el servidor ADB

---

*Actualizado: Marzo 2026*
