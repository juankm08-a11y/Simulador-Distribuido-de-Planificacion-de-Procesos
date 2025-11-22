# SIMULADOR DISTRIBUIDO DE PLANIFICACIÓN DE PROCESO CON ZERO TRUST (NTFTABLES) 

**INTEGRANTES:** 
JUAN MANUEL VIZUETTE FAJARDO 
JUAN CARLOS MUÑOZ PABON
**FECHA:** 21 DE NOVIEMBRE DEL 2025

# 1. ARQUITECTURA DEL SISTEMA

| Máquina     | Nombre VM   | IP Pública (Elastic IP) | IP Privada   | Rol                     |
|-------------|-------------|--------------------------|--------------|-------------------------|
| Controller  | controller  | `20.151.98.85`           | 172.17.0.4   | Envía procesos y recolecta resultados |
| Worker 1    | worker1     | `20.151.84.187`          | 172.17.0.5   | Ejecuta FCFS            |
| Worker 2    | worker2     | `20.151.61.138`          | 172.17.0.6   | Ejecuta FCFS            |


- Configuramos todas las VMs en la misma VNet/subred privada
- **NSG completamente cerrados** → única defensa: nftables
- Comunicación del simulador: por **Elastic IP** en puerto 9001

## 2. Ejecución del Simulador 

```bash
# 1. VER CONFIGURACION DE RED
ip a | grep inet

# 2. HACER PING ENTRE LAS VMS
ping 2 20.151.84.187
ping 2 20.151.61.138

# 3. EJECUTAR WORKERS (UNO EN CADA WORKER)
cd ~/simulador && python3 worker.py

# 4. EJECUTAR CONTROLLER (ROUND-ROBIN AUTOMÁTICO)
cd ~/simulador && python3 controller.py

# 5. VER LOS RESULTADOS GENERADOS
cat ~/simulador/results.json
````

## 3. CONFIGURACION DEL FIREWALL ZERO TRUST 

Guardamos en /etc/nftables.conf en cada maquina VM las nftables

table inet filter {
    set admin_ips { type ipv4_addr; elements = { 181.52.169.145, 190.242.58.130 } }
    set cluster_ips { type ipv4_addr; elements = { 20.151.98.85, 20.151.84.187, 20.151.61.138 } }

    chain input {
        type filter hook input priority 0; policy drop;
        iif "lo" accept
        ct state established,related accept
        tcp dport 22 ip saddr @admin_ips limit rate 5/minute burst 10 packets accept
        tcp dport 22 counter drop
        tcp dport 9001 ip saddr @cluster_ips accept
        tcp dport 9001 counter drop comment "Puerto 9001 bloqueado - fuera del cluster"
        counter
    }
    chain output { type filter hook output priority 0; policy accept; }
    chain forward { type filter hook forward priority 0; policy drop; }
}

## 4. VERIFICACION DEL FIREWALL (COMANDOS CLAVE)

# Recargar firewall
sudo nft -f /etc/nftables.conf

# Mostrar reglas completas
sudo nft list ruleset -a

# PRUEBA DE ORO: contadores subiendo en tiempo real
watch -n 1 "sudo nft list ruleset -a | grep -A 3 9001"

# Estado del servicio
sudo systemctl status nftables

# Puertos escuchando
ss -tuln | grep 9001

# Procesos activos
ps aux | grep python

## 5. PRUEBAS DE ACCESO SSH RESTRINGIDO 
Desde IP autorizada (181.52.169.145 o 190.242.58.130) → SSH funciona
Desde cualquier otra red/dispositivo (datos móviles, compañero, etc.) → Connection timed out
→ Demuestra que el acceso administrativo está 100% restringido por admin_ips


