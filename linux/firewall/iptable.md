# Iptables

## Some usefull command
List all rules  
`# iptables -L`

Common way to add a new rule :  
`# sudo iptables -t filter -A [INPUT/OUTPUT] -p [tcp/udp] --dport [PORT] -j [ACCEPT/DROP]`

Delete rule :  
`# sudo iptables -D [INPUT/OUTPUT] -p [tcp/udp] --dport [PORT] -j [ACCEPT/DROP]`

For more info : RTFM !  
`# man iptables`

## Save iptable rules
Use `iptable-persistent` package for that

On install, it will save current iptable rules. Then you use the service  
`# service iptable-persistent [re]start`  
to restore your saved rules.

If you want to save new rules, use  
`# dpkg-reconfigure iptable-persistent` 

Configuration folder : `/etc/iptables`

## Usefull links
* [Debian wiki page](http://wiki.kartbuilding.net/index.php/Iptables_Firewall)
* [Documentation ubuntu](https://doc.ubuntu-fr.org/iptables)
* [Save new rules with iptable-persistent](https://unix.stackexchange.com/questions/125833/why-isnt-the-iptables-persistent-service-saving-my-changes)