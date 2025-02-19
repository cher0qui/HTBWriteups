
viendo que tenemos una interfaz docker vamos a escanear la red en búsqueda de posibles puntos de ataque;

Ya que nmap no esta  instalado en la maquina vamos a escanear la maquina con tecnicas lotl (living off the land)

[Living  off the land tecniches : nmap](https://sevenlayers.com/index.php/blog/490-living-off-the-land-scanning)
## Escaneo de Ips;
```bash
james@trickster:~$ for i in {1..254}; do ping -w 2 -c 1 172.17.0.$i | grep "64 bytes" | cut -d ' ' -f 4 | sed 's/://'; done;

172.17.0.1
172.17.0.2
```

Vemos que tiene conexion con 2 ips dentro del rango; 
la `172.17.0.1 `es la del propio host; mientras que la `172.17.0.2` parece de un container; vamos a investigar puertos abiertos

## Escaneo de puertos
Escaneamos con nc el rango entero de puertos de la ip `172.17.0.2`

```bash
james@trickster:~$ nc -vnz -w 1 172.17.0.2 1-65535 2>&1 | grep succeeded
Connection to 172.17.0.2 5000 port [tcp/*] succeeded!
```
Tenemos conexion con el puerto 5000 vamos a hacer pivoting con ssh

# Pivoting

Tenemos algun servicio corriendo en un docker en la ip `172.17.0.2` por el puerto 5000 vamos a hacer un puente con ssh para tener conexion con el puerto aunque podriamos hacer pivoting con chisel ,metaexploit o lingolo-ng;

```bash
➜  ~ ssh james@trickster.htb -L 5000:172.17.0.2:5000
```

Podemos escanear el puerto con nmap a ver si encontramos informacion del servicio; ahora que esta establecido el puente la conexion

```bash
sudo nmap -sCV -p 5000 -v -Pn -n localhost
PORT     STATE SERVICE VERSION                                       
5000/tcp open  http    Python http.server 3.5 - 3.10
| http-title: Change Detection
|_Requested resource was /login?next=/
| http-methods: 
|_  Supported Methods: HEAD OPTIONS GET
```

Nos encontramos ante un changedetection.io en la v0.45.20
![[Pasted image 20250213201338.png]]
Vemos 