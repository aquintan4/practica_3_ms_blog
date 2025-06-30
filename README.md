# Práctic Descripción del robot en Xacro, Moovit y Controladores
En esta práctica he montado la descripción de un robot SCARA usando archivos `.xacro`, que básicamente es como una versión más flexible de URDF. 
La idea era tener todo bien organizado en piezas reutilizables, en vez de meter todo el robot en un solo archivo grande.
```
/rover_description/
├── robots
│   └── robot.urdf.xacro
└── urdf
    ├── base
    │   └── robot_base.urdf.xacro
    ├── clamp
    │   └── clamp.urdf.xacro
    ├── leg
    │   └── robot_leg.urdf.xacro
    ├── ros2_control.urdf.xacro
    ├── scara
    │   └── scara.urdf.xacro
    ├── sensors
    │   ├── camera.urdf.xacro
    │   └── imu.urdf.xacro
    └── utils
        └── utils.urdf.xacro
```
El robot completo está definido en `robot.urdf.xacro`, pero este archivo lo que hace principalmente es incluir los distintos módulos:
- `robot_base.urdf.xacro`: define la base fija del robot,
- `robot_leg.urdf.xacro`: con las ruedas
- `clamp.urdf.xacro`: pinza simple
- `imu.urdf.xacro` y `camera.urdf.xacro`: sensores especificados por el enunciado
- `scara.urdf.xacro`: defino las articulaciones principales

Esta modulación hace que el fichero robot se obtenga el robot con unas sencillas intrucciones:

![image](https://github.com/user-attachments/assets/02a38ddd-78be-464e-9e65-2b64db42d9e8)
![image](https://github.com/user-attachments/assets/1e5dd128-a40f-4986-b2e7-f4aa28b210b3)

Y el arbol de Transofadas corresponde con:
[frames_2025-06-29_21.29.44.pdf](https://github.com/user-attachments/files/20975599/frames_2025-06-29_21.29.44.pdf)
![image](https://github.com/user-attachments/assets/92085488-9e62-4b22-a074-a29df989b5eb)

## Configuración controladores
Para controlar el robot tanto desde ROS como en simulación, he configurado el sistema de control usando `ros2_control` y MoveIt.
### Controladores definidos
En `rover_controllers.yaml` y `ros2_controllers.yaml` se definen los controladores principales que se van a lanzar:

- `joint_state_broadcaster`: publica el estado de todas las articulaciones
- `diff_cont`: controlador de tipo `DiffDriveController` para el movimiento diferencial del rover, con sus parámetros de ruedas, límites de velocidad y aceleración.
- `scara_controller`: controlador `JointTrajectoryController` para el brazo SCARA. Controla las articulaciones `q1_joint`, `q2_joint`, `d3_prismatic_joint` y `q4_orientation_joint`.
- `gripper_controller`: también un `JointTrajectoryController` pero para el gripper, controlando las dos pinzas.

Cada controlador tiene definidos los `command_interfaces` (posición) y `state_interfaces` (posición y velocidad).

Para la planificación de movimientos con el brazo, he usado MoveIt a través de la configuración generada con el asistente. 
En el archivo `gazebo_launch.launch.py` se inicializa MoveIt junto con la simulación de Gazebo, y se lanza RViz ya preconfigurado con el robot.
También se incluyen los nodos necesarios para publicar la descripción del robot (`robot_state_publisher`) y las interfaces necesarias para visualizar y simular el estado.

### Bridge ROS Gazebo
En `rover_bridge.yaml` he configurado los bridges necesarios para que Gazebo y ROS2 puedan comunicarse correctamente:
- `/clock` sincronizar el tiempo simulado. 
- `/tf` para transmitir las tfs del mundo simulado.
- `/imu/data` para recibir los datos del sensor inercial.

![image](https://github.com/user-attachments/assets/91406c5b-d0ed-4b73-abd3-9b83d22da3f4)

Vemos que todos los controladores se han activado con éxito, también podemos ver como la simulación se ha lanzado correctamente:
![image](https://github.com/user-attachments/assets/62a71097-f00d-4b3c-9024-75adb944018e)
![image](https://github.com/user-attachments/assets/b7e495c3-e832-4627-ba42-8fafb076b84e)

También vemos que los sensores están correctamente funcionando y colocados segun los estándares de ROS:
![image](https://github.com/user-attachments/assets/1fbf28b4-57d3-437f-93f3-e64f99263dce)
Cambiando la posición se actualiza la imagen:
![image](https://github.com/user-attachments/assets/d33209f4-522c-4895-aa7a-ae3855b3a804)
También vemos que se está publicando los datos de la imu:
![image](https://github.com/user-attachments/assets/9e035256-16fa-484c-a1b4-7fdcdfa0d66a)
![image](https://github.com/user-attachments/assets/8f11d866-4c15-4ad3-b24d-651d9ce5b7cb)

También podemos ver que el plugin de Movit para Rviz2 está bien configurado:
Tanto los controles del gripper como el de la cadena cinemática del scara
![image](https://github.com/user-attachments/assets/eeeb0d38-f689-4205-8410-1c3f43344be8)
![image](https://github.com/user-attachments/assets/da31e1cf-d869-404c-8018-7c6c4c521705)


