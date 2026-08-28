# Cómo publicar el instalador web de SwitchProESP32

Esta carpeta ya tiene la página lista (`index.html` + `manifest.json`).
Solo falta generar el binario del firmware y subir todo a GitHub Pages.

## 1. Generar el binario "merged" (un solo .bin)

Cuando compilas con Arduino IDE / PlatformIO / ESP-IDF, el firmware
queda partido en varios archivos (bootloader, tabla de particiones,
tu app). ESP Web Tools necesita **un solo .bin** con todo junto,
en la dirección 0x0.

Con PlatformIO, después de compilar (`pio run`), los archivos quedan en
`.pio/build/<entorno>/`. Corre esto (ajusta rutas y tamaño de flash a
lo que uses):

```bash
& "$env:USERPROFILE\.platformio\penv\Scripts\python.exe" "$env:USERPROFILE\.platformio\packages\tool-esptoolpy\esptool.py" --chip esp32 merge_bin `
  -o switchpro-navy-merged.bin `
  --flash_mode dio `
  --flash_freq 40m `
  --flash_size 4MB `
  0x1000 .pio/build/esp32dev/bootloader.bin `
  0x8000 .pio/build/esp32dev/partitions.bin `
  0x10000 .pio/build/esp32dev/firmware.bin

```

Si usas Arduino IDE, los mismos tres archivos están en la carpeta que
aparece en "Exportar binario compilado" (Sketch > Exportar binario
compilado), con nombres parecidos a `*.ino.bootloader.bin`,
`*.ino.partitions.bin` y `*.ino.bin`.

## 2. Acomodar los archivos

```
webflasher/
├── index.html
├── manifest.json
└── firmware/
    └── switchpro-navy-merged.bin   <- el que acabas de generar
```

## 3. Subir a GitHub Pages (gratis)

1. Crea un repositorio nuevo en GitHub (puede ser público o privado
   si tienes GitHub Pro/Team; Pages gratis requiere público).
2. Sube el contenido de esta carpeta a la raíz del repo (o a una
   carpeta `docs/`, como prefieras).
3. Ve a **Settings > Pages**, elige la rama y carpeta donde quedó
   `index.html`, guarda.
4. En un par de minutos tu página queda en
   `https://tu-usuario.github.io/tu-repo/`.

## Importante: por qué esto NO expone tu código fuente

El `.bin` es código máquina ya compilado para el ESP32 — no hay forma
práctica de recuperar tu `.cpp`/`.ino` original a partir de él. Lo
único que subes al repo público es ese binario y esta página; tu
código sigue viviendo donde tú decidas (privado, en tu máquina, etc.).

## Actualizar una nueva versión

Cuando saques una nueva versión del firmware, repite el paso 1,
reemplaza `firmware/switchpro-navy-merged.bin` y sube el cambio a GitHub.
También puedes subir el número de `"version"` en `manifest.json` para
que quede registrado cuál es cuál.
