# Runbook: Disco quase cheio

**Alerta:** `DiscoQuaseCheio` | **Severidade:** Critical | **Dono:** Suporte SRE N3

## Sintoma
Partição com mais de 85% de uso. Risco: aplicação sem conseguir gravar logs/dados, banco parando, serviços caindo.

## Como diagnosticar
1. Veja o uso por partição:
   ```bash
   df -hT
   ```
2. Encontre os diretórios maiores:
   ```bash
   sudo du -xh / --max-depth=2 2>/dev/null | sort -rh | head -15
   ```
3. Procure arquivos grandes:
   ```bash
   sudo find / -xdev -type f -size +500M -exec ls -lh {} \; 2>/dev/null
   ```
4. Verifique logs crescendo e arquivos deletados ainda abertos:
   ```bash
   sudo du -sh /var/log/*
   sudo lsof +L1 | head
   ```
5. Verifique inodes (disco pode estar "cheio" por inodes):
   ```bash
   df -i
   ```

## Como resolver
- **Logs antigos:** compacte ou remova conforme política de retenção
  ```bash
  sudo journalctl --vacuum-time=7d
  sudo find /var/log -name "*.gz" -mtime +30 -delete
  ```
- **Arquivo deletado ainda aberto:** reinicie o processo que o mantém aberto.
- **Docker acumulando:** `docker system df` e, com aprovação, `docker system prune`.
- **Crescimento legítimo:** abra mudança para expandir o disco.

> Nunca apague arquivos de dados ou de banco sem confirmação do responsável.

## Falso positivo?
Partições somente leitura (ex.: squashfs, snap) sempre em 100%. Exclua-as da regra por `fstype`/`mountpoint`.

## Quando escalar
- Disco acima de 95% e crescendo.
- Causa em banco de dados ou aplicação: acionar DBA/desenvolvimento.

## Registro
Documente: partição, tamanho antes/depois, o que foi removido e ação preventiva (rotação de logs, expansão).
