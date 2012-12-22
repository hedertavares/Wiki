== Introdução ==
Este é um guia de instalação rápida genérico para Linux e Unix. Esta página deve permanecer genérica, sem referências para distribuições específicas.

== A quem se destina?  ==
Experientes de Linux / Unix usuários familiarizados com FreeSWITCH instalação ™ ou quiser um breve resumo sem ler o [Guia de Instalação http://wiki.freeswitch.org/wiki/Installation_Guide].

== Pré-requisitos ==
=== Obrigatória ===
Esses pré-requisitos obrigatórios fornecer para a elaboração da norma FreeSWITCH instalação ™ e testar a configuração fornecida e IVR amostra. Eles são suficientes para muitos sistemas de produção.
* [Http://en.wikipedia.org/wiki/Git_ (software)'' 'GIT'''] ou [http://en.wikipedia.org/wiki/Wget'' 'wget''']
* [Http://en.wikipedia.org/wiki/Autoconf'' 'Autoconf''']
* [Http://en.wikipedia.org/wiki/Automake'' 'AUTOMAKE''']
* [Http://en.wikipedia.org/wiki/C% 2B% 2B'' 'GCC-C + +''']
* [Http://en.wikipedia.org/wiki/Libjpeg~~HEAD=NNS'' 'libjpeg-devel'''] usados ​​pelo mod_spandsp para codecs básicos
* [Http://en.wikipedia.org/wiki/Libtool'' 'LIBTOOL''']
* [Http://en.wikipedia.org/wiki/Make_ (software)'' 'Faz''']
* [Http://en.wikipedia.org/wiki/Ncurses'' 'ncurses DEVEL''']

Debian / Ubuntu:
<pre>
apt-get -y update
        apt-get -y install autoconf automake autotools-dev binutils bison build-essential cpp curl flex g++ gcc git-core libaudiofile-dev libc6-dev libdb-dev libexpat1  \
libexpat1-dev libgdbm-dev libgnutls-dev libmcrypt-dev libncurses5-dev libnewt-dev libpcre3 libpopt-dev libsctp-dev libsqlite3-dev libtiff4 libtiff4-dev \
 libtool libx11-dev libxml2 libxml2-dev lksctp-tools lynx m4 make mcrypt ncftp nmap openssl sox sqlite3 ssl-cert ssl-cert unixodbc-dev unzip zip zlib1g-dev zlib1g-dev
        apt-get -y install libssl-dev pkg-config
        apt-get -y install libvorbis0a libogg0 libogg-dev libvorbis-dev
        apt-get -y install flite flite1-dev
</pre>

CentOS :
<pre>
yum install git autoconf automake libtool ncurses-devel libjpeg-devel
yum groupinstall 'Development Tools' -y
yum install ncurses-devel mysql-devel -y
</pre>

== Baixando ==
Mude para o diretório /usr/src/ para baixar fonte FreeSWITCH ™:
  cd usr/src


Git:
<pre>
  git clone git://git.freeswitch.org/freeswitch.git
  cd freeswitch
  . / Bootstrap.sh
</pre>

== Editar modules.conf == 
Este é opcional, mas considerar [http://wiki.freeswitch.org/wiki/Installation_Guide # Edit_modules.conf modules.conf edição].

== Compilar o código fonte ==

<pre> 
 ./configure --without-pgsql --prefix=/usr/local/freeswitch --sysconfdir=/etc/freeswitch/
</pre>

== Instalando FreeSWITCH ™ ==

<pre>
  make && make install && make sounds-install && make moh-install
</pre>

== Iniciando o FreeSWITCH ™ ==
Certifique-se de nenhuma outra instância do FreeSWITCH ™ ou Asterisk está sendo executado no mesmo computador. Execute o comando:
 /usr/local/freeswitch/bin/freeswitch

== Teste um telefone SIP ==
Configurar um telefone SIP ou softphone com o endereço IP do computador ™ FreeSWITCH e usuário "1000" e senha "1234". O padrão de configuração FreeSWITCH ™ fornece pré-definidos definições para extensões de 1000-1019, todas as senhas são 1234.

* Disque 9664 para ouvir música.
* Disque 5000 para testar a amostra IVR
* Configurar um telefone SIP segunda como usuário 1001, disque 1001 a partir de 1000, e 1000 de 1001 para testar, entre os telefones
* Confira mais em [http://wiki.freeswitch.org/wiki/Getting_Started_Guide # Some_stuff_to_try_out.21 tentar alguma coisa]

=== Solução de problemas: ===
* Se não houver comunicação entre os telefones e ™ FreeSWITCH ou entre telefones desativar o firewall em seu servidor Linux. Se o teste for bem sucedido ativar o firewall e abrir apenas as portas específicas necessárias. Para mais informações sobre firewalls e portas de abrir, ver o [[Firewall]] página.
* Se você suspeitar de problemas de rede olhar para ferramentas de rede como o Wireshark ou tcpdump.
