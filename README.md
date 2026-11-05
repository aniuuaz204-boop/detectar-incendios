# FireWatch: detector con alertas por correo

Prototipo educativo en Python que evalua lecturas de humo/gas tipo MQ-2 y
temperatura. Al cambiar a prealerta o alarma puede enviar un correo por SMTP.
No predice incendios con una cantidad garantizada de minutos, no está certificado
y no reemplaza detectores de humo certificados ni un sistema de alarma aprobado.

## Requisitos

- Python 3.10 o posterior.
- Para abrir la webcam, instala OpenCV:

```powershell
python -m pip install -r requirements.txt
```

La vista previa requiere una sesion de escritorio con ventanas; no funciona en
un terminal sin interfaz grafica.

## Comprobar la webcam

Desde esta carpeta, abre el indice 0 (normalmente la webcam integrada):

```powershell
python -m firewatch --camera 0
```

Tambien puedes ejecutar directamente `firewatch\__init__.py`: sin argumentos
abre la camara 0 con deteccion visual experimental; con `--help` muestra las
opciones.

Para cerrar, selecciona la ventana de video y pulsa `Q`. Si no abre, comprueba
los permisos de camara en la configuracion de privacidad de Windows, cierra
otras aplicaciones que la esten usando y prueba otro indice, por ejemplo
`--camera 1`. FireWatch intenta DirectShow, Media Foundation y el controlador
predeterminado; muestra un error si no logra abrir o leer la webcam.
Antes de mostrar el video comprueba que reciba fotogramas. Si persiste una
imagen practicamente negra, presenta un aviso para revisar la tapa fisica, la
iluminacion y los permisos de Windows; no modifica el brillo del video.

## Buscar indicios visuales experimentales

```powershell
python -m firewatch --camera 0 --detect
```

La heuristica resalta regiones brillantes rojo/naranja que parpadean como
posible llama y regiones grises que ascienden como posible humo. Ignora gran
parte del movimiento global de la camara y reduce cuadros grandes antes del
analisis. Al confirmar indicios muestra una prealerta en pantalla, la consola
y un sonido del sistema; estima tambien la frecuencia visual del parpadeo en
Hz cuando hay suficientes fotogramas. Es una medicion aproximada del cambio
de area de color entre cuadros, no una identificacion del comportamiento del
fuego. Pulsa `Q` para salir. Para mandar un
correo cuando aparece una prealerta, configura SMTP como arriba y agrega
`--email`:

```powershell
python -m firewatch --camera 0 --detect --email
```

La vision por color y movimiento es experimental: luces, reflejos, objetos
naranjas, sombras y movimiento de la camara pueden causar falsas alarmas; un
incendio real tambien puede no detectarse. No se debe usar para proteger vidas,
ni confiar en el correo como alarma. Usa detectores certificados y una alarma
local aprobada. El analisis de video no sube ni adjunta imagenes al correo.
Una prealerta significa solo que aparecio un patron visual compatible; no
garantiza detectar el inicio de un incendio ni dar anticipacion.

## Probar sin hardware

Desde esta carpeta:

```powershell
python -m firewatch --simulate smoke
```

Escenarios disponibles: `normal`, `smoke`, `rapid-temperature` y `emergency`.
La simulacion sigue ejecutandose hasta detenerla con `Ctrl+C`.

## Habilitar correo

Configura las variables de entorno SMTP de tu proveedor. No escribas la
contraseña en el código ni la compartas:

```powershell
$env:SMTP_HOST = "smtp.tu-proveedor.com"
$env:SMTP_PORT = "587"
$env:SMTP_SECURITY = "starttls"
$env:SMTP_USER = "tu-cuenta"
$env:SMTP_PASSWORD = "tu-contraseña-o-clave-de-aplicacion"
$env:ALERT_FROM = "tu-cuenta@ejemplo.com"
$env:ALERT_TO = "destinatario@ejemplo.com"
python -m firewatch --simulate smoke --email
```

Para SMTP sobre SSL directo, usa `SMTP_SECURITY=ssl` y normalmente el puerto
465. `ALERT_TO` admite varias direcciones separadas por coma. La opción `--email`
es necesaria para que se envíe cualquier mensaje; sin ella la simulación solo
avisa por consola. Los errores de correo se muestran en pantalla y el monitor
continúa funcionando.

## Sensor físico y seguridad

El modo de camara puede buscar indicios visuales experimentales, pero no confirma
fuego ni humo. El simulador todavía no controla un MQ-2, un ADC
ni una sirena física. Para conectar hardware se necesita implementar el
adaptador correspondiente a la placa (por ejemplo Raspberry Pi y ADC compatible),
verificar los niveles eléctricos y probarlo de manera segura. No conectes la
salida analógica del MQ-2 directamente a un GPIO. El correo depende de la red y
no debe ser el único medio de alarma; conserva una alarma local certificada e
independiente.

Los umbrales de ejemplo son valores orientativos, no límites homologados. El
MQ-2 puede responder a otros gases y necesita calibración; un correo puede
retrasarse o no llegar. En una emergencia, sigue los procedimientos locales y
contacta a los servicios de emergencia.

## Pruebas

```powershell
python -m unittest discover -s tests -v
```
