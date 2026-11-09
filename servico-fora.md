# Runbook: Serviço ou alvo fora do ar

**Alerta:** `ServicoForaDoAr` | **Severidade:** Critical | **Dono:** Suporte SRE N3

## Sintoma
O Prometheus não consegue coletar métricas de um alvo (`up == 0`) por mais de 1 minuto. Pode ser queda do serviço, do servidor ou problema de rede.

## Como diagnosticar
1. Veja no Prometheus (`localhost:9090/targets`) qual alvo está DOWN e a mensagem de erro.
2. Teste a rede até o alvo:
   ```bash
   ping <host>
   nslookup <host>          # resolução DNS
   nc -zv <host> <porta>    # porta aberta?
   curl -v http://<host>:<porta>/metrics
   ```
3. Se tiver acesso ao servidor, verifique o serviço:
   ```bash
   systemctl status <servico>
   journalctl -u <servico> --since "30 min ago"
   ```
4. Em Docker:
   ```bash
   docker ps -a
   docker logs <container> --tail 50
   ```
5. Verifique recursos: `df -h`, `free -m`, `uptime`.

## Como resolver
- **Serviço parado:** `systemctl restart <servico>` (ou `docker compose restart <servico>`) e acompanhe os logs.
- **DNS:** corrija o registro ou a configuração de resolvers.
- **Firewall/porta:** valide regras e peça liberação via mudança.
- **Servidor fora:** verifique a VM no hypervisor (VMware/Hyper-V) e acione infraestrutura.

## Falso positivo?
Manutenção planejada, reboot, ou falha momentânea de rede. Confirme no calendário de mudanças e crie um silence se for manutenção.

## Quando escalar
- Serviço não volta após reinício.
- Servidor/VM inacessível: escalar para infraestrutura.
- Impacto em usuários: abrir incidente prioritário e comunicar as partes interessadas.

## Registro
Documente: horário da queda e retorno, causa, ação tomada e se haverá problema/RCA.
