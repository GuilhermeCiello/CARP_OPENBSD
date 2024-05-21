# CARP_OPENBSD
O OpenBSD é um sistema operacional gratuito e multiplataforma que tem sua base em UNIX. Suas maiores características são segurança, padronização e consistência. A garantia de sua segurança se dá ao código fonte, que é escrito e feito por profissionais de segurança, tendo como principal medida segura a criptografia implementada dentro do próprio sistema operacional. 


Firewall de CARP

A configuração de um failover de firewall usando o CARP é importante por inúmeras razões:

Minimiza o Tempo de Inatividade: Em caso de falha de um firewall principal, o CARP permite que um firewall secundário assuma automaticamente, garantindo que a rede continue operando sem interrupções significativas.

Eliminação de Ponto Único de Falha: Ao ter múltiplos firewalls configurados com CARP, você elimina o risco de ter um único ponto de falha na rede.

Facilidade de Manutenção: Firewalls secundários podem ser atualizados ou mantidos sem causar interrupções no serviço, já que o CARP gerencia automaticamente a mudança de um firewall para outro.

Este tutorial tem como objetivo demonstrar a configuração de um failover de firewall utilizando o OpenBSD e o CARP (Common Address Redundancy Protocol).

Ambas as VM's possuem 3 placas de redes virtuais: em0, em1, e em2, com duas interfaces virtuais do CARP.

Interfaces físicas

em0: Esta interface é responsável pela comunicação com a internet, possui um IP real e seu principal propósito é transmitir pacotes de anúncios CARP (Carpv2) para outros firewalls CARP na rede externa. Este IP não aceita conexões para serviços.

em1: Esta interface gerencia a comunicação dentro da LAN interna, possui um IP real e é usada para transmitir pacotes de anúncios CARP para outros firewalls na LAN

em2: Esta é a interface de sincronização CARP, usada para sincronizar o estado entre o firewall MASTER (fw0) e os firewall BACKUP (fw1).

Interfaces virtuais
Para o seu funcionamento, o CARP necessita estar vinculado a uma interface de rede física pois é por meio delas que os pacotes CARP são transmitidos e recebidos.

carp1: vinculada a placa de rede em0. Possui o IP público 50.50.50.11, utilizado para responder aos serviços públicos hospedados. Caso o firewall MASTER falhe, este IP é transmitido de forma automática ao BACKUP.

carp2: vinculada a placa de rede em1. Possui o IP interno 192.198.10.11 IP é usado como a rota padrão pelos clientes da LAN para NAT e acesso à Internet. 
Funcionamento

Teremos a configuração de 2 firewalls "OpenBSD_fw0" e "OpenBSD_fw1", o fw0 será o "master", ou seja, aquele firewall que está ativo e funcionando corretamente. fw1 será o "Backup", ele fica inativo sendo usado somente se o firewall "master" cair.

Os testes foram realizados em máquinas virtuais com a versão 6.6 do OpenBSD, disponível para download na página oficial do sistema. https://www.openbsd.org/faq/faq4.html#Download

Após fazer o download da ISO do OpenBSD, siga os passos abaixo para criação das máquinas virtuais, "OpenBSD_fw0" e "OpenBSD_fw1":

Configuração da Máquina Virtual:
Nome: OpenBSD_fw0 e OpenBSD_fw2
Memória Base: 4 GB
Disco Rígido Virtual: 8 GB
Rede: Adaptador 1: conectado a NAT
Adaptador 2: conectado a rede interna(internal_net)
Adaptador 3: conectado a rede interna(pfsync)

Comandos usados na máquina virtual "OpenBSD_fw0"
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
up syncdev em2
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
up syncdev em2
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
```
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
