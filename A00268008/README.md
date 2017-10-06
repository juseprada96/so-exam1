### Examen 1
**Universidad ICESI**  
**Curso:** Sistemas Operativos  
**Estudiante:** Juan Sebastián Prada.  
**Tema:** Comandos de Linux, Virtualización  
**Correo:** juan.prada1 at correo.icesi.edu.co

### Retos CMD

#### Reto suma

![][1]

![][2]

#### Reto reemplazar espacios en archivos

![][3]

![][4]

#### Reto lectura inversa

![][5]

![][6]

#### Reto remover lineas duplicadas

![][7]

![][8]

#### Reto mostrar tabla

![][9]

![][10]

### Script gunteberg

#### Script gutenberg

![][14]

#### Demostración de existencia usuario gutenberg

![][11]

#### Asignación del script gutenberg al crontab

![][12]

#### Demostración funcionamiento del script junto con el crontab

![][13]


### Descripción código fuente rickroll.c

```
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/syscalls.h>
#include <linux/string.h>
```
Definición de los includes necesarios para el modulo

```
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Kamal Marhubi");
MODULE_DESCRIPTION("Rickroll module");
```
Definición de los parametros del modulo
```
static char *rickroll_filename = "/home/bork/media/music/Rick Astley - Never Gonna Give You Up.mp3";
module_param(rickroll_filename, charp, S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH);
MODULE_PARM_DESC(rickroll_filename, "The location of the rick roll file");
```
Definición de la ubicación del archivo rickroll.mp3 en una variable tipo char
Definición del parametro de modulo con la variable de ubicación y paso de permisos de escritura y lectura
Definición de la descripción del parametro de modulo
```
#define DISABLE_WRITE_PROTECTION (write_cr0(read_cr0() & (~ 0x10000)))
#define ENABLE_WRITE_PROTECTION (write_cr0(read_cr0() | 0x10000))
```
Definición de constantes de desactivación de protección de memoria y activación de protección de memoria
```
static unsigned long **find_sys_call_table(void);
asmlinkage long rickroll_open(const char __user *filename, int flags, umode_t mode);
asmlinkage long (*original_sys_open)(const char __user *, int, umode_t);
asmlinkage unsigned long **sys_call_table;
```
Definición de variables de posición de la tabla de sys_calls, el rickroll_open, la posición original de la tabla de syscalls


```
static int __init rickroll_init(void)
{
    if(!rickroll_filename) {
	printk(KERN_ERR "No rick roll filename given.");
	return -EINVAL;  /* invalid argument */
    }

    sys_call_table = find_sys_call_table();

    if(!sys_call_table) {
	printk(KERN_ERR "Couldn't find sys_call_table.\n");
	return -EPERM;  /* operation not permitted; couldn't find general error */
    }
    DISABLE_WRITE_PROTECTION;
    original_sys_open = (void *) sys_call_table[__NR_open];
    sys_call_table[__NR_open] = (unsigned long *) rickroll_open;
    ENABLE_WRITE_PROTECTION;

    printk(KERN_INFO "Never gonna give you up!\n");
    return 0;  /* zero indicates success */
}

```
Se valida si los parametros de rickroll_file y la posición de la tabla de sys_calls son validos
de lo contrario lanza un error.

Si los parametros son validos se procede a desactivar la protección de escritura y se modifica el
sys_call open para que sea atentido por rickroll_open, una vez realizado se reactiva la protección
de escritura

retorna 0 indicando exito

```

asmlinkage long rickroll_open(const char __user *filename, int flags, umode_t mode)
{
    int len = strlen(filename);
    if(strcmp(filename + len - 4, ".mp3")) {
	return (*original_sys_open)(filename, flags, mode);
    } else {
	mm_segment_t old_fs;
	long fd;
	old_fs = get_fs();
	set_fs(KERNEL_DS);
	fd = (*original_sys_open)(rickroll_filename, flags, mode);
	set_fs(old_fs);
	return fd;
    }
}

```
El rickroll_open se encarga de filtrar los llamados open y reemplazar los que tengan tipo de archivo .mp3
por el archivo rickroll.mp3 y dejar pasar los demas open.
Es decir si se pretende abrir cualquier archivo .mp3 sonará la canción rickroll

```
static void __exit rickroll_cleanup(void)
{
    printk(KERN_INFO "Ok, now we're gonna give you up. Sorry.\n");

    /* Restore the original sys_open in the table */
    DISABLE_WRITE_PROTECTION;
    sys_call_table[__NR_open] = (unsigned long *) original_sys_open;
    ENABLE_WRITE_PROTECTION;
}

```
Se encarga de limpiar los efectos del modulo rickroll
```
static unsigned long **find_sys_call_table() {
    unsigned long offset;
    unsigned long **sct;
    for(offset = PAGE_OFFSET; offset < ULLONG_MAX; offset += sizeof(void *)) {
	sct = (unsigned long **) offset;

	if(sct[__NR_close] == (unsigned long *) sys_close)
	    return sct;
    }
    return NULL;
}

```
Encuentra la posición de la sys_call table
```
module_init(rickroll_init);
module_exit(rickroll_cleanup);
```
Definición del init y exit del modulo


[1]:imagenes/Screenshot_1.png
[2]:imagenes/Screenshot_2.png
[3]:imagenes/Screenshot_3.png
[4]:imagenes/Screenshot_4.png
[5]:imagenes/Screenshot_5.png
[6]:imagenes/Screenshot_6.png
[7]:imagenes/Screenshot_7.png
[8]:imagenes/Screenshot_8.png
[9]:imagenes/Screenshot_9.png
[10]:imagenes/Screenshot_10.png
[11]:imagenes/Screenshot_11.png
[12]:imagenes/Screenshot_13.png
[13]:imagenes/Screenshot_14.png
[14]:imagenes/Screenshot_16.png
