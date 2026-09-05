# Guía de Flasheo: LSI / Broadcom SAS 3008 a Modo IR (Supermicro)

Esta guía detalla los pasos para cambiar una controladora **LSI / Broadcom SAS 3008** de modo **IT (Initiator Target)** a modo **IR (Integrated RAID)** utilizando el paquete de firmware de Supermicro mediante el Shell UEFI.

> ⚠️ **¡ADVERTENCIA IMPORTANTE!**  
> **No apagues ni reinicies el servidor** entre el paso de borrado de la flash (`-e 6`) y la reescritura del firmware. De lo contrario, la controladora podría quedar en un estado inaccesible (*bricked*).

---

## 1. Archivos Requeridos

Asegúrate de tener los siguientes archivos en la raíz de tu unidad USB formateada en **FAT32**:

* `sas3flash.efi` — Herramienta de flasheo para UEFI.
* `3008IR16.ROM` — Firmware en modo IR (Integrated RAID).
* `mptsas3.rom` — ROM de arranque Legacy BIOS.
* `mpt3x64.rom` — ROM de arranque UEFI BSD.
* `SMC3008R.NSH` — Script de automatización de Supermicro (opcional).

---

## 2. Método 1: Ejecución Automática (Recomendado)

Desde la consola UEFI Shell en la raíz de tu USB (`fs0:\>`):

1. Ejecuta el script provisto por Supermicro:
   ```text
   fs0:\> SMC3008R.NSH
   ```
2. Sigue las instrucciones en pantalla si el script te solicita confirmar o ingresar la dirección SAS.

---

## 3. Método 2: Flasheo Manual Paso a Paso

Si el script automático falla o prefieres realizar el procedimiento manualmente, sigue este orden estricto:

### Paso 1: Anotar la dirección SAS actual
Ejecuta el siguiente comando para ver los detalles de la tarjeta y **anota la dirección SAS** (16 dígitos hexadecimales que comienzan por `500...`):

```text
fs0:\> sas3flash.efi -list
```

*(También puedes encontrar este número en la pegatina física ubicada al dorso de la tarjeta).*

---

### Paso 2: Borrar la memoria flash (Paso Crítico)
Elimina el firmware IT guardado en la memoria flash para permitir la instalación del firmware IR:

```text
fs0:\> sas3flash.efi -o -e 6
```

---

### Paso 3: Flashear el Firmware IR y los ROMs de arranque
Graba el archivo de firmware IR junto con las imágenes BIOS y UEFI:

```text
fs0:\> sas3flash.efi -f 3008IR16.ROM -b mptsas3.rom -b mpt3x64.rom
```

---

### Paso 4: Restaurar la dirección SAS
Vuelve a programar la dirección SAS anotada en el Paso 1:

```text
fs0:\> sas3flash.efi -o -sasadd 50060XXXXXXXXXXXX
```

*(Reemplaza `50060XXXXXXXXXXXX` por la dirección SAS real de tu tarjeta).*

---

### Paso 5: Verificación
Verifica que la tarjeta responda correctamente y reporte el firmware IR cargado:

```text
fs0:\> sas3flash.efi -list
```

Una vez verificado, puedes reiniciar el servidor e ingresar al menú de configuración RAID de la controladora durante el post del sistema.
