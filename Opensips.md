==Ambiente Base==
Debian 6.0.3 com kernel 2.6.32

feito a instalação basica com o cd netinst e selecionado pacotes basicos e servidor ssh, nada além.

===Pacotes Basicos===

<pre>
apt-get install gcc make libncurses5-dev libxml2-dev  linux-headers-`uname -r` vim postfix libmysqlclient-dev bison flex \
libmemcached-dev memcached libpcre3-dev libpcre++-dev libxmlrpc-c3-dev  libdbi-perl libdbd-mysql-perl mysql-server ngrep \
git apache2-mpm-prefork libapache2-mod-php5 php5-mysql php5 php-cli php5-cgi php5-gd php5-xmlrpc php-pear
</pre>


O servidor mysql irá lhe perguntar a senha, lembre-se desta senha pois utilizaremos mais tarde

==Download do Opensips==
Vou passar o link direto pois é a versão exata que estou trabalhando, isso evita falhas no tutorial.


Vamos armazenar os fontes em /usr/src

 cd /usr/src/

 wget -c http://downloads.sourceforge.net/project/opensips/OpenSIPS/1.7.1/opensips-1.7.1_src.tar.gz?r=http%3A%2F%2Fsourceforge.net%2Fprojects%2Fopensips%2Ffiles%2F&ts=1327159345&use_mirror=ufpr


  mv opensips-1.7.1_src.tar.gz\?r\=http\:%2F%2Fsourceforge.net%2Fprojects%2Fopensips%2Ffiles%2F opensips-1.7.1_src.tar.gz

 tar -xzvf opensips-1.7.1_src.tar.gz

 cd opensips-1.7.1-tls



==Configuracao do Makefile==

Abra o arquivo Makefile e encontre a linha abaixo (logo no inicio [ Linha 53 no meu caso ])

 exclude_modules?= b2b_logic jabber cpl-c xmpp rls mi_xmlrpc xcap_client \

As opções continuam algumas linhas para baixo, então nós vamos remover desta lista os seguintes modulos

<pre>
mi_xmlrpc 
db_mysql
presence
presence_xml
presence_mwi
memcached
dialplan
</pre>

Nós não vamos entrar no merito de presença, mas ja vai ficar compilado pois é um recurso interessante , se você quiser ativa-lo depois basta
ver a documentação do opensips.


==Iniciando a Compilacao==
 make all
 make install
 mkdir /var/run/opensips
 groupadd opensips
 useradd -d /var/run/opensips -s /bin/true -g opensips opensips
 chown -R opensips.opensips /var/run/opensips
 cp /usr/src/opensips-1.7.1-tls/packaging/debian/opensips.default /etc/default/opensips
 cp /usr/src/opensips-1.7.1-tls/packaging/debian/opensips.init /etc/default/opensips
 chmod +x /etc/init.d/opensips
 ln -s /usr/local/etc/opensips /etc/



===Configuracao do script de inicializacao===
Abra o arquivo /etc/default/opensips

Altere o parametro RUN_OPENSIPS para yes


Abra o arquivo /etc/init.d/opensips

Altere a variavel PATH incluindo 
 :/usr/local/bin:/usr/local/sbin

Altere a variavel DAEMON para
 /usr/local/sbin/opensips


Agora vamos colocar com o comando abaixo para iniciar durante o boot
 update-rc.d opensips defaults

==Iniciando o opensips==

 /etc/init.d/opensips start

Ele deverá mostrar uma mensagem similar a abaixo

<pre>
Starting opensips: opensipsListening on 
             udp: 127.0.0.1 [127.0.0.1]:5060
             udp: 192.168.0.30 [192.168.0.30]:5060
             tcp: 127.0.0.1 [127.0.0.1]:5060
             tcp: 192.168.0.30 [192.168.0.30]:5060
Aliases: 
             tcp: localhost:5060
             udp: localhost:5060

</pre>


Por padrão o opensips irá escutar em todos os ips da maquina


Você pode olhar no log para ver se tem algum erro (neste momento não deve ter)

 tail -n 50 /var/log/syslog  | grep -i opensips


==Fazendo um teste Básico==

Na configuração padrão do OpenSIPS após a compilação e instalação o arquivo de configuração vem de 
maneira funcional ( sem nat e outras opções é claro ) para que possa ser testado, não é necessário criar
nenhum usuário, basta colocar no softphone o usuario 1000 de um lado e 1001 do outro (não precisa senha)

Para verificar o status via linha de comando, utilize o comando abaixo
 opensipsctl ul show

O Resultado deverá ser similar ao abaixo
<pre>
Domain:: location table=512 records=2
	AOR:: 1000
		Contact:: sip:1000@192.168.0.13 Q=1
			Expires:: 3475
			Callid:: 561e9dab-b842-e111-84ed-001d7d88a1ef@linux-5pfa
			Cseq:: 7
			User-agent:: Ekiga/3.2.7
			State:: CS_NEW
			Flags:: 0
			Cflag:: 0
			Socket:: udp:192.168.0.30:5060
			Methods:: 22335
	AOR:: 1001
		Contact:: sip:1001@192.168.0.11 Q=1
			Expires:: 3430
			Callid:: 567871a5-b842-e111-9c21-002268c35676@linux-f433.site
			Cseq:: 3
			User-agent:: Ekiga/3.2.7
			State:: CS_NEW
			Flags:: 0
			Cflag:: 0
			Socket:: udp:192.168.0.30:5060
			Methods:: 22335
</pre>

Até aqui não devemos ter nenhum problema, então vamos começar a montar um modelo de configuração, eu não 
vou simplesmente colocar o arquivo de configuração aqui pois desta maneira você poderá acompanhar a linha de
raciocinio para a configuração do mesmo 



==Preparando o Banco de Dados para o OpenSIPs==

Nós vamos utilizar banco de dados no opensips pois isso nos permite uma maior liberdade para criação de ferramentas, 
além é claro de utilizarmos a opensips-cp para o gerenciamento básico.

===Arquivo de configuracao opensipsctlrc===

Abra o arquivo /etc/opensips/opensipsctlrc

Na linha 10 descomente a variavel SIP_DOMAIN e altere o valor para opensips.org
 --- Ou altere para o nome de dns que sua maquina tem, e nas proximas opções que utilizarmos este nome 
 --- de dominio você altera para o seu nome de maquina

Na linha 19 descomente a variavel DBENGINE=mysql

Na linha 22 descomente a variavel DBHOST=localhost

Na linha 25 descomente a variavel DBNAME=opensips

Na linha 31 descomente a variavel DBRWUSER=opensips
 --- O opensips possui dois usuarios, um usuario ro e outro rw, deste modo voce pode gerar permissoes diferenciadas e utilizar 
 --- bancos de dados diferentes para cada operacao

Na linha 34 descomente a variavel DBRWPW=opensipsrw

Na linha 37 descomente a variavel DBROOTUSER=root

Na linha 75 descomente a variavel INSTALL_EXTRA_TABLE e altere o valor para yes

Na linha 78 descomente a variavel INSTALL_PRESENCE_TABLE e altere o valor para yes

Na linha 96 descomente a variavel ALIASES_TYPE=DB

Na linha 112 descomente a variavel VERIFY_ACL=1

Na linha 116 descomente a variavel ACL_GROUPS
 --- Aqui sao os grupos que iremos utilizar para ACL's, apenas estes serão aceitos

Na linha 134 descomente a variavel STORE_PLAINTEXT_PW=1


=== Criando o banco de dados===

 opensipsctldb create opensips


Para testar a conectividade utilize a linha abaixo.
 mysql -uopensips -popensipsrw opensips

Para listar as tabelas
 show tables;

Para sair
 quit;


 


==Habilitando Autenticação==

O arquivo de configuração do opensips no nosso caso é o /etc/opensips/opensips.cfg (é um link, o arquivo esta em /usr/local/etc/opensips/opensips.cfg)

Abra este arquivo


Na linha 15 altere o valor de debug para 6

Na linha 71 descomente a linha loadmodule "db_mysql.so"

Na linha 85 descomente a linha auth_module e a de baixo auth_module_db

Descomente na linha 89 loadmoule "alias_db.so"


comente a linha 119
 modparam("usrloc", "db_mode",   0)


e Descomente as linhas 122 , 123, 124
<pre>
modparam("usrloc", "db_mode",   2)
modparam("usrloc", "db_url",
        "mysql://opensips:opensipsrw@localhost/opensips")

</pre>

Descomente das linhas 151 a 155
<pre>
modparam("auth_db", "calculate_ha1", yes)
modparam("auth_db", "password_column", "password")
modparam("auth_db", "db_url",
        "mysql://opensips:opensipsrw@localhost/opensips")
modparam("auth_db", "load_credentials", "")
</pre>

Descomente das linhas 161 a 162

<pre>
modparam("alias_db", "db_url",
        "mysql://opensips:opensipsrw@localhost/opensips")
</pre>


Descomente a linha 254
        if (!(method=="REGISTER") && from_uri==myself) /*no multidomain version*/

Descomente da linha 256 a 268

<pre>
        {
                if (!proxy_authorize("", "subscriber")) {
                        proxy_challenge("", "0");
                        exit;
                }
                if (!db_check_from()) {
                        sl_send_reply("403","Forbidden auth ID");
                        exit;
                }
        
                consume_credentials();
                # caller authenticated
        }

</pre>


Descomente da linha 321 a 331
<pre>
                if (!www_authorize("", "subscriber"))
                {
                        www_challenge("", "0");
                        exit;
                }
                
                if (!db_check_to())  
                {
                        sl_send_reply("403","Forbidden auth ID");
                        exit;
                }


</pre>

Descomente a linha 346
        alias_db_lookup("dbaliases");



Agora salve o arquivo


Reinicie o opensips
 /etc/init.d/opensips restart


tente se conectar novamente com os usuários que utilizamos anteriormente 1000 e 1001, você verá que o mesmo não conseguirá se 
registrar no servidor, isso porque os usuários não existem e nós habilitamos a autenticação.

===Criando usuários===

Para podermos testar a autenticação teremos que criar dois usuários, utilize o comando abaixo.
 opensipsctl add 1001 teste
 opensipsctl add 1000 teste

Para listar os usuarios que estão no banco utilize o comando abaixo.

 opensipsctl db show subscriber

Tente fazer a conexão agora, não deverá haver problemas



====Acompanhando o processo====

Uma ferramenta muito util para quem usa o opensips (ou qualquer outro software sip) é o ngrep, com ele você consegue acompanhar as mensagens
que são trocadas na rede, execute o comando abaixo, e depois mande a conta se registrar


 ngrep -p -q -W byline port 5060


O resultado será similar ao abaixo.

<pre>


#### MANDANDO O PEDIDO DE REGISTRO

U 192.168.0.11:5060 -> 192.168.0.30:5060
REGISTER sip:192.168.0.30 SIP/2.0.
CSeq: 7 REGISTER.
Via: SIP/2.0/UDP 192.168.0.11:5060;branch=z9hG4bK1c20e8dd-c342-e111-8b80-002268c35676;rport.
User-Agent: Ekiga/3.2.7.
From: <sip:1000@192.168.0.30>;tag=da9ee7dd-c342-e111-8b80-002268c35676.
Call-ID: 8c6fe7dd-c342-e111-8b80-002268c35676@linux-f433.site.
To: <sip:1000@192.168.0.30>.
Contact: <sip:1000@192.168.0.11>;q=1.
Allow: INVITE,ACK,OPTIONS,BYE,CANCEL,SUBSCRIBE,NOTIFY,REFER,MESSAGE,INFO,PING.
Expires: 3600.
Content-Length: 0.
Max-Forwards: 70.
.



#### RESPONDENDO COM NAO AUTORIZADO E SOLICITANDO AS CREDENCIAIS

U 192.168.0.30:5060 -> 192.168.0.11:5060
SIP/2.0 401 Unauthorized.
CSeq: 7 REGISTER.
Via: SIP/2.0/UDP 192.168.0.11:5060;received=192.168.0.11;branch=z9hG4bK1c20e8dd-c342-e111-8b80-002268c35676;rport=5060.
From: <sip:1000@192.168.0.30>;tag=da9ee7dd-c342-e111-8b80-002268c35676.
Call-ID: 8c6fe7dd-c342-e111-8b80-002268c35676@linux-f433.site.
To: <sip:1000@192.168.0.30>;tag=c97b4d1cb1f3d0da549e06a8d482ef63.c0b5.
WWW-Authenticate: Digest realm="192.168.0.30", nonce="4f1af72c00000005440079e3856dbfecc5eb471cd6657ad8".
Server: OpenSIPS (1.7.1-notls (i386/linux)).
Content-Length: 0.
.


#### NOVO PEDIDO COM AS CREDENCIAS (veja em negrito)

U 192.168.0.11:5060 -> 192.168.0.30:5060
REGISTER sip:192.168.0.30 SIP/2.0.
CSeq: 8 REGISTER.
Via: SIP/2.0/UDP 192.168.0.11:5060;branch=z9hG4bKfca8eddd-c342-e111-8b80-002268c35676;rport.
User-Agent: Ekiga/3.2.7.
</pre>
'''Authorization: Digest username="1000", realm="192.168.0.30", nonce="4f1af72c00000005440079e3856dbfecc5eb471cd6657ad8", uri="sip:192.168.0.30", algorithm=MD5, response="9c52b428e1cbe25355a7ea15cda45763".'''
<pre>
From: <sip:1000@192.168.0.30>;tag=da9ee7dd-c342-e111-8b80-002268c35676.
Call-ID: 8c6fe7dd-c342-e111-8b80-002268c35676@linux-f433.site.
To: <sip:1000@192.168.0.30>.
Contact: <sip:1000@192.168.0.11>;q=1.
Allow: INVITE,ACK,OPTIONS,BYE,CANCEL,SUBSCRIBE,NOTIFY,REFER,MESSAGE,INFO,PING.
Expires: 3600.
Content-Length: 0.
Max-Forwards: 70.
.


#### CONFIRMANDO O PEDIDO DE REGISTRO

U 192.168.0.30:5060 -> 192.168.0.11:5060
SIP/2.0 200 OK.
CSeq: 8 REGISTER.
Via: SIP/2.0/UDP 192.168.0.11:5060;received=192.168.0.11;branch=z9hG4bKfca8eddd-c342-e111-8b80-002268c35676;rport=5060.
From: <sip:1000@192.168.0.30>;tag=da9ee7dd-c342-e111-8b80-002268c35676.
Call-ID: 8c6fe7dd-c342-e111-8b80-002268c35676@linux-f433.site.
To: <sip:1000@192.168.0.30>;tag=c97b4d1cb1f3d0da549e06a8d482ef63.7993.
Contact: <sip:1000@192.168.0.11>;q=1;expires=3600.
Server: OpenSIPS (1.7.1-notls (i386/linux)).
Content-Length: 0.
.

</pre>

Podem ter outras mensagens dependendo do softphone pois ele pode enviar pedido de subscriber e outras coisas que são pra presença voicemail e etc..




===Criando um Alias===

Vamos agora criar um alias para uma das contas, este alias pode ser um numero de telefone, ou um nome
 opensipsctl alias_db add 1135444444@opensips.org 1000@opensips.org

Para visualizar os alias execute o comando abaixo
 opensipsctl alias_db list 1000@opensips.org

A saida deverá ser similar a abaixo

<pre>

Dumping aliases for user=<1000@opensips.org>

+------------------------+
| ALIAS                  |
+------------------------+
| 1135444444@opensip.org |
+------------------------+

</pre>


Agora pegue o softphone que esta com o ramal 1001 e ligue para 1135444444

'''IMPORTANTE o opensips se baseia nos cabeçalhos que chegam pra ele para verificar estas questões, se você não esta utilizando um dns e o 
dominio neste dns então tudo funcionará bem, do contrario voce tem que criar um alias com o ip do servidor (no meu caso é 192.168.0.30)'''
  opensipsctl alias_db add 1135444444@192.168.0.30 1000@192.168.0.30


Ao discar para o número em questão o sistema verifica a tabela alias e redirecionará para o usuario 1000


==Ativando o NATHELP==

O Nathelp é um aplicativo que fará por nós o trabalho de auxilio para travessia de nat, não vou aprofundar muito sobre o funcionamento do 
mesmo, mas com um pouco de leitura na documentação você pode entender bem, lembre-se que hoje é bastante utilizado rede com nat, então é 
um módulo essencial.

===Instalando o rtpproxy===

Após as verificações que o nathelper fizer será necessário que tenhamos um proxy de rtp 
para fazer a travessia do audio, usaremos o rtpproxy para isso, existe o mediaproxy também, 
mas para este manual o rtpproxy atenderá melhor, apenas como observação, o rtpproxy não precisa
estar na mesma maquina, aqui estamos instalando tudo junto, mas voce pode ter servidores diferentes 
para cada operação visando uma grantia de cada parte do serviço.

 cd /usr/src/
 wget -c http://b2bua.org/chrome/site/rtpproxy-1.2.1.tar.gz
 cd rtpproxy-1.2.1
 ./configure
 make
 make install
 groupadd rtpproxy
 useradd -d /var/run/rtpproxy -s /bin/true -g rtpproxy rtpproxy
 mkdir /var/log/rtpproxy
 chown -R rtpproxy.rtpproxy /var/log/rtpproxy
 chown -R rtpproxy.rtpproxy /var/run/rtpproxy

Agora precisamos criar um script de inicializacao

 cd /etc/init.d/
 vi rtpproxy

Coloque o seguinte conteudo no arquivo
<pre>
#!/bin/bash
#
### BEGIN INIT INFO
# Provides:          rtpproxy
# Required-Start:    $syslog $network $local_fs $time
# Required-Stop:     $syslog $network $local_fs
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: Start the RTPPROXY server
# Description:       Start the RTPPROXY server
### END INIT INFO

PATH=/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/bin:/sbin
USELOG=1
USER=rtpproxy
IPADDR="192.168.0.30"

. /lib/lsb/init-functions


start(){
        echo "Iniciando RTP PROXY "
        if [ -z $(pidof rtpproxy) ]; then
                if [ "${USELOG}" = "1" ]; then
                        echo "Iniciando com LOG"
                        /usr/local/bin/rtpproxy -l $IPADDR -s udp:127.0.0.1:7890 -u $USER -F -f d DBUG 2&> /var/log/rtpproxy/rtpproxy.log &
                else
                        echo "Iniciando sem LOG"
                        /usr/local/bin/rtpproxy -l $IPADDR -s udp:127.0.0.1:7890 -u $USER  -F -f d DBUG 2&> /dev/null
                fi

                if [ -n $(pidof rtpproxy) ]; then
                        echo "START OK"
                fi
        else
                echo "Processo ja em execucao"
        fi
}


stop(){

        if [ -z $(pidof rtpproxy) ]; then
                echo "Processo nao encontrado"
        else
                kill -9 $(pidof rtpproxy)
                if [ -n $(pidof rtpproxy) ]; then
                        echo "STOP OK"
                else
                        echo "Falha em realizar stop do servico"
                fi

        fi
}

case $1 in
        start)
                start
        ;;
        stop)
                stop
        ;;
        restart)
                stop
                start

        ;;
        *)
                echo "Utilize: stop | start | restart"
        ;;
esac

</pre>

Alteramos a permissao
 chmod +x /etc/init.d/rtpproxy

Agora adicione a inicializacao com o comando abaixo

 update-rc.d rtpproxy defaults

Para ficar na sequencia correta precisamos remover o opensips da inicializacao 
 update-rc.d opensips remove 

E alterar o arquivo /etc/init.d/opensips para iniciar depois do rtpproxy
 # Required-Start:    $mysql $rtpproxy $syslog $network $local_fs $time

Altere também na linha 107 (abaixo de:   start|debug)
 sleeep 5

Esta alteração é porque o mysql do debian entra em background, então é pra dar um tempo pra ele subir
e depois o opensips entrar, do contrario ele nao vai iniciar pois não conseguirá se conectar ao banco

Agora colocamos na inicializacao novamente

 update-rc.d opensips defaults


Se quiser pode reinicar a maquina para garantir que tudo irá subir na sequencia correta


Agora vamos editar o arquivo /etc/opensips/opensips.cfg para ativar o nat e o rtpproxy

Va na linha 100 do arquivo e adicione
 /* Modulos Adicionais */

E na linha 102 adicione
 /* fim dos modulos adicionais */


Agora entre elas coloque

 loadmodule "nathelper.so"
 loadmodule "avpops.so"
 loadmodule "rtpproxy.so"


Agora antes da linha 
 ####### Routing Logic ########

(no meu caso é 194) Adicione o seguintes parametros

<pre>
/* parametros nathelper */


modparam("usrloc", "nat_bflag", 6)
modparam("registrar", "received_avp", "$avp(42)")
modparam("nathelper", "received_avp", "$avp(42)")
modparam("nathelper", "natping_interval", 30)
modparam("nathelper", "ping_nated_only", 1)
modparam("nathelper", "sipping_bflag", 7)
modparam("nathelper", "sipping_from", "sip:pinger@192.168.0.30")
modparam("rtpproxy", "rtpproxy_sock", "udp:127.0.0.1:7890")
</pre>


Agora vamos iniciar as alterações na nossa logica de processamento, aqui eu ja não consigo referenciar exatamente as linhas
então vou tratar do conteudo anterior e você entra com o novo trecho logo abaixo.


A primeira alteração será na nossa main route, logo abaixo do inicio

 route{

Esta alteração é valida pois servirá para voce bloquear um perfil de usuario (scanner) e pode usar para outras funções
<pre>
        if($ua =~ "friendly-scanner"){
                # estamos identificando o tipo de ataque baseado no user agente e saindo sem resposta
                # neste caso a tendencia é que ele para de tentar
                xlog("L_NOTICE", "Auth error for $fU@$fd from $si cause -1");
                xlog("FRIENDLY-SCANNER: UA: $ua From_TAG: $ft From_URI: $fu Received IP: $Ri IP SOURCE: $si");
                exit;
        }


</pre>

Agora abaixo da configuração seguinte
<pre>
         if (!mf_process_maxfwd_header("10")) {
                sl_send_reply("483","Too Many Hops");
                exit;
        }
 
</pre>
Nós vamos adicioanr uma entrada, que quando recebermos um OPTION enviamos um OK de volta

<pre>
        if(method=="OPTIONS"){
                sl_send_reply("200", "OK");
                exit;
        }

</pre>

Agora vamos iniciar realmente a deteccao de nat, em sequencia do bloco anterior entre com o seguinte.

<pre>
        if(force_rport()){
                xlog("NAT: Force Rport executado");
        }else{
                xlog("NAT: FALHA COM force_rport");
        }


        if(nat_uac_test("18")){
                xlog("NAT: NAT UAC TEST EXECUTADO\n");
                if(method=="REGISTER"){
                        xlog("NAT: REGISTER -> fix_nated\n");
                        fix_nated_register();
                }else{
                        xlog("NAT: NAO REGISTER -> fix_contact\n");
                        fix_nated_contact();
                }
                xlog("NAT: SET FLAG 5");
                setflag(5);
        }else{
                # Normalmente voce so usuaria o setflag(5) para marcar o inicio para
                # o nat, mas ja teve situações de querer marcar tudo, então existe
                # a linha abaixo, use-a se achar viavel
                # setflag(5);
        }


</pre>

Agora localize a linha
         if (is_method("REGISTER"))

Dentro deste bloco no inicio adicione
                if(isflagset(5)){
                        setbflag(6);
                        setbflag(7);

                }

Agora vamos para nossa rota 1, localize
 route[1] {

Logo abaixo dentro do bloco coloque

<pre>
        if(subst_uri('/(sip:.*);nat=yes/\1/')){
                xlog("NAT: SETBFLAG para substituicao de uri");
                setbflag(6);
        }

        if(isflagset(5) || isbflagset(6)){
                xlog("NAT: Direcionando para rota 6");
                route(6);
        }


</pre>

Agora no fim do arquivo vamos criar a rota 6
<pre>
 route[6]{
        if(is_method("BYE|CANCEL")){
                unforce_rtp_proxy();
        }else if(is_method("INVITE")){
                rtpproxy_offer("f");
                t_on_failure("1");
        }

 }
</pre>


Neste caso estamos oferecendo o rtpproxy no invite, entao agora precisamos receber a resposta, vamos para
a rota 
 onreply_route[2]{

Abaixo da linha do xlog adicione
<pre>
        if(isflagset(5) || isbflagset(6) && status =~ "(183)|(2[0-9][0-9])"){
                xlog("NAT: FLAG 5 ou BFLAG 6 e status 183 - 2XX");
                rtpproxy_answer("f");
                append_hf("P-hint: onereply_route|force_rtp_proxy\r\n");
                search_append('Contact:.*sip:[^>[:cntrl:]]*', ';nat=yes');
        }

        if(isbflagset(6)){
                xlog("NAT: BFLAG 6 - fixcontact");
                append_hf("P-hing: onreply-route - fixcontact \r\n");
                fix_nated_contact();
        }

        exit;



</pre>


Ok, reinicie o opensips 
 /etc/init.d/opensips restart


utilize o ngrep para monitorar a chamada, você verá que o audio será fechado no ip do servidor
<pre>
v=0.
o=- 1327175760 1 IN IP4 192.168.0.13.
s=Opal SIP Session.
c=IN IP4 192.168.0.30.
t=0 0.
m=audio 35492 RTP/AVP 110 101 125.
</pre>


se isso não tiver acontecendo com você use aquela opção de setflag(5) que eu deixei comentada informando 
que poderia ser utilizada em outro momento, reinicie o opensips e teste denovo



Ok, neste momento nós temos o opensips autenticando, e falando através de nat com os clientes, agora eu vou avançar um pouco 
mais rapidamente , nós vamos utilizar os modulos dialplan para gerenciamento de plano de discagem por banco, o modulo drouting para escolha
do destino, o modulo siptrace e dialog para visualizar o dialogo sip via interface web e claro o opensips-cp para gerenciar isso tudo


==Habilitando modulos extras==

Abra o arquivo opensips.cfg vá até a marcao de modulos extras que fizemos e adicione

<pre>
loadmodule "group.so"
loadmodule "permissions.so"
loadmodule "drouting.so"
loadmodule "dialplan.so"
loadmodule "dialog.so"
loadmodule "mi_xmlrpc.so"
loadmodule "siptrace.so"
</pre>

Agora antes da route main , adicione.

<pre>
/* parametro para o dialplan (uso posterior) */
modparam("auth_db", "load_credentials", "$avp(rpid)=rpid; $avp(dpid)=dpid; $avp(55)=ha1")



/* parametros dialplan */

modparam("dialplan", "db_url", "mysql://opensips:opensipsrw@localhost/opensips")
modparam("dialplan", "attrs_pvar", "$avp(dest)")

/* parametros drouting */

modparam("drouting", "db_url", "mysql://opensips:opensipsrw@localhost/opensips")
modparam("drouting", "use_domain", 0)

/* parametros permissions */

modparam("permissions", "db_url","mysql://opensips:opensipsrw@localhost/opensips")

/* parametros group */
modparam("group", "db_url","mysql://opensips:opensipsrw@localhost/opensips")

/* parametros dialog */
modparam("dialog", "db_mode", 2)
modparam("dialog", "db_url","mysql://opensips:opensipsrw@localhost/opensips")
modparam("dialog", "dlg_match_mode", 1)
modparam("dialog", "default_timeout", 60)

/* parametros siptrace */
modparam("mi_xmlrpc", "port", 8000)
modparam("siptrace", "db_url","mysql://opensips:opensipsrw@localhost/opensips")
modparam("siptrace", "trace_flag", 22)
modparam("siptrace", "traced_user_avp", "$avp(traceuser)")

</pre>

Ok, daqui pra frente é apenas tratamento nas rotas

Temos que criar em nossa main route uma entrada especifica para jogarmos os
tratamentos para o dialplan, 

Localize a linha 
         alias_db_lookup("dbaliases");

e vamos editar abaixo dela
<pre>
        # Se o parametro iniciar com numero
        if($ruri.user=~"^[0-9]"){
                if(!dp_translate("$avp(dpid)", "$ruri.user/$ruri.user")){
                        xlog("DIALPLAN: DP_TRANSLATE nao encontrado [ $avp(dpid) - $ruri.user ]");
                        sl_send_reply("420", "Invalid Extension");
                        exit;
                }

                xlog("DIALPLAN: Subscriber dpid: $avp(dpid)");
                xlog("DIALPLAN: Destino: $avp(dest) - $ruri.user");

                if($avp(dest)=="3"){
                        # roteando para usrloc (chamada interna
                        $avp(legtype)="USRLOC";
                        setflag(1);
                        setflag(3);

                        route(3);
                }

                if($avp(dest)=="4"){
                        # roteadndo para drouting
                        $avp(legtype)="PSTN";
                        setflag(1);
                        setflag(3);

                        route(4);
                }

                if($avp(dest)=="5"){
                        # Roteando para mediaserver (aplicacoes como voicemail e outras)
                        $avp(legtype)="MEDIASERVER";
                        setflag(1);
                        setflag(3);

                        route(5);
                }

        }else{
                xlog("DIALPLAN: Usuario para destino $ru");
                route(3);
        }


</pre>

Se você reparou bem, nós apontamos as rotas para as rotas 3, 4 e 5, precisamos criar estas rotas

Vamos no fim do arquivo e vamos começar pela rota 4

<pre>
route[4]{
        xlog("DROUTING: Enviando chamada para drouting");
        xlog("DROUTING: PSTN - <$ru> - <$fu>");
        if(subst_uri('/(sip:.*);nat=yes/\1/')){
                xlog("NAT, SETBFLAG com substituicao de uri");
                setbflag(6);
        }

        if(isflagset(5) || isbflagset(6)){
                xlog("Direcionando para rota 6\n");
                route(6);
        }

        if(!do_routing("0", "0")){
                sl_send_reply("500", "Sem rota disponivel");
        }

        if(is_method("INVITE")){
                t_on_branch("2");
                t_on_reply("2");
                t_on_failure("2");
        }

        if(!t_relay()){
                sl_reply_error();
        }

        exit;

}

</pre>


Se voce reparar na verificacao do invite nós mandamos ele pra t_on_failure(2), precisamos criar a rota para isso

<pre>
failure_route[2] {
        if(isbflagset(6) || isflagset(5)){
                unforce_rtp_proxy();
        }

        if(t_was_cancelled()){
                exit;
        }

        xlog("FAILURE ROUTE 2");
        if(t_check_status("(408)|(5[0-9]{3})")){
                if(use_next_gw()){
                        xlog("PROXIMO GATEWAY: destino -> $ru\n");
                        t_on_failure("2");
                        t_relay();
                        exit;
                }else{
                        t_reply("503", "Servico indisponivel, sem mais gateways");
                        exit;
                }
        }
}

</pre>


Agora continuamos para a rota 5

<pre>
route[5]{
        # Redirecionando para o mediaserver sem mais delongas
        # Veja que este é o ip do seu servidor (um asterisk ou algo do tipo)
        rewritehostport("192.168.0.29:5060");
}


</pre>


Agora falta a rota 3, localize no arquivo a linha

         if (!lookup("location","m")) {

voce vai mover o bloco todo para a rota 3

<pre>

route[3]{

        # do lookup with method filtering
        if (!lookup("location","m")) {
                switch ($retcode) {
                        case -1:
                        case -3:
                                t_newtran();
                                t_reply("404", "Not Found");
                                exit;
                        case -2:
                                sl_send_reply("405", "Method Not Allowed");
                                exit;
                }
        }

        # when routing via usrloc, log the missed calls also
        setflag(2);

        route(1);

}

</pre>

Agora vamos instalar o painel para gerenciar isso.


==Opensips-CP==

 cd /usr/src/
 wget -c http://downloads.sourceforge.net/project/opensips-cp/opensips-cp/4.1/opensips-cp_4.1.tgz?r=http%3A%2F%2Fsourceforge.net%2Fprojects%2Fopensips-cp%2Ffiles%2Fopensips-cp%2F4.1%2F&ts=1327178872&use_mirror=ufpr
  mv opensips-cp_4.1.tgz\?r\=http\:%2F%2Fsourceforge.net%2Fprojects%2Fopensips-cp%2Ffiles%2Fopensips-cp%2F4.1%2F opensips-cp_4.1.tgz 
  tar -xzvf opensips-cp_4.1.tgz -C /var/www
</pre>

===Alterando o php.ini===

No debian é padrão, mas em todo caso certifique-se de que no arquivo /etc/php5/apache2/php.ini a opção
 short_open_tag
esteja marcada com On

Vamos dar permissão ao apache nos arquivos
 chown -R www-data.www-data /var/www/opensips-cp

Precisamos instalar o modulo MDB2 da pear
 pear install MDB2
 pear install MDB2#mysql
 pear install log


===Alterando o apache===
Vamos acrescentar um alias ao apahce, abra o arquivo /etc/apache2/sites-available/default 

Abaixo da seção directory do /var/www  (depois de </Directory>) acrescente
 Alias /cp "/var/www/opensips-cp/web"

Saia do arquivo, salve e reinicie o apache


===Criando as tabelas===
 mysql -uroot -p opensips < /var/www/opensips-cp/config/tools/system/cdrviewer/cdrs.mysql
 mysql -uroot -p opensips < /var/www/opensips-cp/config/tools/system/cdrviewer/opensips_cdrs_1_6.mysql


===Configurando o Opensips-cp===
 cd /var/www/opensips-cp
 
Abra o arquivo config/boxes.global.inc.php

Comente a linha:
 $boxes[$box_id]['mi']['conn']="127.0.0.1:8080";
e descomente a linha
 $boxes[$box_id]['mi']['conn']="/tmp/opensips_fifo";


Comente as linhas em seguida referente ao monit

após a linha 
 $boxes[$box_id]['smonitor']['charts']=1;

Comente (ou remova) as demais linhas (isso é para voce administrar por esta interface mais de um opensips)

Logo Abaixo você vai ter as definições do sistema, deixe apenas a primeira sequencia, remova ou comente a segunda

====Habilitando o dialplan====

 cd /var/www/opensips-cp/config/tools/system/dialplan/
Abra o arquivo db.inc

Altere as variaveis para as suas informacoes do banco

Exemplo:
<pre>
 //database host
 $config->db_host_dialplan = "localhost";

 //database port - leave empty for default
 //$config->db_port_dialplan = "";

 //database connection user
 $config->db_user_dialplan = "opensips";

 //database connection password
 $config->db_pass_dialplan = "opensipsrw";

 //database name
 $config->db_name_dialplan = "opensips";

 //if ($config->db_port_dialplan != "") $config->db_host_dialplan = $config->db_host_dialplan . ":" . $config->db_port_dialplan;

</pre>


Agora altere o arquivo local.inc.php

Comente o array que existe no mesmo e deixe apenas os valores a seguir
<pre>
                     array("3", "USRLOC"),
                     array("4", "EXTERN"),
                     array("5", "SERVICE"),

</pre>

Agora execute:
 mysql -uroot -p opensips < /var/www/opensips-cp/config/tools/admin/add_admin/ocp_admin_privileges.mysql 

Agora entre no mysql:
 mysql -uroot -p opensips

dentro digite:
  INSERT INTO ocp_admin_privileges (username,password,ha1,available_tools,permissions) values ('admin','admin',md5('admin:admin'),'all','all');
  alter table subscriber add column `dpid` int(10) default '0' ;
  alter table subscriber add column `quota` int(10) ;
  update subscriber set dpid = 0;

===Colocando o apache no grupo do opensips===
Isso é necessário para que o mesmo tenha acesso ao socket
  usermod -G opensips www-data



===Acessando a interface===

Coloque no seu browser o ip de sua maquina /cp

 http://192.168.0.30/cp

Usuario admin e senha admin 

Agora podemos gerenciar as configurações por ai.

==Criando um tronco==

Vamos inicialmente presumir que voce ja configurou um asterisk que esta pronto para receber estes dados

Clique no menu system 
Depois clique no botão ADD NEW

Primeiro selecionamos o TYPE proxy

Agora colocamos o ip
 
A Opcao Strip é um digito que voce queira remover

A Opcao PRI PREFIX é um digito que voce queira acrescentar

Description é uma descrição para uso seu



Nas abas superiores voce tem uma opcao gateway list, que voce pode adicionar varios gatweays em uma lista


Precisamos agora ir na aba rules 

La vamos apontar o nosso tronco pra um id

selecione o grupo id regular e clique em add

priority 0


route id 0

selecione use gateway

Selecione o seu gateway e clique em add

Coloque uma descrição (usei Default)



===Criando um DialPlan===

Clique no menu dialplan, e no botão ADD NEW

Coloque dialplan id e rule priority em 0

Matching operator coloque REGEX

matching regular expression: ^0[1-9][1-9][2-5][0-9]{7}$
 
matching string length: 0

substituition expression: (^0|\+)([1-9][1-9][2-5][0-9]{7})$

replacement expression 55\2

Marque a opção EXTERN e clique em save.




==Configuração basica de asterisk apenas para visualizar==
[ necessário asterisk instalado é claro ]

Abra o arquivo /etc/asterisk/sip.conf e adicione ao fim do arquivo

 [opensips]
 host=IP_DO_HOST
 context=opensips
 qualify=yes


Abra o arquivo /etc/asterisk/extensions.conf e adicione ao fim do arquivo

 [opensips]
 exten => _X.,1,NoOp(Ligacao entrando de ${CALLERID(all)} para ${EXTEN})
 exten => _X.,n,Answer()
 exten => _X.,n,PlayBack(tt-monkeys)
 exten => _X.,n,Hangup


Salve e saia, de um restart no asterisk (ou reload)  e do opensips ligue para um numero no modelo 
 0 DDD NUMERO 

O numero tem que ser fixo, pois nossa experessão regular só pega numeros iniciados de 2 a 5 .




==Conclusão==

É isso, 10 horas de trabalho para escrever este tutorial, acredito que não tenha ficado nenhum erro, e espero que você possa aproveitar, 
é claro que eu nao falei aqui de todas as opções que temos de usar, mas a base esta toda configurada.

Se você quer conhecer mais sobre opensips leia bem a documentação disponivel no site deles (em ingles) e também participe da lista de email, 

Lista de email: http://www.opensips.org/Resources/MailingLists
Documentação (por modulo): http://www.opensips.org/html/docs/modules/1.7.x/


A Voffice também da curso de opensips e o instruturo (Flavio E. Gonçalves) escreveu um bom livro sobre o assunto

Livro do Flavio: http://www.packtpub.com/building-telephony-systems-with-opensips-1-6/book


É isso, espero que este tutorial sirva de base para você.



Mike Tesliuk


[[Opensips|Voltar]]
