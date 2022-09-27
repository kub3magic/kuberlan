# kuberlan

Aqui vao ficar todas as configuracoes do cluster local com GPU de aws eks anywhere, e tambem as instrucoes para instalar


## Instrucoes:

### Preparar arquivos de sistema operacional

  - A instalacao envolve usar a /tmp sem ser pelo root user. Achei um workaround: (https://unix.stackexchange.com/questions/71622/what-are-correct-permissions-for-tmp-i-unintentionally-set-it-all-public-recu)

chmod 1777 /tmp
find /tmp \
     -mindepth 1 \
     -name '.*-unix' -exec chmod 1777 {} + -prune -o \
     -exec chmod go-rwx {} +



  - Em "Artifacts" no site do eks anywhere baixar o "Hook OS" (kernel e initial ram disk). https://anywhere.eks.amazonaws.com/docs/reference/artifacts/#building-ubuntu-based-node-images
  - Ver em "building ubuntu based node images (https://anywhere.eks.amazonaws.com/docs/reference/artifacts/#building-ubuntu-based-node-images) como montar ubuntu.gz, que sera usado no pxe boot
  

### Instalacao do cluster
  - instalar eksctl conforme o site da aws orientar.
  - ativar UEFI networking stack com IPv4 netboot pela placa mae de todos os nodes
  - anotar os mac addresses de todos os nodes
  - escolher os IPs dos nodes
  - gerar o arquivo de configuracao inicial e escolher uma rede para os pods que seja diferente da rede que estiver usando para os nodes. Esta em: 
  - desativar o DHCP da rede onde os nodes forem instalados

> antes de inicializar o cluster, a porta 67 precisa estar desocupada. Ver isso por:
sudo netstat -nlp|grep 67

> matar o processo com o <PID> achado
kill <PID>

> destruir tambem os containers de docker:
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)

  - seguir a instalacao conforme o site do eks anywhere orienta

> comando pra inicializar o cluster:
eksctl anywhere create cluster -f eksa-mgmt-cluster.yaml --hardware-csv hardware.csv -v 9



### Post Instalation

  - ssh > colocar chaves nas maquinas
  - entrar com ssh e mudar a senha do "builder" e do "ec2-user" para uma senha fixa
  - em todas as maquinas instalar ubuntu-desktop
  - em todas as maquinas instalar os drivers essenciais do ubuntu (nvidia ja incluso)
  - em todas as maquinas alterar o /etc/containerd/config.toml para usar Nvidia GPUs
  - em todas as maquinas instalar nfs-server
  - na maquina com o HD principal configurar o /etc/exports como mostrado em /etc/exports
  - nas outras maquinas (sem o HD principal) criar o diretorio dataplanet e configurar /etc/fstab incluindo o nfs


### Data science stack
  - instalar helm na maquina administradora
  - instalar metallb
  - instalar kubeflow usando os pv especificos que temos
  - instalar o pv de analine padrao (dataplanet-pv) que sempre esta montado em todo pod de AI
