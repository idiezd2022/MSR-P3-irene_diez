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

![imagen](https://github.com/user-attachments/assets/07596ec5-88b0-4f5a-b54c-f64d4757b186)

### Lanzamiento y visualización en RViz

Una vez creado el modelo del robot, se procedemos a implementar el *launcher* y configurar RViz para su visualización. Primero, se crea un archivo de lanzamiento (`launch/lykos_state_publisher.launch.py`) que carga el modelo del robot y lanza RViz con la configuración deseada. Este archivo utiliza `robot_state_publisher` para publicar la descripción del robot e incluye un nodo adicional, `joint_state_publisher_gui`, para controlar manualmente las articulaciones. También se incluye un archivo de configuración para RViz (`rviz/robot.rviz`) donde se define la vista inicial, los *frames* a mostrar y los sensores del robot. Un trozo del launcher:

```bash
robot_state_publisher_node = Node(
        package='robot_state_publisher',
        executable='robot_state_publisher',
        name='robot_state_publisher',
        output='screen',
        parameters=[{
            'use_sim_time': use_sim_time,
            'robot_description': robot_description_param,
        }]
    )

    joint_state_publisher_gui_node = Node(
        package='joint_state_publisher_gui',
        executable='joint_state_publisher_gui',
        name='joint_state_publisher_gui',
        output='screen'
    )

    rviz_node = Node(
        package='rviz2',
        executable='rviz2',
        name='rviz2',
        output='screen',
        arguments=['-d', PathJoinSubstitution([FindPackageShare("lykos_description"), "rviz", "robot.rviz"])]
    )
```

Al lanzar este archivo con:

```bash
ros2 launch lykos_description robot_launch.py
```

Vemos que:
- La visualización de las tf se realiza correctamente:

![imagen](https://github.com/user-attachments/assets/87a6202c-5839-4f44-8a5a-c44def1a80a5)

- El modelo del robot se ve correctamente y además, el panel del control de los joints tambien funciona correctamente.

![imagen](https://github.com/user-attachments/assets/bbf4b5e8-816d-4d8f-9128-3695a9cb787a)

[Video del funcionamiento](https://github.com/user-attachments/assets/63d3e0dc-52f5-4471-a9f4-67fbcd923f40)

Para completar este primera parte de la practica, se nos pedia mostrar el árbol de transformadas entre los links del robot:

![imagen](/visual_things/frame.png)









## PARTE B:
