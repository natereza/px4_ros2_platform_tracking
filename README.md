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
