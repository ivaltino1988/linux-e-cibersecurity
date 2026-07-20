# Sessão 3 — Hardening de Redes Linux e Configuração de Firewalls

## Contexto

Configuração de uma política defensiva estrita para impedir acessos não autorizados a serviços críticos do servidor, combinando **UFW** e **iptables**.

---

## Passo 1 — Estado inicial do UFW

```bash
$ sudo ufw status
Status: inactive
```

O firewall não estava ativo por defeito neste ambiente.

## Passo 2 — Configuração das políticas padrão (Default Deny)

```bash
$ sudo ufw default deny incoming
Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)

$ sudo ufw default allow outgoing
Default outgoing policy changed to 'allow'
(be sure to update your rules accordingly)
```

Política aplicada: **bloquear todo o tráfego de entrada por defeito**, permitindo apenas o tráfego de saída — princípio de *Default Deny*.

## Passo 3 — Exceção controlada para SSH

```bash
$ sudo ufw allow 22/tcp
Rules updated
Rules updated (v6)
```

Abertura de uma exceção específica para o serviço SSH (porta 22/TCP), garantindo acesso administrativo remoto mesmo com a política de bloqueio geral ativa.

## Passo 4 — Simulação de bloqueio de IP malicioso via iptables

```bash
$ sudo iptables -A INPUT -s 203.0.113.50 -j DROP
```

Regra manual adicionada diretamente na chain **INPUT** do Netfilter, bloqueando de forma **silenciosa** (`DROP`, sem resposta ao emissor) qualquer tráfego proveniente do IP fictício `203.0.113.50`.

## Passo 5 — Persistência das regras do iptables

```bash
$ sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

Exporta o conjunto de regras ativo para um ficheiro persistente, garantindo que a regra de bloqueio sobrevive a um reinício do sistema.

## Passo 6 — Ativação do firewall

```bash
$ sudo ufw enable
Firewall is active and enabled on system startup
```

⚠️ Passo crítico: as políticas configuradas nos passos anteriores só passam a ter efeito real a partir deste momento.

---

## Evidências finais — `sudo ufw status verbose`

```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
```

## Evidências finais — `sudo iptables -L -v`

```
Chain INPUT (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 DROP       all  --  any    any     203.0.113.50         anywhere
12453   10M ufw-before-logging-input  all  --  any    any     anywhere             anywhere
12453   10M ufw-before-input  all  --  any    any     anywhere             anywhere
   23  7012 ufw-after-input  all  --  any    any     anywhere             anywhere
   23  7012 ufw-after-logging-input  all  --  any    any     anywhere             anywhere
   23  7012 ufw-reject-input  all  --  any    any     anywhere             anywhere
   23  7012 ufw-track-input  all  --  any    any     anywhere             anywhere

Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ufw-before-logging-forward  all  --  any    any     anywhere             anywhere
    0     0 ufw-before-forward  all  --  any    any     anywhere             anywhere
    0     0 ufw-after-forward  all  --  any    any     anywhere             anywhere
    0     0 ufw-after-logging-forward  all  --  any    any     anywhere             anywhere
    0     0 ufw-reject-forward  all  --  any    any     anywhere             anywhere
    0     0 ufw-track-forward  all  --  any    any     anywhere             anywhere

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
12149   16M ufw-before-logging-output  all  --  any    any     anywhere             anywhere
12149   16M ufw-before-output  all  --  any    any     anywhere             anywhere
   56 10966 ufw-after-output  all  --  any    any     anywhere             anywhere
   56 10966 ufw-after-logging-output  all  --  any    any     anywhere             anywhere
   56 10966 ufw-reject-output  all  --  any    any     anywhere             anywhere
   56 10966 ufw-track-output  all  --  any    any     anywhere             anywhere

Chain ufw-user-input (1 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 ACCEPT     tcp  --  any    any     anywhere             anywhere             tcp dpt:ssh
```

*(output completo com todas as chains internas do UFW disponível no log da sessão)*

---

## Explicação da política aplicada

| Chain | Política | O que significa |
|---|---|---|
| **INPUT** | `DROP` | Todo o tráfego de entrada é bloqueado por defeito (Default Deny), exceto o que corresponder a uma regra explícita |
| **OUTPUT** | `ACCEPT` | O servidor pode iniciar livremente ligações de saída (atualizações, DNS, etc.) |
| **FORWARD** | `DROP` | O servidor não atua como router — não reencaminha tráfego entre redes |

### O que está bloqueado e porquê

- **Todo o tráfego de entrada não solicitado** é bloqueado por defeito (`DROP` silencioso, sem resposta ao emissor) — reduz a superfície de ataque e dificulta reconhecimento de rede por scanners externos.
- **O IP `203.0.113.50`** está explicitamente bloqueado na chain INPUT, simulando a resposta a um IP identificado como malicioso (por exemplo, através da análise de logs realizada na Sessão 2).
- **Exceção única e controlada:** a porta 22/TCP (SSH) está aberta, para garantir que o acesso administrativo remoto continua possível — sem esta exceção, o próprio administrador ficaria bloqueado do servidor.

### Porque se usou DROP e não REJECT

A ação `DROP` foi escolhida (tanto na política por defeito da INPUT como no bloqueio do IP malicioso) por ser mais **furtiva**: não informa o emissor de que o pacote foi bloqueado, dificultando a um atacante distinguir entre "porta fechada", "porta filtrada" ou "host inexistente" — ao contrário do `REJECT`, que responde ativamente e revela a presença do firewall.

### Nota técnica sobre a arquitetura interna do UFW

O `iptables -L -v` revela que o UFW não substitui o iptables — funciona como uma camada de abstração que gera as suas próprias chains internas (`ufw-before-input`, `ufw-user-input`, `ufw-after-input`, etc.), organizadas em fases de processamento, e que traduzem os comandos simplificados (`ufw allow 22/tcp`) em regras Netfilter equivalentes (visível na chain `ufw-user-input`, com a regra `ACCEPT tcp ... dpt:ssh`).
