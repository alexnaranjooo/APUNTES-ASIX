---
description: >-
  Práctica de copias de seguridad con rsync y tar.gz: en una misma máquina y
  entre dos, copias incrementales y diferenciales, y cuándo usar cada una.
---

# rsync + tar.gz

## Escenario de la práctica

Se copia una **carpeta de contenidos** y se restaura después. Los nombres son de ejemplo; hay que sustituirlos por los de cada máquina.

| Elemento                                         | Valor de ejemplo                             |
| ------------------------------------------------ | -------------------------------------------- |
| Carpeta a copiar                                 | `/srv/contenidos`                            |
| Carpeta de copias                                | `/backup`                                    |
| Máquina de origen y máquina de destino (parte 2) | `origen` y `destino`, con la IP `IP_DESTINO` |
| Usuario en el destino                            | `usuario`                                    |

Para tener datos de prueba:

```bash
mkdir -p /srv/contenidos/imagenes /backup
echo "documento 1" > /srv/contenidos/doc1.txt
echo "documento 2" > /srv/contenidos/doc2.txt
```

***

## 12. Creación y restauración de las copias

### 12.1 En una misma máquina virtual

**Con tar.gz** (un solo archivo comprimido y con la fecha en el nombre):

```bash
tar -czvf /backup/contenidos_$(date +%F).tar.gz -C /srv contenidos
tar -tzvf /backup/contenidos_$(date +%F).tar.gz
```

El segundo comando solo **lista** el contenido, para comprobar que la copia es buena. Para restaurar, primero se simula el desastre y después se extrae:

```bash
rm -rf /srv/contenidos
tar -xzvf /backup/contenidos_2026-11-05.tar.gz -C /srv
```

**Con rsync** (una copia de la carpeta tal cual, sin comprimir):

```bash
rsync -av /srv/contenidos/ /backup/contenidos/
```

Para restaurar se copia en sentido contrario:

```bash
rsync -av /backup/contenidos/ /srv/contenidos/
```

{% hint style="warning" %}
La barra final de `/srv/contenidos/` importa: con ella se copia **el contenido** de la carpeta; sin ella se copia la propia carpeta dentro del destino.
{% endhint %}

### 12.2 En dos máquinas: una de origen y otra de destino

Para que la copia no pida contraseña (y se pueda automatizar), se usa **SSH con clave**. En la máquina de origen:

```bash
ssh-keygen -t ed25519
ssh-copy-id usuario@IP_DESTINO
ssh usuario@IP_DESTINO
```

El último comando comprueba que se entra sin contraseña. Después:

**Con rsync**, la copia viaja por SSH y comprimida:

```bash
rsync -avz -e ssh /srv/contenidos/ usuario@IP_DESTINO:/backup/contenidos/
```

**Con tar.gz**, el archivo se crea y se envía de una vez, sin dejarlo en el origen:

```bash
tar -czf - -C /srv contenidos | ssh usuario@IP_DESTINO "cat > /backup/contenidos_$(date +%F).tar.gz"
```

**Restaurar** desde la máquina de origen, tras perder la carpeta:

```bash
rsync -avz -e ssh usuario@IP_DESTINO:/backup/contenidos/ /srv/contenidos/

ssh usuario@IP_DESTINO "cat /backup/contenidos_2026-11-05.tar.gz" | tar -xzf - -C /srv
```

Para repetirlo cada noche, se añade el comando con `crontab -e`, por ejemplo a las 03:00:

&#x20;`0 3 * * * comando`.

***

## 13. ¿rsync o tar.gz para un script de copias?

|                          | rsync                                                  | tar.gz                                                              |
| ------------------------ | ------------------------------------------------------ | ------------------------------------------------------------------- |
| Qué guarda               | Una réplica de los archivos                            | Un solo archivo comprimido                                          |
| Copias repetidas         | Solo envía lo que ha cambiado: muy rápido              | Copia todo cada vez (salvo con modo incremental)                    |
| Espacio                  | Sin comprimir, salvo en el viaje                       | Comprimido: ocupa menos                                             |
| Restaurar                | Copiar de vuelta, y se puede restaurar un solo archivo | Descomprimir; para un archivo suelto hay que buscarlo en el paquete |
| Historial de versiones   | No, por defecto solo la última                         | Sí, un archivo por fecha                                            |
| Cifrar, mover o archivar | Menos cómodo                                           | Muy cómodo: es un archivo único                                     |
| Remoto                   | Nativo con SSH                                         | Posible con una tubería a SSH                                       |

**¿Cuál es la mejor opción?** Depende de qué se necesite, pero para un script de copias de una carpeta de contenidos, mi elección es **rsync para la copia diaria** y **tar.gz para guardar un archivo fechado de vez en cuando**:

* El **rsync** es rápido porque no repite lo que no ha cambiado, y deja los archivos listos para usar.
* El **tar.gz** da el historial: si algo se estropeó hace tres semanas, hay un archivo de esa fecha. Además se cifra y se traslada fácilmente.

Si hubiera que elegir solo uno, **rsync**, ya que puede guardar versiones con la opción `--link-dest`:

```bash
HOY=$(date +%F)
rsync -a --delete --link-dest=/backup/ultima /srv/contenidos/ /backup/$HOY/
ln -sfn /backup/$HOY /backup/ultima
```

Cada carpeta fechada parece una copia completa, pero los archivos que no han cambiado son enlaces duros y no ocupan espacio extra.

Un script que combina ambos:

```bash
#!/bin/bash
ORIGEN="/srv/contenidos"
DESTINO="usuario@IP_DESTINO"
FECHA=$(date +%F)

# Cada noche: sincronizar con rsync
rsync -az --delete -e ssh "$ORIGEN/" "$DESTINO:/backup/contenidos/"

# Los domingos: archivo fechado con tar.gz
if [ "$(date +%u)" -eq 7 ]; then
  tar -czf "/tmp/contenidos-$FECHA.tar.gz" -C /srv contenidos
  scp "/tmp/contenidos-$FECHA.tar.gz" "$DESTINO:/backup/semanal/"
  rm "/tmp/contenidos-$FECHA.tar.gz"
fi
```

***

## RSYNC

### 1) ¿Qué es rsync? ¿En qué contexto se usa?

**rsync** es una herramienta que **sincroniza archivos y carpetas**, en local o entre equipos, enviando solo **las diferencias** (con un algoritmo de transferencia delta) en lugar de copiar todo. Conserva permisos, propietarios y fechas, y puede viajar cifrada por SSH.

Se usa para **copias de seguridad**, para mantener un **espejo** de un servidor web, para **migrar** datos entre servidores y para **sincronizar** equipos.

| Opción               | Qué hace                                                           |
| -------------------- | ------------------------------------------------------------------ |
| `-a`                 | Modo archivo: recursivo y conserva permisos, propietarios y fechas |
| `-v`                 | Muestra lo que va haciendo                                         |
| `-z`                 | Comprime durante el envío                                          |
| `-e ssh`             | Usa SSH como transporte                                            |
| `--delete`           | Borra en el destino lo que ya no existe en el origen               |
| `-n` (o `--dry-run`) | Simula, sin copiar nada                                            |
| `--progress`         | Muestra el progreso                                                |
| `--exclude`          | Deja fuera archivos o carpetas                                     |
| `--bwlimit`          | Limita el ancho de banda                                           |

### 2) Ejemplo de uso entre dos máquinas

```bash
rsync -avz --delete -e ssh /srv/contenidos/ usuario@IP_DESTINO:/backup/contenidos/
```

Copia la carpeta de la máquina de origen a la de destino, por SSH y comprimida, y deja el destino igual que el origen. Antes de usar `--delete` por primera vez conviene probarlo con `-n`.

***

## TAR.GZ

### 3) ¿Qué es tar.gz?

**tar** (_tape archive_) **empaqueta** varios archivos y carpetas en un solo archivo (`.tar`), conservando permisos y estructura. **gzip** lo **comprime** (`.gz`). Juntos dan un `.tar.gz`: un paquete único, comprimido y fácil de guardar o mover. No cifra por sí mismo.

| Opción | Qué hace                           |
| ------ | ---------------------------------- |
| `c`    | Crear el archivo                   |
| `x`    | Extraer                            |
| `t`    | Listar el contenido                |
| `v`    | Mostrar lo que hace                |
| `z`    | Comprimir o descomprimir con gzip  |
| `f`    | Indicar el nombre del archivo      |
| `-C`   | Cambiar de carpeta antes de actuar |

### 4) Copias incrementales y diferenciales con tar

GNU tar lo hace con un **archivo de estado** (`.snar`), que recuerda qué se copió y cuándo. Se usa la opción `--listed-incremental`.

**Copia completa** (si el archivo de estado no existe, copia todo y lo crea):

```bash
tar --listed-incremental=/backup/inc.snar -czf /backup/inc-completa-$(date +%F).tar.gz -C /srv contenidos
```

**Incrementales**: se repite el mismo comando con el **mismo** archivo de estado. Cada ejecución guarda solo lo cambiado desde la anterior y actualiza el estado:

```bash
echo "documento 3" > /srv/contenidos/doc3.txt
tar --listed-incremental=/backup/inc.snar -czf /backup/inc-$(date +%F).tar.gz -C /srv contenidos
tar -tzvf /backup/inc-$(date +%F).tar.gz
```

El listado debe mostrar solo `doc3.txt` (y las carpetas), no todo.

**Diferenciales**: se guarda una copia del estado de la copia completa y se usa **esa copia** cada vez, así cada diferencial recoge todo lo cambiado desde la completa:

```bash
tar --listed-incremental=/backup/dif.snar -czf /backup/dif-completa-$(date +%F).tar.gz -C /srv contenidos
cp /backup/dif.snar /backup/dif-base.snar

cp /backup/dif-base.snar /backup/dif-hoy.snar
tar --listed-incremental=/backup/dif-hoy.snar -czf /backup/dif-$(date +%F).tar.gz -C /srv contenidos
```

**Restaurar**: se extrae la completa y después las demás, **en orden**, con el archivo de estado nulo para que tar aplique también los borrados:

```bash
tar --listed-incremental=/dev/null -xzf /backup/inc-completa-2026-11-02.tar.gz -C /srv/restauracion
tar --listed-incremental=/dev/null -xzf /backup/inc-2026-11-03.tar.gz -C /srv/restauracion
tar --listed-incremental=/dev/null -xzf /backup/inc-2026-11-04.tar.gz -C /srv/restauracion
```

Con incrementales hacen falta **todas** en orden; con diferenciales, la completa y **solo la última** diferencial.

| Semana                 | Incremental                  | Diferencial                             |
| ---------------------- | ---------------------------- | --------------------------------------- |
| Lunes                  | Completa                     | Completa                                |
| Martes                 | Lo del martes                | Lo del martes                           |
| Miércoles              | Lo del miércoles             | Lo del martes y el miércoles            |
| Restaurar el miércoles | Completa, martes y miércoles | Completa y la diferencial del miércoles |
