# Runbook: Alta utilização de CPU

**Alerta:** `AltaCPU` | **Severidade:** Warning | **Dono:** Suporte SRE N3

## Sintoma
CPU acima de 80% por mais de 5 minutos. Possíveis efeitos: lentidão, timeouts, fila de requisições.

## Como diagnosticar
1. Confirme no Grafana (dashboard Node Exporter Full, painel "CPU Busy") se o pico é real ou pontual.
2. Acesse o servidor e identifique os processos que mais consomem:
   ```bash
   top -o %CPU          # ou: htop
   ps aux --sort=-%cpu | head -10
   ```
3. Verifique a carga média e o número de CPUs:
   ```bash
   uptime
   nproc
   ```
   Carga (load) acima do número de CPUs indica saturação.
4. Verifique se houve deploy, job agendado ou aumento de tráfego no mesmo horário:
   ```bash
   journalctl --since "30 min ago" | tail -50
   crontab -l
   ```

## Como resolver
- **Processo travado ou em loop:** reinicie o serviço (`systemctl restart <servico>`) após avisar o responsável.
- **Job agendado pesado:** reagende para horário de baixo uso.
- **Aumento real de demanda:** solicite aumento de recursos (CPU/VM) via chamado de mudança.

## Falso positivo?
Picos curtos (backup, antivírus, batch) que normalizam sozinhos. Se recorrente, ajuste o `for:` ou o limite do alerta.

## Quando escalar
- CPU segue acima de 80% após reiniciar o serviço.
- Causa está no código da aplicação: escalar para o time de desenvolvimento com evidências (prints, logs, horário).

## Registro
Documente no chamado: horário, processo causador, ação tomada e causa raiz.
