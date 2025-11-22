# SIMULADOR DISTRIBUIDO CON ZERO TRUST (NTFTABLES)

** JUAN MANUEL VIZUETTE FAJARDO && JUAN CARLOS MUÑOZ PABON

# ARQUITECTURA DEL SISTEMA

| Máquina     | Nombre VM   | IP Pública (Elastic IP) | IP Privada   | Rol                     |
|-------------|-------------|--------------------------|--------------|-------------------------|
| Controller  | controller  | `20.151.98.85`           | 172.17.0.4   | Envía procesos y recolecta resultados |
| Worker 1    | worker1     | `20.151.84.187`          | 172.17.0.5   | Ejecuta FCFS            |
| Worker 2    | worker2     | `20.151.61.138`          | 172.17.0.6   | Ejecuta FCFS            |
