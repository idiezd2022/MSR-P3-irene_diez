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

```python
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

![imagen](https://github.com/idiezd2022/MSR-P3-irene_diez/blob/main/visual_things/frames.png)


# PARTE B:

En esta segunda parte, el objetivo principal era conseguir un análisis de coste del mecanismo pick and place (equivalente a la Fase 3 de la práctica 2) en el simulador Gazebo mediante el framework de MoveIt2. Para ello, partimos de la base de la primera parte de la practica, con el robot ya implementado en ros2.

Para esta fase, se debían de realizar cambios muy relevantes en cada uno de los archivos.

- `Xacros de las ruedas`: Para poder lanzarlas correctamente en Gazebo y permitir su control, fue necesario añadir las siguientes líneas de código:

```xacro
    <transmission name="${prefix}_Wheel_${lado de la rueda}_joint_trans">
      <type>transmission_interface/SimpleTransmisson</type>
      <joint name="${prefix}_Wheel_R_joint">
        <hardwareInterface>hardware_interface/VelocityJointInterface</hardwareInterface>
      </joint>
      <actuator name="${prefix}_Wheel_${lado de la rueda}_joint_motor">
        <mechanicalReduction>1</mechanicalReduction>
      </actuator>
    </transmission>

    <gazebo reference="${prefix}_Wheel_${lado de la rueda}_link">
      <mu1>5</mu1>
      <mu2>5</mu2>
    </gazebo>

```
- `Xacro de los sensores`: En la parte anterior, estos se implementaron simplemente como un elemento decorativo más, sin ningun tipo de funcionamiento. Esa era la base para en esta parte poder implementar su verdadera función. Para ello, en cada xacro se añadió las funciones correspondientes que permitían su funcionalidad.
  - `camera.urdf.xacro`: En este caso implemntarmos una función generica para poder llamar cuantas cámaras queramos.
```xacro
<gazebo reference="${frame_prefix}_camera_frame">
    <sensor name="${frame_prefix}_sensor" type="camera">
      <visualize>true</visualize>
      <update_rate>30</update_rate>
      <topic>${frame_prefix}/image</topic> 
      <camera>
        <horizontal_fov>${radians(float(20))}</horizontal_fov>
        <image>
          <width>1280</width>
          <height>720</height>
          <format>R8G8B8</format>
        </image>
        <clip>
          <near>0.10</near>
          <far>15.0</far>
        </clip>
        <noise>
          <type>gaussian</type>
          <mean>0.0</mean>
          <stddev>0.007</stddev>
        </noise>
        <optical_frame_id>${frame_prefix}_camera_frame</optical_frame_id>
      </camera>
    </sensor>
  </gazebo>
</xacro:macro>
```
  - `imu.urdf.xacro`: Añadimos el pluggin de gazebo correspondiente.
```xacro
<gazebo reference="gps_base_link">
    <sensor name="gps_sensor" type="navsat">
      <always_on>1</always_on>
      <update_rate>5.0</update_rate>
      <topic>/robot/gps/fix</topic>
      <gz_frame_id>gps_base_link</gz_frame_id>
    </sensor>
  </gazebo>
```
  - `gps.urdf.xacro`:Añadimos el pluggin de gazebo correspondiente.
```xacro
<gazebo reference="imu_link">
    <sensor name="imu_sensor" type="imu">
      <always_on>1</always_on>
      <update_rate>30</update_rate>
      <topic>'imu/data'</topic>
    </sensor>
  </gazebo>
```
- `lykos.urdf.xacro`: Se añadieron las lineas que implementan el archivo de configuración de los controladores.
```xacro
<!-- Gazebo ROS control pluggins -->
<xacro:include filename="$(find lykos_description)/urdf/ros2_control.urdf.xacro"/>
<xacro:arg name="config_controllers" default="$(find lykos_description)/config/lykos_controllers.yaml"/>
<xacro:arg name="update_rate" default="20"/>
<xacro:ros2_control/>
```
- `ros2_control.urdf.xacro`: Creamos el archivo que especifica la configuración de ros2_control para controlar las ruedas del robot mediante ROS 2.
- `robot_brigde.yaml`: Creamos esste archivo para poder implementar las funciones de los sensores (solamente los del gps y imu).
- `lykos_controllers.yaml`: Creamos este archivo para poder implementar los controladores del brazo y de las ruedas.
- `Launchers`: además del que se hizo para la anterior parte, se tuvo que proceder a crear dos nuevos launchers. Uno para el gazebo y otro para lanzar los contoladores.
  - `robot_gazebo.launch.py`: En este, como su propio nombre indica, launchea el gazebo y al robot para poder visualizarlo. Además, lanza rviz, el mundo usado, las camaras, entre otras cosas.
  - `robot_controllers.launch.py`: Por otro lado, este launcher está especificamente creado para launchear los archivos de configuración de los controladores para poder cargarlos y activarlos.
- `package.xml`: para poder utilizar las , había que implementar las dependencias necesarias para teleoperar el robot, usar los controladores, etc.

Una vez hemos tenido estos archivos, pasamos a la parte de configuración del scara. Para ello, creamos otro paquete de ros con las carpetas launch y config; y usamos el asistente de moveit con el siguiente comando: `ros2 launch moveit_setup_assistant setup_assistant.launch.py`. Y siguiendo los pasos del video proporcionado en el aula virutal, generamos el paquete. Y cambiamos en los siguientes archivos lo siguiente:

- `joint_limits.yaml`: En cada joint debemos de poner esto:
```yaml
has_velocity_limits: true
has_acceleration_limits: true
max_acceleration: 10.0
```
- `lykos.ros2_control.urdf.xacro`: cambiar el plugin existente por este: <plugin>gz_ros2_control/GazeboSimSystem</plugin>.
- `lykos.urdf.xacro`: debemos añadir esta linea <xacro:include filename="lykos.srdf"/> y cambiar el nombre de lykos_ros2_control a ScaraArm.
- `moveit_controllers.yaml`: Al principio debemos de añadir un use_sim_time:true y en cada controller:
```yaml
action_ns: follow_joint_trajectory
default: true
use_sim_time: true
```

Una vez que tenemos esto, tras comprobar que tenemos los dos paquetes correctamente implementados y los launchers modificados con los nuevos cambios, procedemos a la siguiente parte de la parte B. 
Launcheamos los siguientes launchers:

```bash
ros2 launch lykos_description robot_gazebo.launch.py world_name:=urjc_excavation_msr
ros2 launch lykos_moveit_config move_group.launch.py
ros2 launch lykos_description robot_controllers.launch.py
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

![Captura desde 2025-05-08 21-27-04](https://github.com/user-attachments/assets/3a817cbc-52f8-4f40-8922-2c2eaebe3e3e)
![Captura desde 2025-05-08 21-19-56](https://github.com/user-attachments/assets/852ab205-0858-458c-b568-bf0e4e26ec5a)


Con esto lanzamos todo lo relacionado con la simulación del robot y su teleoperación.
![Captura desde 2025-05-08 21-19-26](https://github.com/user-attachments/assets/3bf390bd-c38a-423d-bc80-2f14eae48b6b)

Como veremos en el siguiente video, la teleoperacion de las ruedas fucnionan ya que los controladores se han cargado correctamente. Aunque como se puede ver, no se manejar el robot correctamente.

[wheel_controllers]()

Por otro lado, para poder mover el brazo, usaremos el plugin en rviz de moveit, MotionPlanning. Una vez lo añadimos, usaremos dos pestañas en concreto.
- `Context`: Esta pestaña sirve para elegit el planificador con el que queremos realizar el plan. Muy importante seleccionar el planificador OMPL para poder planificar el movimiento de nuestro brazo, ya que sino no funciona. Cabe destacar que en la siguiente pestaña cada vez que se cambia de grupo, el planificador se configura al de por defecto (que es CHOMP), por ello hay que tener en cuenta esto cada vez que lo realizamos para poder cambiarlo y no experimentar errores.
- `Planning`: Esta otra pestaña es la que nos permite realizar las planificaciones que necesitemos y ejecutarlas. Podremos elegir entre los dos grupos creados en el paquete moveit, Scara o Gripper, y sus correspondientes poses. La actual, target, release, etc. En este caso tuve que añadir una pose mas a mano ya que a la hora de coger la caja no ajusté bien al principio los parámetros. Por último, plan and execute es el botón que realiza la acción. Veremos, tanto en el simulador de gazebo como en el rviz, el movimiento que realiza el scara (como se observa en la siguiente imagen).

![Captura desde 2025-05-10 15-20-12](https://github.com/user-attachments/assets/78330c06-5f41-4740-bd17-420742716a49)


![Video_pick_and_place](https://urjc-my.sharepoint.com/:v:/g/personal/i_diezd_2022_alumnos_urjc_es/EU3Gq4X8mPNHizQja2--0XkB-a8X2gV9mKxNTDHSXPa3CQ?nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJTdHJlYW1XZWJBcHAiLCJyZWZlcnJhbFZpZXciOiJTaGFyZURpYWxvZy1MaW5rIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXcifX0%3D&e=wLPijL)


![Captura desde 2025-05-10 13-12-37](https://github.com/user-attachments/assets/f635f7ba-5fd8-4201-9141-987867b2096b)































