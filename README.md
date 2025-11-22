# SIMULADOR DISTRIBUIDO DE PLANIFICACIÓN DE PROCESO CON ZERO TRUST (NTFTABLES) 

** INTEGRANTES:** 
JUAN MANUEL VIZUETTE FAJARDO 
JUAN CARLOS MUÑOZ PABON
** FECHA:**21 DE NOVIEMBRE DEL 2025

# ARQUITECTURA DEL SISTEMA

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
# En worker1 y worker2
python3 ~/simulador/worker.py

# En controller
python3 ~/simulador/controller.py

## 3. Configuracion del Firewall Zero Trust

table inet filter {
    set admin_ips { elements = { 181.52.169.145, 190.242.58.130 } }
    set cluster_ips { elements = { 20.151.98.85, 20.151.84.187, 20.151.61.138 } }

    chain input {
        type filter hook input priority 0; policy drop;
        iif "lo" accept
        ct state established,related accept
        tcp dport 22 ip saddr @admin_ips limit rate 5/minute burst 10 packets accept
        tcp dport 9001 ip saddr @cluster_ips accept
    }
}

## 4. Lectura de los resultados generados por el simulador
cat ~/simulador/controller.py

## 5. Leemos la 
