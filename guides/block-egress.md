# Block Egress to Australia & China

## Using Ipset to block outgoing traffic

###### Prerequisites:

```
sudo apt install -y locales
sudo locale-gen en_CA
sudo locale-gen en_CA.UTF-8
sudo su
/usr/bin/perl -MCPAN -e'install Text::CSV_XS'
exit
sudo localedef -i en_CA -c -f UTF-8 -A /usr/share/locale/locale.alias en_CA.UTF-8
sudo update-locale LC_ALL=en_CA.UTF-8
sudo apt install -y netfilter-persistent iptables-persistent ipset
```
If a dialogue box pops up, it asks if you want to save rules. Save current ipv4 rules, `yes`, save current ipv6 rules, `no`.

[Disable ipv6](https://www.techrepublic.com/article/how-to-disable-ipv6-on-linux/) with `sudo nano /etc/sysctl.conf` and paste at the very bottom, then save and exit (`ctrl + x` and then `ctrl + o`):
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```
###### Block Australia and China Egress:

If you need to block other websites, the full list of countries is found [here](https://www.ipdeny.com/ipblocks/).

In the home directory (`cd ~`):
```
nano block-egress.sh
```
Paste:
```
#!/bin/bash

echo "### BLOCKING AUSTRALIA EGRESS ###"
echo

ipset -N block-australia hash:net -exist
ipset -F block-australia

if [ -f "au.zone" ]
then
	rm au.zone
fi

curl -o au.zone -sSL "https://www.ipdeny.com/ipblocks/data/countries/au.zone"

if [ $? -eq 0 ]
then
	echo "Download Finished!"
fi

echo

echo -n "Adding Networks to ipset ..."
for net in `cat au.zone`
do
	ipset -A block-australia $net
done

echo "Done"

echo "### BLOCKING CHINA EGRESS ###"
echo

ipset -N block-china hash:net -exist
ipset -F block-china

if [ -f "cn.zone" ]
then
	rm cn.zone
fi

curl -o cn.zone -sSL "https://www.ipdeny.com/ipblocks/data/countries/cn.zone"

if [ $? -eq 0 ]
then
	echo "Download Finished!"
fi

echo

echo -n "Adding Networks to ipset ..."
for net in `cat cn.zone`
do
	ipset -A block-china $net
done

echo "Done"

echo "### SAVING IPSET RULES ###"
echo

ipset save > /etc/iptables/ipset

echo "Done"
```
Save, exit and then make script executable:
```
chmod +x block-egress.sh
```
Create blank ipset file and save it (`sudo mkdir /etc/iptables` first if you get the warning that the folder does not exist):
```
sudo nano /etc/iptables/ipset
```
Run the script:
```
sudo ./block-egress.sh
```
Add the iptables rules:
```
sudo iptables -I OUTPUT -m set --match-set block-australia src -j DROP && sudo iptables -I OUTPUT -m set --match-set block-china src -j DROP
```
Check with `sudo iptables -L` to make sure the rules are there; `sudo iptables -L -v` for more detailed rules. `sudo iptables -S` will list the rules in the format that’s like the rules.v4 file.

Then save all the current rules to the rules.v4 file:
```
sudo su
iptables-save > /etc/iptables/rules.v4
exit
```
*Note: cat /etc/iptables/rules.v4 will list the rules in the rules.v4 file

Then make the rules persistent after reboot:
```
sudo service netfilter-persistent start
sudo service netfilter-persistent save
sudo service netfilter-persistent reload
```
###### [Make ipset persistent](https://selivan.github.io/2018/07/27/ipset-save-with-ufw-and-iptables-persistent-and.html):

```
sudo nano /etc/systemd/system/ipset-persistent.service
```
Paste the following:
```
[Unit]
Description=ipset persistent configuration
Before=network.target

# ipset sets should be loaded before iptables
# Because creating iptables rules with names of non-existent sets is not possible
Before=netfilter-persistent.service
Before=ufw.service

ConditionFileNotEmpty=/etc/iptables/ipset

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/sbin/ipset restore -exist -file /etc/iptables/ipset
# Uncomment to save changed sets on reboot
# ExecStop=/sbin/ipset save -file /etc/iptables/ipset
ExecStop=/sbin/ipset flush
ExecStopPost=/sbin/ipset destroy

[Install]
WantedBy=multi-user.target

RequiredBy=netfilter-persistent.service
RequiredBy=ufw.service
```
*Note: Unless you first execute ipset save > /etc/iptables/ipset (which is in the script) then there will be no /etc/iptables/ipset file and the service will be inactive (dead).

Save, and then:
```
sudo systemctl daemon-reload
sudo systemctl enable ipset-persistent.service
sudo systemctl start ipset-persistent.service
```
Add cron job of ipset script to ipdate CIDR ip addresses once a day at 10 pm to `sudo crontab -e` (changing $USER to your username):
```
0 22 * * * /home/$USER/block-egress.sh
```
Reboot and check iptables rules (`sudo iptables -L`) to make sure the new rules are still there.

If you want to check cron that has run (at the time you set it):
```
grep CRON /var/log/syslog
```
If iptables needs to be reloaded for some reason:
```
sudo iptables-restore < /etc/iptables/rules.v4
```
