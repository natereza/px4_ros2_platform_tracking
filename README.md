# px4_ros2_platform_tracking

README - Ana PX4 ROS 2 Platform Tracking

Projeto para simular um drone PX4 no Gazebo/GZ usando ROS 2 Humble + MAVROS2, com objetivo de futuramente controlar o drone para seguir uma plataforma móvel.

Base Docker usada:
https://github.com/felipeccalegari/mavros2_px4_docker

Este projeto usa:
- Windows + WSL/Ubuntu
- Docker Desktop
- PX4 1.16.1
- ROS 2 Humble
- MAVROS2
- Gazebo/GZ
- Jason BDI Agent
- noVNC ou XLaunch para interface gráfica

======================================================================
1. Instalação
======================================================================
## 1. Abrir programas necessários

Antes de começar, abrir no Windows:

1. XLaunch: multiple windows > start no client > disable acess control > finish
2. Docker Desktop: esperar estar rodando
3. Ubuntu/WSL

4. Clonar projeto do Felipe
```bash
mkdir mavros_px4
cd mavros_px4
git clone https://github.com/felipeccalegari/mavros2_px4_docker.git
```
5. Build a imagem do Docker
```bash
cd mavros2_px4_docker
docker compose -f docker-compose_xhost.yaml build
```
Obs: usar no-cache apenas na primeira vez construindo

6. Escolher GUI mode
```bash
xhost +local:docker
```
7. Se é WSL precisamos editar o arquivo
```bash
cd ~/mavros_px4/mavros2_px4_docker
cp docker-compose_xhost.yaml docker-compose_xhost_wsl.yaml
nano docker-compose_xhost_wsl.yaml
```
Removemos o bloco
```bash
devices:
  - /dev/dri:/dev/dri
```
Também precisamos montar o workspace ROS 2 do WSL dentro do container, para os arquivos não sumirem quando o container for recriado. Ainda no docker-compose_xhost_wsl.yaml, procurar a parte volumes: e adicionar:
```bash
  - ./ws:/root/ws
```
A parte de volumes fica parecida com isso:
```bash
volumes:
  - ./PX4-Autopilot:/opt/PX4-Autopilot
  - ./ws:/root/ws
```
```bash
Ctrl + O: “write out”, salvar.
Enter: confirma o nome do arquivo.
Ctrl + X: sai do nano.
```
Conferir se foi deletado mesmo, não deve aparecer nada
```bash
grep -n "dri" docker-compose_xhost_wsl.yaml
```

Sobe usando o arquivo novo
```bash
docker rm -f px4_ros2 2>/dev/null || true
xhost +local:docker
docker compose -f docker-compose_xhost_wsl.yaml up -d
```
9. Conferir se apareceu o docker, tem que aparecer "px4_ros2"
```bash
docker ps
```
10. Entrar no container
```bash
docker exec -it px4_ros2 bash
```
11. Rodar a simulação
```bash
make px4_sitl gz_x500
```
Se apareceu a simulação, deu certo
======================================================================
2. Fazer funcionar depois da primeira vez
======================================================================
Antes de começar, abrir no Windows:
1. XLaunch: multiple windows > start no client > disable acess control > finish
2. Docker Desktop: esperar estar rodando
3. Ubuntu/WSL

4. Entrar no container
```bash
cd ~/mavros_px4/mavros2_px4_docker
docker rm -f px4_ros2 2>/dev/null || true
xhost +local:docker
docker compose -f docker-compose_xhost_wsl.yaml up -d
docker exec -it px4_ros2 bash
```

5. Botar para rodar
```bash
cd /opt/PX4-Autopilot
make px4_sitl gz_x500
```
6. Para abrir outro terminal no mesmo container:
Abrimos um terminal ubuntu ( Ctrl + Alt + 4)
```bash
docker exec -it px4_ros2 bash
```

7. Comandos que podemos usar (ROS 2 Humble)

- ros2                            -> comando principal para interagir com o ROS 2 pelo terminal
- ros2 topic list                 -> lista todos os tópicos ativos
- ros2 topic echo /nome_do_topico -> mostra em tempo real as mensagens publicadas em um tópico
- ros2 topic type /nome_do_topico -> mostra o tipo de mensagem usado por um tópico
- ros2 topic info /nome_do_topico -> mostra informações do tópico, como tipo, publishers e subscribers
- ros2 node list                  -> lista os nós ROS 2 ativos
- ros2 run nome_do_pacote nome_do_executavel -> roda um executável de um pacote ROS 2
- ros2 launch nome_do_pacote arquivo.launch.py -> roda um arquivo de launch para iniciar nós/configurações
- colcon build                    -> compila o workspace ROS 2
- colcon build --packages-select nome_do_pacote -> compila só um pacote específico
- source install/setup.bash       -> carrega no terminal os pacotes compilados
- rclcpp                          -> biblioteca para programar nós ROS 2 em C++
- rclpy                           -> biblioteca para programar nós ROS 2 em Python
- ament_cmake                     -> sistema de build usado em pacotes ROS 2 com C++

# Editar o mundo para colocar uma plataforma móvel
1. Achar o mundo local
Abrir um terminal ubuntu
```bash
cd ~/mavros_px4/mavros2_px4_docker
find . -path "*Tools/simulation/gz/worlds/default.sdf"
```
Deve aparecer isso, é o arquivo que vamos editar
```bash
./PX4-Autopilot/Tools/simulation/gz/worlds/default.sdf
```

2. Abrir o mundo no vscode
```bash
code ./PX4-Autopilot/Tools/simulation/gz/worlds/default.sdf
```
3. Antes de começar a editar no vscode, a gente faz um backup
```bash
cd ~/mavros_px4/mavros2_px4_docker
sudo cp ./PX4-Autopilot/Tools/simulation/gz/worlds/default.sdf ./PX4-Autopilot/Tools/simulation/gz/worlds/default_backup.sdf
```
4. Editando no vs code, adicionamos a plataforma no mundo
Adicionar depois de
```bash
    </spherical_coordinates>
```
E antes de 
```bash
  </world>
</sdf>
```
Descrição da plataforma
```bash
    <!-- Plataforma móvel adicionada para o projeto -->
    <model name="moving_platform">
      <pose>5 0 0.05 0 0 0</pose>
      <static>false</static>

      <link name="platform_link">
        <inertial>
          <mass>10.0</mass>
          <inertia>
            <ixx>1.0</ixx>
            <iyy>1.0</iyy>
            <izz>1.0</izz>
            <ixy>0.0</ixy>
            <ixz>0.0</ixz>
            <iyz>0.0</iyz>
          </inertia>
        </inertial>

        <visual name="platform_visual">
          <geometry>
            <box>
              <size>3.0 3.0 0.1</size>
            </box>
          </geometry>
          <material>
            <ambient>0.1 0.1 0.8 1</ambient>
            <diffuse>0.1 0.1 0.8 1</diffuse>
            <specular>0.1 0.1 0.8 1</specular>
          </material>
        </visual>

        <collision name="platform_collision">
          <geometry>
            <box>
              <size>3.0 3.0 0.1</size>
            </box>
          </geometry>
        </collision>
      </link>
    </model>

```
Plataforma aparece em (x, y, z) = (5, 0, 0.05) com as dimensões (x, y, z) = (3 m x 3 m x 0.1 m) e a cor azul escuro

5. Salvar e testar para ver se a alteração funcionou
```bash
cd ~/mavros_px4/mavros2_px4_docker
docker rm -f px4_ros2 2>/dev/null || true
xhost +local:docker
docker compose -f docker-compose_xhost_wsl.yaml up -d
docker exec -it px4_ros2 bash
```

```bash
cd /opt/PX4-Autopilot
make px4_sitl gz_x500
```
Deve aparecer o drone e a plataforma

6. Criar nó em ROS 2 para controlar a plataforma
Abrir um terminal ubuntu e entrar no container
```bash
docker exec -it px4_ros2 bash
```
Criar workspace ROS 2
```bash
mkdir -p /root/ros2_ws/src
cd /root/ros2_ws/src
```
Criar pacote ROS 2 em Python
```bash
ros2 pkg create ana_platform_control --build-type ament_python --dependencies rclpy geometry_msgs
```

8. Editar nó no vs code
Antes de qualquer coisa, temos que nos certificar que o container tá rodando. Tem que aparecer "px4_ros2"
```bash
docker ps
```
Abre o vscode e aperta "Ctrl + Shift + P", seleciona "Dev Containers: Attach to Running Container" e escolhe "px4_ros2".

Depois que abrir uma nova janela do VS Code:
File > Open Folder
/root/ros2_ws/src/ana_platform_control

Agora estamos editando dentro do container

Dentro da pasta:

ana_platform_control/ana_platform_control/

cria:

moving_platform_controller.py

E bota esse código
```bash
#importar bibliotecas
import rclpy # biblioteca ROS 2 para Python
from rclpy.node import Node # classe base para criar um nó ROS 2

#criar uma classe para o controlador que é herdeira de Node
class MovingPlatformController(Node):
    def __init__(self):
        #inicializa o nó ros com esse nome
        super().__init__("moving_platform_controller")
        #imprime uma mensagem no log
        self.get_logger().info("Moving platform controller iniciado")


def main(args=None):
    #inicializa o ros no programa
    rclpy.init(args=args)

    #cria o nó
    node = MovingPlatformController()

    #mantém o nó rodando
    rclpy.spin(node)

    #finaliza o nó
    node.destroy_node()
    rclpy.shutdown()


#permite todar o arquivo diretamente como script, vamos rodar com 'ros2 run'
if __name__ == "__main__":
    main()
```
10. Vamos registrar o arquivo no setup.py, para o ROS 2 saber que ele pode ser rodado com ros2 run
Abrimos o arquivo 'setup.py'
Vamos preencher a lista de console_scripts
```bash
entry_points={
    'console_scripts': [
        'moving_platform_controller = ana_platform_control.moving_platform_controller:main',
    ],
},
```
12. Vamos ver se funciona
Abrir terminal ubuntu
```bash
docker exec -it px4_ros2 bash
```
```bash
cd /root/ros2_ws
colcon build --packages-select ana_platform_control
source install/setup.bash
ros2 run ana_platform_control moving_platform_controller
```
Deve aparecer a mensagem de inicialização do nodo.
Pronto, nosso nó funciona!
13. Agora vamos construir melhor o nó
Voltamos a editar o arquivo do nó
```bash
#importar bibliotecas
import rclpy # biblioteca ROS 2 para Python
from rclpy.node import Node # classe base para criar um nó ROS 2
import math
import subprocess #usamos para chamar um comando do terminal dentro do Python

#mensagens padrão do ROS
from geometry_msgs.msg import PoseStamped # posição + orientação + tempo
from geometry_msgs.msg import TwistStamped # linear + velocidade angular + tempo


#criar uma classe para o controlador que é herdeira de Node
class MovingPlatformController(Node):
    def __init__(self):
        #inicializa o nó ros com esse nome
        super().__init__("moving_platform_controller")
        #imprime uma mensagem no log
        self.get_logger().info("Moving platform controller iniciado")

        #nome do modelo no Gazebo, tem que ser igual ao nome do modelo em default.sdf
        self.model_name = "moving_platform"
        #serviço do gazebo que move modelos
        self.service_name = "/world/default/set_pose"

        # bota as posições iniciais da plataforma
        self.x0 = 5.0
        self.y0 = 0.0
        self.z0 = 0.05

        #define um movimento senoidal
        # x(t) = x0 + A sin(ωt)
        self.amplitude = 4.0
        self.omega = 0.25

        #define o periodo de atualização
        self.timer_period = 0.2 #atualiza a cada 0.2 segundos
        #tempo inicial, vamos usar para saber há quanto tempo o nó está rodando
        self.start_time = self.get_clock().now()
        #cria um timer ROS2
        self.timer = self.create_timer(
            self.timer_period,
            self.timer_callback
        )


        #cria um publisher no tópico "/moving_platform/pose" do tipo PoseStamped
        self.pose_publisher = self.create_publisher(
            PoseStamped,
            "/moving_platform/pose",
            10
        )
        #cria um publisher no tópico "/moving_platform/velocity" do tipo TwistStamped
        self.velocity_publisher = self.create_publisher(
            TwistStamped,
            "/moving_platform/velocity",
            10
        )

        
    #função chamada periodicamente pelo timer
    def timer_callback(self):
        #tempo agora
        now = self.get_clock().now()

        #tempo desde que o nó começou, em segundos
        elapsed_time = (now - self.start_time).nanoseconds * 1e-9

        #posição senoidal da plataforma no eixo x
        x = self.x0 + self.amplitude * math.sin(self.omega * elapsed_time)
        y = self.y0
        z = self.z0

        #velocidade correspondente: derivada de x(t)
        vx = self.amplitude * self.omega * math.cos(self.omega * elapsed_time)
        vy = 0.0
        vz = 0.0

        #move a plataforma no Gazebo.
        self.move_platform_in_gazebo(x, y, z)

        #publica os dados no ROS 2
        self.publish_pose(now, x, y, z)
        self.publish_velocity(now, vx, vy, vz)



    def move_platform_in_gazebo(self, x, y, z):
        #monta a requisição para o serviço do Gazebo
        request = (
            f'name: "{self.model_name}" '
            f'position {{x: {x} y: {y} z: {z}}} '
            f'orientation {{w: 1.0}}'
        )

        #chama o serviço /world/default/set_pose, que move o modelo 
        result = subprocess.run(
            [
                "gz", "service",
                "-s", self.service_name,
                "--reqtype", "gz.msgs.Pose",
                "--reptype", "gz.msgs.Boolean",
                "--timeout", "1000",
                "--req", request
            ],
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            text=True
        )

        #se falhar mostra um aviso
        if result.returncode != 0:
            self.get_logger().warn(
                f"Falha ao mover plataforma: {result.stderr}"
            )


    #cria mensagem de pose
    def publish_pose(self, time_stamp, x, y, z):
        pose_msg = PoseStamped()

        pose_msg.header.stamp = time_stamp.to_msg()
        pose_msg.header.frame_id = "world"

        pose_msg.pose.position.x = x
        pose_msg.pose.position.y = y
        pose_msg.pose.position.z = z

        pose_msg.pose.orientation.x = 0.0
        pose_msg.pose.orientation.y = 0.0
        pose_msg.pose.orientation.z = 0.0
        pose_msg.pose.orientation.w = 1.0

        self.pose_publisher.publish(pose_msg)
    
    #cria mensagem de velocidade
    def publish_velocity(self, time_stamp, vx, vy, vz):
        velocity_msg = TwistStamped()

        velocity_msg.header.stamp = time_stamp.to_msg()
        velocity_msg.header.frame_id = "world"

        velocity_msg.twist.linear.x = vx
        velocity_msg.twist.linear.y = vy
        velocity_msg.twist.linear.z = vz

        velocity_msg.twist.angular.x = 0.0
        velocity_msg.twist.angular.y = 0.0
        velocity_msg.twist.angular.z = 0.0

        self.velocity_publisher.publish(velocity_msg)


def main(args=None):
    #inicializa o ros no programa
    rclpy.init(args=args)

    #cria o nó
    node = MovingPlatformController()

    #mantém o nó rodando
    rclpy.spin(node)

    #finaliza o nó
    node.destroy_node()
    rclpy.shutdown()


#permite todar o arquivo diretamente como script, vamos rodar com 'ros2 run'
if __name__ == "__main__":
    main()
```
14. Testar para ver se o controlador da placa está funcionando
Terminal 1:
```bash
cd ~/mavros_px4/mavros2_px4_docker
xhost +local:docker
docker compose -f docker-compose_xhost_wsl.yaml up -d
docker exec -it px4_ros2 bash
```
```bash
cd /opt/PX4-Autopilot
make px4_sitl gz_x500
```
Terminal 2:
```bash
docker exec -it px4_ros2 bash
```
Deve abrir a simulação e a placa se mexer conforme o esperado

16. Agora vamos partir para o drone
Encontramos o tópico que dá a posição do drone
```bash
ros2 topic list | grep -E "pose|odom|position|local"
```
Encontramos o tópico /mavros/local_position/pose, vamos ver o que tem nele
```bash
ros2 topic echo --once /mavros/local_position/pose
```
Está funcionando e printa PoseStamped


18. Criar nodo de visão computacional simulada
Abrir a pasta no vs code
```bash
cd ~/mavros_px4/mavros2_px4_docker
sudo chown -R $USER:$USER ws
```
Coloca senha ubuntu e agora abre vs code
```bash
code ws/src/ana_platform_control
```

Dentro da pasta:
```bash
ana_platform_control/ana_platform_control/
```
cria o arquivo:
```bash
simulated_computer_vision.py
```

Agora vamos escrever o nó
```bash
#importar bibliotecas
#calcular a visao do drone usando abertura da camera do drone como tg
#e usando a altura do drone
import math

import rclpy # biblioteca ROS 2 para Python
from rclpy.node import Node # classe base para criar um nó ROS 2

#mensagens padrão do ROS
# receber pose do drone e da plataforma
from geometry_msgs.msg import PoseStamped # header (tempo + frame de referência), pose.position(x, y, z), pose.orientation(orientação em quaternion)
# publicar o pixel detectado
from geometry_msgs.msg import PointStamped # header(tempo + frame de referência), point.x, point.y,  point.z
# para dizer se a baleia está visível
from std_msgs.msg import Bool #boolean


# Assumimos as seguintes configurações para o drone
# Câmera apontada perfeitamente para baixo
# Assumimos o yaw como irrelevante, drone não gira em torno do eixo vertical
# Resolução da câmera
image_width = 800
image_height = 600
# Angulo da câmera em radianos
fov_x = math.pi/3 # 60 graus 
fov_y = math.pi/4 # 45 graus 

# taxa de atualização callback
time_callback = 0.1 #calculado 10 vezes por segundo


class SimulatedComputerVision(Node):
    def __init__(self):
        super().__init__("simulated_computer_vision")

        self.get_logger().info("Simulated computer vision iniciado")

        # configuração da imagem simulada
        self.image_width = image_width
        self.image_height = image_height
        self.center_x = self.image_width / 2.0
        self.center_y = self.image_height / 2.0

        # campo de visão da câmera
        self.fov_x = fov_x
        self.fov_y = fov_y


        # tópicos

        self.drone_pose = None
        self.platform_pose = None

        self.drone_subscriber = self.create_subscription(
            PoseStamped,
            "/mavros/local_position/pose",
            self.drone_pose_callback,
            10
        )

        self.platform_subscriber = self.create_subscription(
            PoseStamped,
            "/moving_platform/pose",
            self.platform_pose_callback,
            10
        )

        self.pixel_publisher = self.create_publisher(
            PointStamped,
            "/whale_detection/pixel",
            10
        )

        self.visible_publisher = self.create_publisher(
            Bool,
            "/whale_detection/visible",
            10
        )
        # chama o callback no tempo que quisermos
        self.timer = self.create_timer(
            time_callback,
            self.timer_callback
        )

    def drone_pose_callback(self, msg):
        self.drone_pose = msg

    def platform_pose_callback(self, msg):
        self.platform_pose = msg

    def timer_callback(self):
        if self.drone_pose is None or self.platform_pose is None:
            return

        drone_x = self.drone_pose.pose.position.x
        drone_y = self.drone_pose.pose.position.y
        drone_z = self.drone_pose.pose.position.z

        platform_x = self.platform_pose.pose.position.x
        platform_y = self.platform_pose.pose.position.y

        dx = platform_x - drone_x
        dy = platform_y - drone_y

        altitude = drone_z

        # protege de divisao por zero
        if altitude <= 0.1:
            self.publish_detection(False, 0.0, 0.0)
            return

        # calcula a dimensão da janela enxergada pela camera
        view_width = 2.0 * altitude * math.tan(self.fov_x / 2.0)
        view_height = 2.0 * altitude * math.tan(self.fov_y / 2.0)

        # se a distância do centro(x, y) é menor que a janela, está visivel
        visible = (
            abs(dx) <= view_width / 2.0 and
            abs(dy) <= view_height / 2.0
        )

        if not visible:
            self.publish_detection(False, 0.0, 0.0)
            return


        # calcula pixels da visão computacional, como se fosse uma imagem
        # pixel_x = centro_da_imagem + porcentagem_do_deslocamento_ate_a_borda * metade_da_imagem
        pixel_x = self.center_x + (dx/(view_width / 2.0)) * self.center_x
        # sinal negativo porque em imagem o eixo y cresce para baixo.
        pixel_y = self.center_y - (dy/(view_height / 2.0)) * self.center_y
        #publica visible true e o pixel calculado
        self.publish_detection(visible, pixel_x, pixel_y)

    def publish_detection(self, visible, pixel_x, pixel_y):
        now = self.get_clock().now().to_msg()

        visible_msg = Bool()
        visible_msg.data = visible

        pixel_msg = PointStamped()
        pixel_msg.header.stamp = now
        pixel_msg.header.frame_id = "camera_image"

        pixel_msg.point.x = pixel_x
        pixel_msg.point.y = pixel_y
        pixel_msg.point.z = 0.0

        self.visible_publisher.publish(visible_msg)
        self.pixel_publisher.publish(pixel_msg)


def main(args=None):
    rclpy.init(args=args)

    node = SimulatedComputerVision()
    rclpy.spin(node)

    node.destroy_node()
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```

Agora colocamos no setup.py, dentro de console_scripts:
```bash
'simulated_computer_vision = ana_platform_control.simulated_computer_vision:main',
```

Confere se no package.xml tem:

```bash
<depend>std_msgs</depend>
```
Se não tiver, coloca.

Agora vamos ver se está funcionando certinho
```bash
cd ~/mavros_px4/mavros2_px4_docker
docker exec -it px4_ros2 bash
```

```bash
cd /root/ws
colcon build --symlink-install --packages-select ana_platform_control
source install/setup.bash
ros2 run ana_platform_control simulated_computer_vision
```
Deve aparecer "[INFO] [1781196119.322675343] [simulated_computer_vision]: Simulated computer vision iniciado"

Agora, para ver se ele está publicando, abre outro terminal Ubuntu
```bash
docker exec -it px4_ros2 bash
```
Deve aparecer "/whale_detection/pixel" e "/whale_detection/visible"
```bash
source /root/ws/install/setup.bash
ros2 topic list | grep whale
```


Agora vamos construir o nodo do controlador

Criar arquivo pid.py
```bash
class PID:
    def __init__(self, kp, ki, kd, dt):
        #ganhos 
        self.kp = kp #proporcional
        self.ki = ki #integral
        self.kd = kd #derivativo

        #período de loop
        self.dt = dt

        #valores
        self.referencia = {"x": 0.0, "y": 0.0}
        self.erro = {"x": 0.0, "y": 0.0}
        self.erro_anterior = {"x": 0.0, "y": 0.0}
        self.integral = {"x": 0.0, "y": 0.0}
        self.derivada = {"x": 0.0, "y": 0.0}

    #Metodos

    def atualizar_referencia(self, x, y):
        self.referencia['x'] = x
        self.referencia['y'] = y

    def atualizar_erro(self, coord_x, coord_y):
        self.erro_anterior['x'] = self.erro['x']
        self.erro_anterior['y'] = self.erro['y']

        self.erro['x'] = self.referencia["x"] - coord_x
        self.erro['y'] = self.referencia["y"] - coord_y

    def atualizar_integral(self):
        self.integral["x"] += self.erro['x'] * self.dt
        self.integral["y"] += self.erro['y'] * self.dt

    def atualizar_derivada(self):
        self.derivada["x"] = (self.erro['x'] - self.erro_anterior['x']) / self.dt
        self.derivada["y"] = (self.erro['y'] - self.erro_anterior['y']) / self.dt

    def atualizar_pid(self, coord_x, coord_y):
        self.atualizar_erro(coord_x, coord_y)
        self.atualizar_integral()
        self.atualizar_derivada()

    def calculo_variacoes(self):
        vx = self.kp * self.erro['x'] + self.ki * self.integral['x'] + self.kd * self.derivada['x']
        vy = self.kp * self.erro['y'] + self.ki * self.integral['y'] + self.kd * self.derivada['y']
        
        
        return vx, vy

```

OK ARQUIVO TEMPORÀRIO

cria controller.py
```bash
import rclpy
from rclpy.node import Node

from rclpy.qos import QoSProfile
from rclpy.qos import ReliabilityPolicy
from rclpy.qos import HistoryPolicy
from rclpy.qos import DurabilityPolicy

from geometry_msgs.msg import PointStamped
from geometry_msgs.msg import PoseStamped
from geometry_msgs.msg import TwistStamped
from std_msgs.msg import Bool

from mavros_msgs.msg import State
from mavros_msgs.srv import CommandBool
from mavros_msgs.srv import SetMode


class DronePixelController(Node):
    def __init__(self):
        super().__init__("drone_pixel_controller")

        self.get_logger().info("Drone pixel controller iniciado")

        # Imagem simulada
        self.image_width = 800
        self.image_height = 600
        self.center_x = self.image_width / 2.0
        self.center_y = self.image_height / 2.0

        # Drone sobe e tenta manter 10 m
        self.target_altitude = 10.0

        # Ganhos simples
        self.k_pixel = 0.01
        self.k_z = 0.5

        # Limites de velocidade
        self.max_xy_speed = 2.0
        self.max_z_speed = 0.8

        # Estados recebidos
        self.pixel = None
        self.visible = False
        self.pose = None
        self.state = State()

        # QoS compatível com tópicos do MAVROS/PX4
        mavros_qos = QoSProfile(
            reliability=ReliabilityPolicy.BEST_EFFORT,
            durability=DurabilityPolicy.VOLATILE,
            history=HistoryPolicy.KEEP_LAST,
            depth=10
        )

        # Recebe pixel detectado
        self.create_subscription(
            PointStamped,
            "/whale_detection/pixel",
            self.pixel_callback,
            10
        )

        # Recebe se alvo está visível
        self.create_subscription(
            Bool,
            "/whale_detection/visible",
            self.visible_callback,
            10
        )

        # Recebe pose do drone
        self.create_subscription(
            PoseStamped,
            "/mavros/local_position/pose",
            self.pose_callback,
            mavros_qos
        )

        # Recebe estado do PX4/MAVROS
        self.create_subscription(
            State,
            "/mavros/state",
            self.state_callback,
            10
        )

        # Publica comando de velocidade
        self.velocity_publisher = self.create_publisher(
            TwistStamped,
            "/mavros/setpoint_velocity/cmd_vel",
            10
        )

        # Serviços para OFFBOARD e ARM
        self.arming_client = self.create_client(
            CommandBool,
            "/mavros/cmd/arming"
        )

        self.set_mode_client = self.create_client(
            SetMode,
            "/mavros/set_mode"
        )

        self.last_service_call = self.get_clock().now()

        # Loop de controle a 50 Hz
        self.timer = self.create_timer(0.02, self.timer_callback)

    def pixel_callback(self, msg):
        self.pixel = msg

    def visible_callback(self, msg):
        self.visible = msg.data

    def pose_callback(self, msg):
        self.pose = msg

    def state_callback(self, msg):
        self.state = msg

    def timer_callback(self):
        if self.pose is None:
            self.publish_velocity(0.0, 0.0, 0.0)
            return

        self.try_offboard_and_arm()

        drone_z = self.pose.pose.position.z

        # Controle vertical: subir até 10 m
        error_z = self.target_altitude - drone_z
        vz = self.k_z * error_z
        vz = self.clamp(vz, -self.max_z_speed, self.max_z_speed)

        if abs(error_z) < 0.2:
            vz = 0.0

        # Controle horizontal pelo pixel
        vx = 0.0
        vy = 0.0

        if self.visible and self.pixel is not None:
            pixel_x = self.pixel.point.x
            pixel_y = self.pixel.point.y

            error_x = pixel_x - self.center_x
            error_y = pixel_y - self.center_y

            vx = self.k_pixel * error_x

            # pixel_y cresce para baixo; por isso o sinal negativo
            vy = -self.k_pixel * error_y

            vx = self.clamp(vx, -self.max_xy_speed, self.max_xy_speed)
            vy = self.clamp(vy, -self.max_xy_speed, self.max_xy_speed)

        self.publish_velocity(vx, vy, vz)

    def publish_velocity(self, vx, vy, vz):
        msg = TwistStamped()

        msg.header.stamp = self.get_clock().now().to_msg()
        msg.header.frame_id = "map"

        msg.twist.linear.x = vx
        msg.twist.linear.y = vy
        msg.twist.linear.z = vz

        msg.twist.angular.x = 0.0
        msg.twist.angular.y = 0.0
        msg.twist.angular.z = 0.0

        self.velocity_publisher.publish(msg)

    def try_offboard_and_arm(self):
        now = self.get_clock().now()
        elapsed = (now - self.last_service_call).nanoseconds * 1e-9

        if elapsed < 2.0:
            return

        self.last_service_call = now

        if not self.state.connected:
            self.get_logger().info("Aguardando conexão com PX4/MAVROS...")
            return

        if self.state.mode != "OFFBOARD":
            if self.set_mode_client.service_is_ready():
                request = SetMode.Request()
                request.custom_mode = "OFFBOARD"
                self.set_mode_client.call_async(request)
                self.get_logger().info("Solicitando OFFBOARD")
            return

        if not self.state.armed:
            if self.arming_client.service_is_ready():
                request = CommandBool.Request()
                request.value = True
                self.arming_client.call_async(request)
                self.get_logger().info("Solicitando ARM")
            return

    def clamp(self, value, min_value, max_value):
        return max(min_value, min(max_value, value))


def main(args=None):
    rclpy.init(args=args)

    node = DronePixelController()
    rclpy.spin(node)

    node.destroy_node()
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```

agora bota no setup.py
```bash
'controller = ana_platform_control.controller:main',
```

e bota no package.xml
```bash
<depend>mavros_msgs</depend>
```




Rodando do zero
Terminal 1:
```bash
cd ~/mavros_px4/mavros2_px4_docker
xhost +local:docker
docker compose -f docker-compose_xhost_wsl.yaml up -d
docker exec -it px4_ros2 bash
```
Dentro:
```bash
cd /opt/PX4-Autopilot
make px4_sitl gz_x500
```
Terminal 2:

docker exec -it px4_ros2 bash

Dentro:

cd /root/ws
colcon build --symlink-install --packages-select ana_platform_control
source install/setup.bash
ros2 run ana_platform_control moving_platform_controller

Aí a plataforma deve mexer. Depois disso, seguimos para o nó da câmera simulada.
