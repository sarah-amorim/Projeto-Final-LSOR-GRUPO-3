```
sudo cp -r /etc/netplan /root/netplan.bak3
sudo python3 - <<'EOF'
import glob, yaml
f = glob.glob('/etc/netplan/*.yaml')[0]
d = yaml.safe_load(open(f))
d['network']['ethernets']['ens192']['nameservers'] = {
    'search': ['grupo3.bsi-26-2.maceio.lab'],
    'addresses': ['192.168.26.50', '192.168.26.51'],
}
yaml.safe_dump(d, open(f, 'w'), default_flow_style=False)
EOF
cat /etc/netplan/*.yaml
sudo netplan try
```

Aperte Enter quando o netplan try pedir. Se perder a conexão, ele desfaz sozinho em 120 s. Confira o resultado:
```
resolvectl status | grep -E "Link|DNS Servers|DNS Domain"
```
