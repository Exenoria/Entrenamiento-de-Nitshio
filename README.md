# Entrenamiento-de-Nitshio
Cuando Colab te desconecta se borra todo lo que estaba en `/content`: tus archivos, los paquetes instalados y también los parches que aplicamos. Lo que sobrevive es lo que está en GitHub y en Drive. No sé qué es tu «guardado fijo», así que cubro las dos opciones posibles.

## Pasos para volver a dejarlo todo igual

**1. Activa la GPU.** Entorno de ejecución → Cambiar tipo de entorno → T4 GPU.

**2. Recupera el proyecto en `/content/Nitshio`.** Esa es la carpeta que el notebook busca. Elige una de estas dos formas:

- **Desde GitHub.** Ejecuta esta celda con tu repositorio:
```python
import subprocess
from pathlib import Path

REPO = "https://github.com/TU_USUARIO/TU_REPO.git"   # cámbialo
DEST = Path("/content/Nitshio")
if not DEST.exists():
    subprocess.run(["git", "clone", "--depth", "1", REPO, str(DEST)], check=True)
else:
    subprocess.run(["git", "-C", str(DEST), "pull"])
```
  Si el repo es **privado**, necesitas un token de acceso. Guárdalo en Colab Secrets (el icono de la llave, a la izquierda) y no lo pegues dentro de la celda, porque se quedaría guardado en el notebook.

- **Desde tu guardado fijo.** Si es un zip en Google Drive, móntalo con `drive.mount("/content/drive")`, cópialo a `/content` y descomprímelo en `/content/Nitshio`. Si es un zip en tu PC, súbelo desde el panel Archivos y la celda 1 del notebook lo descomprime sola.

**3. Abre el notebook y ejecuta desde la celda 1.** La celda 1 reconoce el proyecto ya presente y no extrae nada. La celda 2 va a instalar los paquetes y te pedirá reiniciar la sesión. Reinicia y ejecuta de nuevo las celdas 1 y 2: la segunda vez debe terminar en `Entorno OK`.

**4. Vuelve a aplicar los parches.** Ejecuta la celda de los cuatro arreglos (la que empieza con `aplicar("pretokenizar dataset"...`). Es obligatoria en cada sesión nueva, porque esos cambios solo vivían en la copia de Colab. Si no la ejecutas, vuelven los errores de pickle y de `BFloat16`.

**5. Sigue con las celdas 4 y 5.** Dataset y entrenamiento.

## Antes de que se desconecte otra vez
- **Copia el resultado a Drive apenas termine el entrenamiento.** Si se pierde la sesión sin guardarlo, hay que entrenar de nuevo. Agrega esta celda después de la 5:
```python
from google.colab import drive
import shutil
drive.mount("/content/drive")
shutil.copytree(OUTPUT_LORA, "/content/drive/MyDrive/Nitshio/nitshio-lora", dirs_exist_ok=True)
```
- **Haz lo mismo con el `.gguf`.** La celda 9 del notebook ya lo copia a Drive.
- **Guarda el script con parches.** Cuando el entrenamiento funcione, descarga `/content/Nitshio/training/finetune_qlora.py` desde el panel Archivos y súbelo a tu repo, o dáselo a OpenCode. Con los cambios dentro del repo, el paso 4 deja de ser necesario.

## Puntos débiles
- **Comprueba qué incluye tu repo.** Para entrenar hacen falta `config/config.yaml`, la carpeta `brain/` (con la persona), `training/` con los datasets y `requirements-train.txt`. Si tu `.gitignore` excluye alguno, la celda 1 o la 4 se detendrán con un mensaje de archivo faltante. No sé qué subiste a GitHub, así que no puedo asegurarlo.
- **La celda 3 del notebook es del script viejo.** Con el script nuevo puede decir «ya estaba» o mostrar un `AVISO`. Es normal y no causa problemas. La actualizaré cuando confirmes que el entrenamiento funciona.

Dime qué contiene tu guardado fijo (zip en Drive, carpeta, o el repo mismo) y te digo exactamente cuál de las dos rutas del paso 2 te conviene.
