# SRE Lab: Observabilidade com Prometheus + Grafana

Laboratório prático de monitoramento de um host Linux usando Docker. Cobre métricas, dashboards, alertas e runbooks versionados em Git.

## O que tem aqui
| Componente | Função |
|---|---|
| Node Exporter | Expõe métricas do Linux (CPU, memória, disco, rede) |
| Prometheus | Coleta métricas e avalia regras de alerta |
| Grafana | Dashboards e visualização |
| `runbooks/` | Procedimentos de resposta a incidentes |

## Estrutura
```
.
├── docker-compose.yml
├── prometheus/
│   ├── prometheus.yml
│   └── rules/alertas.yml
├── grafana/provisioning/datasources/prometheus.yml
└── runbooks/
    ├── alta-cpu.md
    ├── disco-cheio.md
    └── servico-fora.md
```

## Como rodar
Pré-requisito: Docker Desktop (ou Docker + Compose no Linux).

```bash
docker compose up -d
```

- Prometheus: http://localhost:9090 (alvos em `/targets`, alertas em `/alerts`)
- Grafana: http://localhost:3000 (usuário `admin`, senha `admin`)

O Grafana já sobe com o Prometheus configurado como fonte de dados.

### Importar o dashboard
No Grafana: **Dashboards > New > Import** > ID `1860` (Node Exporter Full) > selecione a fonte Prometheus.

## Alertas configurados
| Alerta | Condição | Severidade | Runbook |
|---|---|---|---|
| AltaCPU | CPU > 80% por 5 min | warning | [alta-cpu](runbooks/alta-cpu.md) |
| AltaMemoria | Memória > 90% por 5 min | warning | n/a |
| DiscoQuaseCheio | Disco > 85% por 5 min | critical | [disco-cheio](runbooks/disco-cheio.md) |
| ServicoForaDoAr | alvo `up == 0` por 1 min | critical | [servico-fora](runbooks/servico-fora.md) |

## Testando um alerta
Gere carga de CPU para disparar `AltaCPU`:
```bash
docker run --rm -it busybox sh -c "while true; do :; done"
```
Acompanhe em http://localhost:9090/alerts (Pending > Firing). Para testar `ServicoForaDoAr`: `docker stop node-exporter`.

## O que aprendi
- Diferença entre métricas, logs e traces e onde cada uma entra.
- Escrever regras de alerta em PromQL e reduzir falsos positivos com `for:`.
- Documentar procedimentos em runbooks (sintoma, diagnóstico, resolução, escalonamento).
- Versionar configuração e documentação com Git.

## Próximos passos
- Alertmanager para notificações (e-mail/Teams).
- Loki para logs e Tempo/Jaeger para traces.
- Monitorar um banco de dados (postgres_exporter) e uma aplicação web.
