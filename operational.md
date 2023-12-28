- create user
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/b2a9ba6f-d5d0-4507-902c-03fd372617de)

- create repository docker
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/d0ca1998-e028-40aa-87f3-902eb3fc0e9b)
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/00e264ea-2b61-44cf-9c54-21186bd2200a)
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/1cad19ba-f336-42fe-aa12-ce851ae219c2)

- install & set haproxy
> haproxy digunakan agar registry berjalan pada port 443(https)

```
apt install haproxy -y
```

```
cat << 'EOF' >> /etc/haproxy/haproxy.cfg
frontend registry
        bind *:443
        mode tcp
        option tcplog
        default_backend registry

backend registry
        mode tcp
        server localhost registry-nexus.cloud:7443
EOF
```

```
systemctl restart haproxy
```

- login on registry docker
copy ca root ke node docker
```
cat << 'EOF' >> /etc/hosts
192.168.210.2 registry-nexus.cloud
EOF
```
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/55d56d34-34bc-499b-9d3d-d08448c67aac)
![image](https://github.com/galihtw04/nexus-registry/assets/96242740/8be0ede8-b7f8-4392-8321-5f67433a48e9)
