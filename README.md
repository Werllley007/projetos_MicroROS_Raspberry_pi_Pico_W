# Configurando a ROS2 Foxy MicroRos Raspberry Pico W

Este guia detalha o processo de configuração do ambiente, compilação e execução de um exemplo básico do micro-ROS em uma Raspberry Pi Pico W, utilizando o Pico SDK e o toolchain ARM GCC.

**Pré-requisitos**
Para seguir este manual, você precisará de:
- Um ambiente Linux (Ubuntu 20.04 no Docker).
- Uma placa Raspberry Pi Pico W.
- As extensões do VS Code C/C++ e CMake Tools instaladas.

# Projeto a ROS2 Foxy MicroRos Raspberry Pico W via Serial

## 1. Instalação das Dependências Necessárias
Começaremos instalando as ferramentas de desenvolvimento essenciais, incluindo o compilador ARM GCC e o CMake.

```bash
sudo apt update
sudo apt install build-essential cmake gcc-arm-none-eabi libnewlib-arm-none-eabi doxygen git python3
```

## 2. Obtenção dos Códigos-Fonte (Workspace)
Criaremos um workspace (micro_ros_ws) e clonaremos os repositórios necessários: o Pico SDK (base) e o exemplo micro-ROS.

```bash
mkdir -p ~/micro_ros_ws/src
cd ~/micro_ros_ws/src
```

```bash
# 1. Clona o Pico SDK (incluindo submódulos)
git clone --recurse-submodules https://github.com/raspberrypi/pico-sdk.git

# 2. Clona o repositório de exemplo micro-ROS
git clone https://github.com/micro-ROS/micro_ros_raspberrypi_pico_sdk.git
```

## 3. Configuração do VS Code e CMake
Para que o VS Code e o CMake encontrem o SDK e o compilador, configuraremos o workspace.

### A. Configuração do Caminho do SDK
Navegue até o diretório do projeto e crie o arquivo de configuração settings.json para o VS Code:
```
```bash
cd ~/micro_ros_ws/src/micro_ros_raspberrypi_pico_sdk
mkdir -p .vscode
touch .vscode/settings.json
```

Abra o arquivo .vscode/settings.json em seu editor e adicione o seguinte conteúdo:

```bash
JSON

{
    "cmake.configureEnvironment": {
        "PICO_SDK_PATH": "/home/$USER/micro_ros_ws/src/pico-sdk"
    }
}
```
Nota: A variável PICO_SDK_PATH é essencial para que o CMake saiba onde encontrar a biblioteca base de desenvolvimento.

### B. Configuração do Kit de Compilação
Abra o projeto no VS Code: code .

Abra a Paleta de Comandos (Ctrl+Shift+P).

Procure por CMake: Scan for Kits e execute.

Procure por CMake: Select a Kit e selecione o compilador GCC for arm-none-eabi.

## 4. Compilação do Exemplo
Com o Kit selecionado, estamos prontos para compilar o firmware do exemplo.

Abra a Paleta de Comandos (Ctrl+Shift+P).

Execute CMake: Build.

Se a compilação for bem-sucedida, será criada a pasta build contendo o arquivo pico_micro_ros_example.uf2.

## 5. Upload para a Raspberry Pi Pico
O arquivo .uf2 gerado deve ser copiado para a memória da Pico para iniciar a execução.

Modo BOOTSEL: Conecte sua Raspberry Pi Pico ao computador enquanto pressiona o botão branco rotulado BOOTSEL. A placa será montada como um pendrive (geralmente sob o caminho /media/$USER/RPI-RP2).

Copiar o Arquivo: Copie o firmware para a placa:

```bash
cp build/pico_micro_ros_example.uf2 /media/$USER/RPI-RP2
```

Assim que a cópia for concluída, a placa será automaticamente reinicializada e começará a executar o código.

## 6. Configuração e Execução do micro-ROS Agent
Para que o ROS 2 no seu host se comunique com a Pico, é necessário rodar o micro-ROS Agent. Usaremos o Snap para a instalação mais fácil (assumindo Ubuntu).

### A. Instalar e Configurar o Agent

```bash
# 1. Instalar o micro-ros-agent via Snap
sudo snap install micro-ros-agent

# 2. Configurar o hotplug (necessário para detecção serial)
sudo snap set core experimental.hotplug=true
sudo systemctl restart snapd

# 3. Conectar a permissão serial (A Pico deve estar plugada)
snap connect micro-ros-agent:serial-port snapd:pico
```

### B. Rodar o Exemplo
Verifique o nome do dispositivo serial (geralmente /dev/ttyACM0).

Inicie o Agent com a taxa de baud correta:

```bash
micro-ros-agent serial --dev /dev/ttyACM0 baudrate=115200
```

Se o LED da Pico acender (indicando que o main loop está rodando), o Agent está conectado!

## 7. Verificação no ROS 2 (Host)
Em um novo terminal, utilize as ferramentas ROS 2 para verificar a comunicação.

Configurar o Ambiente ROS 2 (Use o caminho correto para sua distribuição, ex: Foxy, Humble):

```bash
source /opt/ros/foxy/setup.bash
```

Listar Nodes (Verifica se o micro-ROS node está visível):

```bash
ros2 node list
# Espera-se ver: /pico_node
```

Monitorar o Tópico (Verifica o fluxo de dados):

```bash
ros2 topic echo /pico_publisher
```

### Espera-se ver a mensagem std_msgs/Int32 sendo publicada a cada segundo.
Com isso, a ponte de comunicação entre sua Raspberry Pi Pico e o ROS 2 está completa!



# Projeto a ROS2 Foxy MicroRos Raspberry Pico W com Wifi

### A. Rodar o Exemplo

Mude aqui no arquivo picow_udp_transports.h:

```bash
#define ROS_AGENT_IP_ADDR   "192.168.0.8"    // You need modify IP
```

Coloque o IP que você encontrou com o comando abaixo:

```bash
ip addr show | grep 'inet ' | grep -v '127.0.0.1' | awk '{print $2}' | cut -d/ -f1
```

Coloque os dados da rede (WIFI_SSID, WIFI_PASSWORD) no arquivo micro_ROS_UDP.c:

```bash
printf("Connecting to WiFi...\n");
    if (cyw43_arch_wifi_connect_timeout_ms(WIFI_SSID, WIFI_PASSWORD, CYW43_AUTH_WPA2_AES_PSK, 30000)) {
        printf("failed to connect.\n");
        return 1;
    } else {
        printf("Connected.\n");
    }
```

### B. Rodar o Exemplo
Verifique o nome do dispositivo serial (geralmente /dev/ttyACM0).

Inicie o Agent com a taxa de baud correta:

```bash
ros2 run micro_ros_agent micro_ros_agent udp4 --port 8889
```

Sem entrar no conteiner:

```bash
docker run -it --rm --net=host microros/micro-ros-agent:foxy udp4 -p 8889
```

# Configure o CmakeLists.txt

link_directories(libmicroros)
add_executable(micro_ROS_UART 
        micro_ROS_UART.c 
        pico_uart_transport.c
        picow_udp_transports.c
)

# Add the standard library to the build
target_link_libraries(micro_ROS_UART
        pico_stdlib
        microros        
)

# Add the standard include files to the build
target_include_directories(micro_ROS_UART PRIVATE
        ${CMAKE_CURRENT_LIST_DIR}
        ${CMAKE_CURRENT_LIST_DIR}/libmicroros/include
)
