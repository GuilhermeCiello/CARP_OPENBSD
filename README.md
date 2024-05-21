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


