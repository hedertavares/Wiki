== Compilando o modulo Fail2Ban ==
=== Pre-requisitos ===

Ter o soruce do FreeSWITCH no diretório /usr/src

==== Baixando o modulo ====

<pre>
cd /usr/src
git clone git://github.com/eluizbr/mod_fail2ban.git
</pre>

==== Compilando o modulo ====

<pre>
cd /usr/src/mod_fail2ban
make clean
make
make install
</pre>

==== Carregando o modulo ===

<pre>

</pre>
