---
{"dg-publish":true,"permalink":"/ctf/yummy-ctf/"}
---


### Nmap

```
echo "10.10.11.36 yummy.htb" | sudo tee -a /etc/hosts 
```

```
nmap yummy.htb -p- --min-rate 10000
```

![Pasted image 20250115161657.png](/img/user/Pasted%20image%2020250115161657.png)

```
nmap -p 22,80 -sCV yummy.htb
```

![Pasted image 20250115162002.png](/img/user/Pasted%20image%2020250115162002.png)

![Pasted image 20250115162043.png](/img/user/Pasted%20image%2020250115162043.png)

**HTTP Server Header**: Sunucu, kendisini "Caddy" olarak tanıtmış.


### Directory Brute Force

![Pasted image 20250115162631.png](/img/user/Pasted%20image%2020250115162631.png)
![Pasted image 20250115162713.png](/img/user/Pasted%20image%2020250115162713.png)
![Pasted image 20250115162756.png](/img/user/Pasted%20image%2020250115162756.png)
![Pasted image 20250115162828.png](/img/user/Pasted%20image%2020250115162828.png)
![Pasted image 20250115162842.png](/img/user/Pasted%20image%2020250115162842.png)
