+--------------------------------------------------------------------+
|Professor Luiz Antonio de Oliveira				     |
|								     |
|Atividade retirada do Livro Laboratórios de Tecnologias CISCO       |
+--------------------------------------------------------------------+



Leia atentamente todo o tutorial!



Nesta simulação será possível realizar a configuração de telefonia IP integrada a rede de dados por meio de solução proprietária CISCO.
Para isto, o router terá de ser o 2811 ISR, este modelo possui o Call Manager Express (CME) da CISCO e tem a função de torna o roteador um PABX.
Observe que na toplogia, os computadores não são conectados diretamente ao switch, mas diretamente ao telefone Ip da CISCO, pois eles tem três portas, sendo:

- uma porta logica de voz;
- uma porta física para conexão do cabo ao switch da LAN;
- uma porta física para conexão do cabo ao computador.

Desta forma temos uma economia de recurso de rede.

No switch escolhido, a tecnologia PoE (Power over Ethernet) já está ativada, ou seja, ele é capaz de alimentar os telefones usando o próprio cabo de rede. 

Observação!!! O desafio incial aqui será fazer os telefones funcionarem após desativar o recurso de PoE no roteiro 15.1

Outra coisa que faremos é a criação de duas vlans, porque temos dois serviços distintos.

vlan 100 para dados
vlan 200 para voz

Antes de mais nada, faça o comando abaixo para desativar a resolução de nomes:

no ip domain lookup



==============================================
Roteiro 15.1 - SW-PoE - Comando de desativação do PoE -----> só faça este roteiro se seus telefones ips estiverem alimentados por fonte externa!!!!

enable
conf t
interface range f 0/1 - 24
power inline never
end



==============================================
Roteiro 15.2 - SW-PoE - Comando de reativação do PoE -----> por padrão o switch 3560 já vem com a funcionalidade de PoE habilitada por padrão.

enable
conf t
interface range f 0/1 - 24
power inline auto
end


*caso você tenha dúvidas sobre o que é a tecnologia PoE, pesquise agora na Internet.



==============================================
Roteiro 15.3 - SW-PoE - Configuração das VLANs de dados e voz no switch e respectivas associações de portas as interfaces
enable
conf t
hostname SW-PoE
vlan 100
name VLAN-Dados
vlan 200
name VLAN-VoIP
end

------->int f 0/24 será o link de trunk, será o encapsulamento dot1q
conf t
interface f 0/24
switchport trunk encapsulation dot1q
switchport mode trunk

------->serão as interfaces associadas a vlan 100 (dados) e simultaneamente com a vlan auxiliar de voz (200)
interface range f 0/1 - 3
switchport mode access
switchport access vlan 100
switchport voice vlan 200
end




Neste modelo de switch 3560 é possível usar uma vlan auxiliar de voz e o próprio aparelho vai tratar essa vlan com prioridade, isso é ótimo, pois aplicações em tempo real do tipo uma ligação telefônica não se pode ter um atraso (latência) maior de 400ms por questões de QoS. Uma rede de qualidade é de até 150ms.



==============================================
Roteiro 15.4 - Configuração do trunk (VLANs) no roteador ISR. -----> aqui estamos criando os "gateways" de cada vlan.

enable
conf t
hostname ISR
interface f 0/0
no shut
interface f 0/0.100
encapsulation dot1q 100
ip address 192.168.100.254 255.255.255.0
interface f 0/0.200
encapsulation dot1q 200
ip address 192.168.200.254 255.255.255.0
end


Dentro da interface física, foram criadas duas interfaces lógicas,  subinterface f0/0.100 e subinterface f0/0.200.
Os ips escolhidos para gateway foram os de final .254, por mera organização. (poderia ter sido outro)



==============================================
Roteiro 15.5 - Configuração do DHCP dos computadores no roteador ISR
enable
conf t
ip dhcp excluded-address 192.168.100.254
ip dhcp pool DHCP-Dados
network 192.168.100.0 255.255.255.0
default-router 192.168.100.254
end



==============================================
Roteiro 15.6 - Configuração do DHCP dos telefones  IP no roteador ISR
enable
conf t
ip dhcp excluded-address 192.168.200.254
ip dhcp pool DHCP-VoIP
network 192.168.200.0 255.255.255.0
default-router 192.168.200.254
option 150 ip 192.168.200.254
end



*campo option do quadro dhcp é utilizado para indicar um servidor tftp, que será uma máquina qualquer ou será uma aplicação no próprio roteador.

*existem várias opções, mas a option 150 é a que diz que o ip 192.168.200.254 é onde os telefones irão buscar as configurações.

MUITO IMPORTANTE CONFERIR SE OS PCS ASSUMIRAM O IP DO ESCOPO 192.168.100.0 E OS TELEFONES ASSUMIRAM O IP DO ESCOPO 192.168.200.0



==============================================
Roteiro 15.7 - Configuração do CME (Call Manager Express) no roteador ISR
enable
conf t
telephony-service
max-dn 10
max-ephones 10
ip source-address 192.168.200.254 port 2000
auto-reg-ephone
end




*esta é a configuração efetivamente do PBXIP, primeiro foi configurado o máximo de telefones ips que teremos na rede ( max-dn 10), depois o máximo de diretórios (10)
*no simulador, só o router 221 tem suporte a isso
*após digitar o comando "auto-reg-ephone", sem aspas, aguarde um pouco e verifique no roteador ISR se os telefones se registraram.



==============================================
Roteiro 15.8 - Agora vamos verificar no roteador quem é cada ephone de acordo com o mac address através do comando:
#show ephone

>>>>procure ephone-1, verifique o mac (anote no seu cenário)
>>>>verifique se o ephone 1 está com algum número de "dn", ou se está vazio.

faça o mesmo para ephone 2 e 3.



veja na lista abaixo a relação entre ephone e número de ramal:
ephone 1 - 54001
ephone 2 - 54002
ephone 3 - 54003




==============================================
Roteiro 15.9 - Configuração dos ramais VoIP no roteador ISR

enable
conf t
ephone-dn 1
number 54001
ephone-dn 2
number 54002
ephone-dn 3
number 54003
end
conf t
ephone 1
button 1:1
ephone 2
button 1:2
ephone 3
button 1:3
end







*Sendo, exemplo:

ephone 1
button 1:1

1 -> o primeiro número é o perfil (como perfis de celular), 1 é padrão;
: -> é o tipo do toque, quer dizer 1 toque longo é chamada externa;
1 -> o segundo número é o ramal ao perfil 1, com esse toque no telefone 1





+------------------------------------------------------------------------------------------------------+
														
Chegou o grande momento do teste                                                                       

Você deverá abrir a interface gráfica do telefone e realizar chamadas para os outros ramais.            
Se vc conseguiu configurar tudo corretamenete, irá conseguir tirar o telefone do gancho ao clicar sobre ele.            

Caso tenha funcionado corretamente, você deverá inserir mais um ramal, pois o funcionário Frederico foi contratado.       

Boa sorte!                                                                                                                
+------------------------------------------------------------------------------------------------------+


