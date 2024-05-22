# Failover de Firewall com CARP
O OpenBSD é um sistema operacional baseado em UNIX, conhecido por sua segurança, padronização e consistência. Uma de suas características distintivas é o uso do Common Address Redundancy Protocol (CARP) para configuração de failover de firewall, garantindo alta disponibilidade e segurança.

O protocolo CARP, sigla para Common Address Redundancy Protocol serve para aumentar a disponibilidade de um serviço ao compartilhar um endereço IP único entre vários servidores em um mesmo segmento de rede. Pode ser usado em servidores, roteadores e  firewalls.

**Firewall de Failover com CARP**

O CARP permite a configuração de failover de firewall para minimizar o tempo de inatividade, eliminar pontos únicos de falha e facilitar a manutenção. Este tutorial demonstrará como configurar um firewall de failover usando o OpenBSD e o CARP.

**Configuração das Interfaces**

As máquinas virtuais (VMs) do OpenBSD possuem três placas de rede virtuais (em0, em1 e em2) e duas interfaces virtuais CARP.

**Interfaces físicas:**

- em0: Comunicação com a internet, utilizada para transmitir pacotes CARP para firewalls na rede externa.
- em1: Comunicação dentro da LAN interna, usada para pacotes CARP entre firewalls na LAN.
- em2: Interface de sincronização CARP para manter o estado entre o firewall MASTER e BACKUP.

**Interfaces virtuais:**

- carp1: Vinculada à placa de rede em0, com IP público 50.50.50.11 para responder a serviços públicos.
- carp2: Vinculada à placa de rede em1, com IP interno 192.198.10.11 para rota padrão dos clientes da LAN.

**Funcionamento**

Configuração de dois firewalls: "OpenBSD_fw0" como master e "OpenBSD_fw1" como backup. O fw0 está ativo, enquanto o fw1 fica inativo, acionado apenas se o master falhar.
Para sincronizar estados de conexão de múltiplos firewalls que utilizam o Packet Filter (PF) é necessário a utilização da interface virtual pfsync0
**Passos para Configuração**

1. Faça o download da ISO do OpenBSD 6.6 na página oficial: [OpenBSD Download](https://www.openbsd.org/faq/faq4.html#Download).

2. Crie as VMs "OpenBSD_fw0" e "OpenBSD_fw1" com as seguintes configurações:
   - Nome: OpenBSD_fw0 e OpenBSD_fw2
   - Memória Base: 4 GB
   - Disco Rígido Virtual: 8 GB
   - Rede:
     - Adaptador 1: Conectado à NAT
     - Adaptador 2: Conectado à rede interna (internal_net)
     - Adaptador 3: Conectado à rede interna (pfsync)

3. Configure as interfaces e interfaces virtuais conforme descrito acima.

Comandos na Máquina Virtual "OpenBSD_fw0"

Segue a configuração dos comandos específicos na máquina virtual "OpenBSD_fw0" para implementar o firewall de failover com CARP.

```
openbsdfw0# vi /etc/hostname.em0
inet 50.50.50.10 255.255.255.0 #Configura a interface em0 com o endereço IP 50.50.50.10 e máscara de rede 255.255.255.0
```
```
openbsdfw0# vi /etc/hostname.em1
inet 192.168.10.10 255.255.255.0 #Configura a interface em1 com o endereço IP 192.168.10.10 e máscara de rede 255.255.255.0
```
```
openbsdfw0# vi /etc/hostname.em2
inet 10.20.30.10 255.255.255.0 #Configura a interface em2 com o endereço IP 10.20.30.10 e máscara de rede 255.255.255.0
```
```
openbsdfw0# vi /etc/hostname.carp1
inet 50.50.50.11 255.255.255.0 50.50.50.255 vhid 1 advbase 10 advskew 0 pass carppass carpdev em0

#Configura a interface carp1 com o endereço IP virtual 50.50.50.11
#vhid 1: Identificador Virtual Host ID, unico entre interfaces CARP na mesma rede
#advbase 10: Base de tempo de anúncio em segundos
#advskew 0: Skew do tempo de anúncio, usado para definir a prioridade
#pass carppass: Senha para autenticação CARP
#carpdev em0: Placa em que o CARP esta vinculado
```
```
openbsdfw0# vi /etc/hostname.carp2
inet 192.168.10.11 255.255.255.0 50.50.50.255 vhid 2 advbase 20 advskew 0 pass carppass2 carpdev em1

#Configura a interface carp2 com o endereço IP virtual 192.168.10.11
#vhid 2: Identificador Virtual Host ID para a rede interna
#advbase 20: Base de tempo de anuncio
#advskew 0: Skew do tempo de anuncio
#pass carppass2: Senha para autenticação CARP
#carpdev em1: Placa em que o CARP esta vinculado
```
```
openbsdfw0# vi /etc/hostname.pfsync0
up syncdev em2 # configura a interface pfsync0 para usar uma interface de sincronização dedicada
```
```
openbsdfw0# vi /etc/sysctl.conf
net.inet.carp.allow=1      #Ativa o CARP
net.inet.carp.preempt=1	   #Permite que o BACKUP assuma o papel de MASTER automaticamente.
net.inet.ip.forwarding=1   #Ativa o encaminhamento de pacotes IP
```
```
openbsdfw0# vi /etc/rc.conf.local
pf=YES
pf_rules=/etc/pf.conf
```
```
openbsdfw0# chmod +x /etc/netstart 
```
```
openbsdfw0# /etc/netstart
```
Comandos usados na máquina virtual "OpenBSD_fw1"
```
openbsdfw1# vi /etc/hostname.em0
inet 50.50.50.20 255.255.255.0  #Configura a interface em0 com o endereço IP 50.50.50.20 e máscara de rede 255.255.255.0   
```
```
openbsdfw1# vi /etc/hostname.em1
inet 192.168.10.20 255.255.255.0  #Configura a interface em1 com o endereço IP 192.168.10.20 e máscara de rede 255.255.255.0
```
```
openbsdfw1# vi /etc/hostname.em2
inet 10.20.30.20 255.255.255.0  #Configura a interface em2 com o endereço IP 10.20.30.20 e máscara de rede 255.255.255.0
```
```
openbsdfw1# vi /etc/hostname.carp1
inet 50.50.50.11 255.255.255.0 50.50.50.255 vhid 1 advbase 20 advskew 10 pass carppass carpdev em0

#Configura a interface carp1 com o endereço IP virtual 50.50.50.11
#vhid 1: Identificador Virtual Host ID, unico entre interfaces CARP na mesma rede
#advbase 20: Base de tempo de anúncio em segundos
#advskew 10: Skew do tempo de anúncio, usado para definir a prioridade
#pass carppass: Senha para autenticação CARP
#carpdev em0: Placa em que o CARP esta vinculado
```
```
openbsdfw1# vi /etc/hostname.carp2
inet 192.168.10.11 255.255.255.0 50.50.50.255 vhid 2 advbase 20 advskew 10 pass carppass2 carpdev em1

#Configura a interface carp2 com o endereço IP virtual 192.168.10.11
#vhid 2: Identificador Virtual Host ID para a rede interna
#advbase 20: Base de tempo de anuncio
#advskew 10: Skew do tempo de anuncio
#pass carppass2: Senha para autenticação CARP
#carpdev em1: Placa em que o CARP esta vinculado
```
```
openbsdfw1# vi /etc/hostname.pfsync0
up syncdev em2 # configura a interface pfsync0 para usar uma interface de sincronização dedicada
```
```
openbsdfw1# vi /etc/sysctl.conf
net.inet.carp.allow=1      #Ativa o CARP
net.inet.carp.preempt=1	   #Permite que o BACKUP assuma o papel de MASTER automaticamente.
net.inet.ip.forwarding=1   #Ativa o encaminhamento de pacotes IP
```
```
openbsdfw1# vi /etc/rc.conf.local
pf=YES
pf_rules=/etc/pf.conf
```
```
openbsdfw1# chmod +x /etc/netstart
```
```
openbsdfw1# /etc/netstart
```

Criação das regras no Packet Filter(PF)
Estas regras são configuradas nas duas VM's 
```
vi /etc/pf.conf

# Definição de Variáveis e Interfaces
ExtIf="em0"
IntIf="em1"
CarpSyncIf="em2"
CarpExt="carp1"
CarpInt="carp2"
ExtIP="50.50.50.10"
IntIP="192.168.10.10"
SyncIP="10.20.30.10"
CarpExtIP="50.50.50.11"
CarpIntIP="192.168.10.11"

# Filtragem de Pacotes
pass log on $CarpSyncIf inet proto pfsync keep state
pass log on { $ExtIf, $IntIf } inet proto carp from { $ExtIP, $IntIP } to any keep state
block in quick on { $ExtIf, $IntIf } from any to $CarpExtIP
block in quick on { $ExtIf, $IntIf } from any to $CarpIntIP
```
