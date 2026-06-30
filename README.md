ALUNOS: Pedro Henrique Rodrigues Alves e Felipe da Silva Oliveira


-Subir UERANSIM:

DEPENDÊNCIAS:

sudo apt update
sudo apt upgrade
sudo apt install make
sudo apt install gcc
sudo apt install g++
sudo apt install libsctp-dev lksctp-tools
sudo apt install iproute2
sudo snap install cmake --classic


INSTALAÇÃO:
git clone https://github.com/aligungr/UERANSIM.git
cd ~/UERANSIM
make


CONFIGURAÇÃO DO ARQUIVO DA GNB:
vim ~/UERANSIM/config/open5gs-gnb.yaml

# MODIFICAR APENAS ESSES CAMPOS:

mcc: '999'     # Coloca '001' 
mnc: '70'     # Coloca '01' 

linkIp: 127.0.0.1     # Deixa igual porque a UE e a GNB estão na mesma máquina 
ngapIp: 127.0.0.1     # Coloca o IP dessa máquina (VM1)
gtpIp: 127.0.0.1     # Coloca o IP dessa máquina (VM1)

amfConfigs:
  - address: 127.0.0.5     # Coloca o IP da máquina onde está o Open5gs (VM2)


CONFIGURAÇÃO DO ARQUIVO DA UE:
vim ~/UERANSIM/config/open5gs-ue.yaml

# MODIFICAR APENAS ESSES CAMPOS:

supi: 'imsi-999700000000001'     # Coloca 'imsi-001010000000001' 
mcc: '999'     # Coloca '001 ' 
mnc: '70'     # Coloca '01 ' 

gnbSearchList:
  - 127.0.0.1     # Deixa igual porque a UE e a GNB estão na mesma máquina


-Subir o OPEN5GS:

INSTALAÇÃO
git clone https://github.com/herlesupreeth/docker_open5gs.git
cd ~/docker_open5gs.git

# FAZ O PULL DAS IMAGENS DOCKER
docker pull ghcr.io/herlesupreeth/docker_open5gs:master
docker tag ghcr.io/herlesupreeth/docker_open5gs:master docker_open5gs

docker pull ghcr.io/herlesupreeth/docker_grafana:master
docker tag ghcr.io/herlesupreeth/docker_grafana:master docker_grafana

docker pull ghcr.io/herlesupreeth/docker_metrics:master
docker tag ghcr.io/herlesupreeth/docker_metrics:master docker_metrics


CONFIGURAÇÃO DO .ENV:
vim ~/docker_open5gs.git/.env

DOCKER_HOST_IP=192.168.1.223     # Coloca o IP dessa máquina (VM2)
UPF_ADVERTISE_IP=172.22.0.8     # Coloca o IP dessa máquina (VM2)
UE_IPV4_INTERNET=192.168.100.0/24     # Coloca 10.45.0.0/16
UE_IPV4_IMS=192.168.101.0/24     # Coloca 10.46.0.0/16


CONFIGURAÇÃO DO ARQUIVO DE EXECUÇÃO:
vim ~/docker_open5gs.git/sa-deploy.yaml

# DESCOMENTA ESSE TRECHO DO AMF
...
    # ports:
    #   - "38412:38412/sctp"
…


# DESCOMENTA ESSE TRECHO DA UPF
...
    # ports:
    #   - "2152:2152/udp"
...


EXECUÇÃO:
# PARA SUBIR O CORE 5G E A UPF
docker compose -f sa-deploy.yaml up -d

# PARA DESATIVAR (QUANDO FOR DESLIGAR A MÁQUINA OU EDITAR ALGO)
docker compose -f sa-deploy.yaml down


	Adicionando a UE na WEB UI do Open5gs:
# ACESSA PELO NAVEGADOR (SÓ VAI PRECISAR FAZER ISSO DA PRIMEIRA VEZ)

<IP-DA-VM2>:9999

Username : admin
Password : 1423

# VAI EM “ADD A SUBSCRIBER”
IMSI*: 001010000000001     # SALVA E PODE SAIR

	
Na máquina do UERANSIM, executa os dois arquivos abaixo pra subir a GNB e a UE:
cd ~/UERANSIM/build

./nr-gnb -c ../config/open5gs-gnb.yaml
sudo ./nr-ue -c ../config/open5gs-ue.yaml



