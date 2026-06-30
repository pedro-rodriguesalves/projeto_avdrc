Configuração do UERANSIM e Open5GS

Alunos: Pedro Henrique Rodrigues Alves e Felipe da Silva Oliveira

Arquitetura

-   VM1: UERANSIM (gNB + UE)
-   VM2: Open5GS (Core 5G)
-   VM3: Open5GS (UPF)


1. Instalação do UERANSIM (VM1)

Dependências

    sudo apt update
    sudo apt upgrade

    sudo apt install make
    sudo apt install gcc
    sudo apt install g++
    sudo apt install libsctp-dev lksctp-tools
    sudo apt install iproute2

    sudo snap install cmake --classic

Instalação

    git clone https://github.com/aligungr/UERANSIM.git
    cd ~/UERANSIM
    make

2. Configuração da gNB

Edite:

    vim ~/UERANSIM/config/open5gs-gnb.yaml

Altere:

    mcc: '001'
    mnc: '01'

    linkIp: <IP_VM1>
    ngapIp: <IP_VM1>
    gtpIp: <IP_VM1>

    amfConfigs:
      - address: <IP_VM2>

3. Configuração da UE

Edite:

    vim ~/UERANSIM/config/open5gs-ue.yaml

Altere:

    supi: 'imsi-001010000000001'
    mcc: '001'
    mnc: '01'

    gnbSearchList:
      - <IP_VM1>

4. Instalação do Open5GS (VM2)

    git clone https://github.com/herlesupreeth/docker_open5gs.git
    cd ~/docker_open5gs

Baixar imagens

    docker pull ghcr.io/herlesupreeth/docker_open5gs:master
    docker tag ghcr.io/herlesupreeth/docker_open5gs:master docker_open5gs

    docker pull ghcr.io/herlesupreeth/docker_grafana:master
    docker tag ghcr.io/herlesupreeth/docker_grafana:master docker_grafana

    docker pull ghcr.io/herlesupreeth/docker_metrics:master
    docker tag ghcr.io/herlesupreeth/docker_metrics:master docker_metrics

5. Configurar .env

    vim ~/docker_open5gs/.env

    DOCKER_HOST_IP=<IP_VM2>
    UPF_ADVERTISE_IP=<IP_VM2>
    UE_IPV4_INTERNET=10.45.0.0/16
    UE_IPV4_IMS=10.46.0.0/16

6. Configurar sa-deploy.yaml

Descomente:

AMF

    ports:
      - "38412:38412/sctp"

UPF

    ports:
      - "2152:2152/udp"

7. Executar Open5GS

    docker compose -f sa-deploy.yaml up -d

Parar:

    docker compose -f sa-deploy.yaml down

8. Adicionar Subscriber

Acesse:

http://:9999

Usuário: admin

Senha: 1423

IMSI:

001010000000001

9. Executar UERANSIM

    cd ~/UERANSIM/build

    ./nr-gnb -c ../config/open5gs-gnb.yaml

    sudo ./nr-ue -c ../config/open5gs-ue.yaml
