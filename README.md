# MODELADO Y SIMULACION DE ROBOTS - PRACTICA 3

### 3º Ingenieria Robotica Software - 2025 / Irene Diez de toro :)

# PARTE A:

El objetivo de esta parte consistió en configurar el modelo del robot utilizado en la Práctica 2 para que pudiera ser visualizado, operado y controlado utilizando el software disponible en ROS 2.

## Estructura del paquete

Para ello, lo primero fue crear un paquete de ROS 2 donde almacenar los archivos del robot junto con los archivos específicos necesarios para ROS 2. En mi caso, el paquete se llama `lykos_description`, siguiendo el nombre de mi robot: **Lykos**.

Una vez creado el paquete, creé varias carpetas siguiendo la estructura indicada en el enunciado. Posteriormente, procedí a implementar el robot utilizando archivos XACRO. A continuación, se describe el contenido de cada carpeta:

### Carpeta `urdf/`

- **base/**: contiene el archivo `lykos_base.urdf.xacro`, que define la estructura del chasis del robot, incluyendo todos los *links* y *joints* que lo conforman. También se incluye el `base_footprint`, tal como se especifica en el enunciado.
  
- **scara_arm/**: incluye tres archivos:
  - `arm.urdf.xacro`: macro del brazo SCARA.
  - `gripper.urdf.xacro`: macro de la pinza.
  - `fingers.urdf.xacro`: macro de los dedos de la pinza.
  
- **sensors/**: contiene las macros de los sensores:
  - `camera.urdf.xacro`
  - `imu.urdf.xacro`
  - `gps.urdf.xacro`

- **utils/**: define materiales y colores utilizados en los distintos *links* mediante el archivo `utils.urdf.xacro`.

- **wheels/**: contiene la estructura de las ruedas:
  - `left_wheels.urdf.xacro`
  - `right_wheels.urdf.xacro`

### Carpeta `robots/`

- **lykos.urdf.xacro**: macro principal del robot, la cual incluye y ensambla todas las macros anteriores para formar el modelo completo de Lykos.


## PARTE B:
