# Sessão 2 — Auditoria de Sistemas Linux e Análise Avançada de Logs

## Contexto

Um servidor da infraestrutura foi alvo de conexões anómalas. Atuámos como analista forense para determinar a origem e o sucesso do ataque, com base na análise de logs de autenticação (`auth.log`), na sala TryHackMe **Linux Server Forensics**.

> ⚠️ **Nota metodológica:** o tempo de utilização da AttackBox/máquina alvo esgotou-se a meio do exercício, antes de se confirmar o login bem-sucedido (`Accepted password`). Este documento regista fielmente o progresso alcançado até esse ponto, incluindo os obstáculos técnicos encontrados e como foram resolvidos. A investigação será retomada numa sessão futura.

---

## Passo 1 — Navegação até à diretoria de logs

```bash
root@ip-10-129-161-225:~# cd /var/log/
root@ip-10-129-161-225:/var/log#
```

## Passo 2 — Confirmação da existência e estado do `auth.log`

```bash
root@ip-10-129-161-225:/var/log# ls -la
...
-rw-r-----   1 syslog   adm   4326 Jul 21 18:12 auth.log
-rw-r-----   1 syslog   adm   2192 Jul 15 15:32 auth.log.1
-rw-r-----   1 syslog   adm    724 Jul  8 14:13 auth.log.2.gz
...
```

Confirmada a existência do ficheiro, com rotação de logs ativa (`auth.log.1`, `auth.log.2.gz`, etc. — gerida por `logrotate`).

## Passo 3 — Inspeção inicial do conteúdo bruto

```bash
root@ip-10-129-161-225:/var/log# head -20 auth.log
2026-07-16T04:11:52.134445+00:00 ip-10-82-109-30 sudo: pam_unix(sudo:session): session closed for user root
2026-07-21T18:05:24.805370+00:00 ip-10-129-161-225 usermod[703]: change user 'root' password
2026-07-21T18:05:24.805382+00:00 ip-10-129-161-225 passwd[736]: password for 'ubuntu' changed by 'root'
...
```

**Observação:** este `auth.log` não continha, inicialmente, nenhuma entrada de tentativa de login falhada — apenas eventos administrativos de rotina (sudo, cron, systemd, mudanças de password). Para prosseguir com a análise prática, foram geradas tentativas de autenticação reais no próprio ambiente de laboratório.

## Passo 4 — Geração de tentativas de autenticação para análise

Foi tentado o acesso via SSH com um utilizador de teste (`fred`, credenciais fornecidas pelo cenário da sala), com passwords incorretas propositadas:

```bash
root@ip-10-129-161-225:/var/log# ssh fred@localhost
...
fred@localhost's password:
Permission denied, please try again.
```


## Passo 5 — Isolamento das tentativas falhadas geradas

```bash
root@ip-10-129-161-225:/var/log# grep "Failed password" auth.log
2026-07-21T18:23:55.719862+00:00 ip-10-129-161-225 sshd[8738]: Failed password for invalid user fred from 127.0.0.1 port 36204 ssh2
2026-07-21T18:24:07.053564+00:00 ip-10-129-161-225 sshd[8738]: Failed password for invalid user fred from 127.0.0.1 port 36204 ssh2
2026-07-21T18:24:14.001872+00:00 ip-10-129-161-225 sshd[8738]: Failed password for invalid user fred from 127.0.0.1 port 36204 ssh2
2026-07-21T18:26:58.346196+00:00 ip-10-129-161-225 sshd[9724]: Failed password for invalid user fred from 127.0.0.1 port 56648 ssh2
2026-07-21T18:27:22.519725+00:00 ip-10-129-161-225 sshd[9724]: Failed password for invalid user fred from 127.0.0.1 port 56648 ssh2
2026-07-21T18:27:47.615563+00:00 ip-10-129-161-225 sshd[9724]: Failed password for invalid user fred from 127.0.0.1 port 56648 ssh2
```

**Nota técnica sobre o formato do log:** 

```bash
root@ip-10-129-161-225:/var/log# grep "Failed password" auth.log | awk '{print $(NF-3)}'
127.0.0.1
127.0.0.1
127.0.0.1
127.0.0.1
127.0.0.1
127.0.0.1
```

## Passo 6 — Extração e contagem dos IPs com mais tentativas

```bash
root@ip-10-129-161-225:/var/log# grep "Failed password" auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr
      6 127.0.0.1
```

**Resultado:** 6 tentativas falhadas, todas provenientes de `127.0.0.1` — esperado, já que os testes foram gerados a partir do próprio servidor (via `localhost`), para fins de simulação e aprendizagem.

---

## Estado da investigação (por concluir)

| Critério de entrega | Estado |
|---|---|
| IP do atacante identificado | ✅ `127.0.0.1` (ambiente de teste simulado) |
| Hora exata do comprometimento | ⏳ Pendente — máquina expirou antes da confirmação do login bem-sucedido |
| Utilizador afetado | ⏳ Pendente 
| Linha temporal completa (falhas → sucesso) | ⏳ Parcial — só a fase de falhas está documentada |

### Linha temporal parcial

| Hora (UTC) | Evento |
|---|---|
| 18:23:55 | 1ª tentativa falhada — utilizador inválido `fred`, a partir de `127.0.0.1` |
| 18:24:07 – 18:24:14 | 2ª e 3ª tentativas falhadas, mesma sessão SSH |
| 18:26:58 – 18:27:47 | 4ª, 5ª e 6ª tentativas falhadas, nova sessão SSH |
| *(pendente)* | Login bem-sucedido — a confirmar em sessão futura |

---

## Próximos passos (a retomar)


4. Correr `grep -E "Accepted password|Accepted publickey" auth.log` para confirmar o timestamp exato do comprometimento
5. Completar a tabela de critérios de entrega acima
