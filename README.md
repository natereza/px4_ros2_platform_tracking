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
18. 




