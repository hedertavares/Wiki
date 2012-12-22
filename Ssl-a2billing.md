Esse post é sobre como gerar os certificados SSL para ser utilizado no A2Billing. Desta forma iremos incrementar a  segurança.


====Instalando dependências====

Dependências necessárias para a instalação do SSL.

<pre>
yum install mod_ssl openssl
</pre>

==== Agora iremos gerar a chave privada ====

<pre>
openssl genrsa -out ca.key 1024
</pre>

==== Gerando o arquivo CSR ====

<pre>
openssl req -new -key ca.key -out ca.csr

You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [GB]:BR
State or Province Name (full name) [Berkshire]:DIGITE O SEU ESTADO
Locality Name (eg, city) [Newbury]:DIGITE SUA CIDADE
Organization Name (eg, company) [My Company Ltd]:NOME DA EMPRESA
Organizational Unit Name (eg, section) []:
Common Name (eg, your name or your servers hostname) []:DOMINIO OU IP
Email Address []:EMAIL DE CONTATO

Please enter the following extra attributes
to be sent with your certificate request
A challenge password []: EM BRANCO
An optional company name []: EMBRANCO
</pre>

==== Assine o arquivo CSR utilizando a chave privada para gerar o certificado ====

<pre>
openssl x509 -req -days 365 -in ca.csr -signkey ca.key -out ca.crt
</pre>

==== Mova os arquivos para a localização correta ====
<pre>
mv ca.crt /etc/pki/tls/certs
mv ca.key /etc/pki/tls/private/ca.key
mv ca.csr /etc/pki/tls/private/ca.csr
</pre>

=== Apeche sem virtual host ===

Caso não utilize Virtual Host no seu servidor Apache, basta alterar as linhas abaixo no arquivo de configuração do SSL:

<pre>
mcedit /etc/httpd/conf.d/ssl.conf

SSLCertificateFile /etc/pki/tls/certs/ca.crt
SSLCertificateKeyFile /etc/pki/tls/private/ca.key
</pre>


==== Reinicie o Apache ====
<pre>
/etc/init.d/httpd restart
</pre>

=== Apeche com virtual host ===

==== Crindo a configuração ====

<pre>
mcedit /etc/httpd/conf.d/a2billing.conf

NameVirtualHost *:443
SSLEngine on
SSLCertificateFile /etc/pki/tls/certs/ca.crt
SSLCertificateKeyFile /etc/pki/tls/private/ca.key

DocumentRoot /var/www/html/billing/
ServerName IP OU DOMINIO
</pre>

==== Reinicie o Apache ====
<pre>
/etc/init.d/httpd restart
</pre>
