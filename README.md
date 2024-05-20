# CARP_OPENBSD
O OpenBSD é um sistema operacional gratuito e multiplataforma que tem sua base em UNIX. Suas maiores características são segurança, padronização e consistência. A garantia de sua segurança se dá ao código fonte, que é escrito e feito por profissionais de segurança, tendo como principal medida segura a criptografia implementada dentro do próprio sistema operacional. 


CARP no OpenBSD
Este tutorial tem como objetivo demonstrar a configuração de um failover de firewall utilizando o OpenBSD e o CARP (Common Address Redundancy Protocol).

Os testes foram realizados em máquinas virtuais com a versão 6.6 do OpenBSD, disponível para download na página oficial do sistema. https://www.openbsd.org/faq/faq4.html#Download

Após fazer o download da ISO do OpenBSD, siga os passos abaixo para criação das máquinas virtuais, "OpenBSD_fw0" e "OpenBSD_fw1":

Configuração da Máquina Virtual:
Nome: OpenBSD_fw0 e OpenBSD_fw2
Memória Base: 4 GB
Disco Rígido Virtual: 8 GB
Rede: Adaptador 1: conectado a NAT
Adaptador 2: conectado a rede interna(internal_net)
Adaptador 3: conectado a rede interna(pfsync)
