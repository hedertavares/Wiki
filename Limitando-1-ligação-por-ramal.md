'''Edite'''<pre>vim /etc/freeswitch/dialplan/default.xml</pre>
'''Adicione'''<pre><action application="limit" data="hash ${domain} $1 1 handle_over_limit XML over_limit_actions"/></pre> Logo após a linha:

<pre>
 <extension name="Local_Extension">
      <condition field="destination_number" expression="^(10[01][0-9])$">
        <action application="export" data="dialed_extension=$1"/>
        <-- bind_meta_app can have these args <key> [a|b|ab] [a|b|o|s] <app> -->
</pre>