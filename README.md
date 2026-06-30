# Caderno de Estudos: Fundamentos de Linux (com foco em Pentesting)

## Contexto e Objetivos

Comecei a estudar Linux Fundamentals pelo HTB Academy como base pra evoluir em pentesting, hoje meu trabalho é majoritariamente WordPress/fullstack, e quero migrar parte do meu tempo de estudo pra segurança ofensiva. O problema é que o curso entrega muito conteúdo teórico em sequência (containers, redes, hardening, logs) e eu sentia que estava lendo e esquecendo.

Criei este caderno num notebook de IA pra resolver isso: em vez de só ler a documentação, uso a IA pra me forçar a comparar, confrontar minhas próprias hipóteses erradas e simular cenários de ataque/defesa em cima do conteúdo. O objetivo não é só "passar pelo módulo", é sair com material que eu consiga reusar quando for estudar HackTheBox machines de verdade.

## Curadoria de Fontes

Fontes abertas usadas como base no notebook:

1. **HTB Academy — Linux Fundamentals** (módulo principal do curso): https://academy.hackthebox.com/course/preview/linux-fundamentals
2. https://docs.docker.com/
3. https://netfilter.org/
4. https://linuxblog.io/securing-linux-selinux-apparmor/


---

## Engenharia de Prompts e "Cicatrizes" (Troubleshooting)

"Cicatriz" = ponto onde o prompt inicial não trouxe profundidade suficiente, e precisei refinar a pergunta pra extrair algo tecnicamente útil. Documentei abaixo na ordem em que aconteceram, sem arredondar os tamanhos, algumas exigiram mais voltas que outras.

### Cicatriz 1 — Namespaces vs cgroups

Comecei perguntando de forma ampla:

> `"o que são kubernetes?"` e `"resuma Linux Containers (LXC)"`

A resposta foi correta mas rasa: contêineres como instâncias leves compartilhando o kernel do host, Kubernetes como orquestrador. Faltava entender *como* o isolamento acontece de fato no nível do SO.

Refinei colando o trecho exato da doc oficial do LXC e pedindo resumo só daquele trecho:

> `"resuma especificamente: When working with containers, it is important to understand the lxc.cgroup..."`
> `"resuma LXC use namespaces..."`

Aí sim veio a diferenciação que eu precisava: **cgroups** limitam quanto de CPU/memória um contêiner pode consumir; **namespaces** isolam o que ele *enxerga* (PIDs, rede, sistema de arquivos). São dois mecanismos diferentes resolvendo dois problemas diferentes, não sinônimos.

*Aprendizado:* colar o trecho-fonte e pedir resumo daquele trecho específico impede a IA de generalizar, ela é obrigada a trabalhar em cima do que eu de fato dei pra ela.

### Cicatriz 2 — SSH, VNC, X11 e XDMCP: tudo "acesso remoto", mas quando usar qual?

Perguntei protocolo por protocolo, separado:

> `"resuma xserver"`, `"resuma X11 Security"`, `"resuma XDMCP"`, `"resuma VNC"`

Cada resposta veio correta e isolada: portas, riscos básicos — mas sem nenhuma conexão entre elas. No fim eu tinha 4 resumos soltos e ainda não sabia qual usar quando.

O prompt que resolveu foi forçar a comparação direta:

> `"qual a diferença de XDMCP, VNC, X11 Security para o famoso ssh?"`

Resultado: SSH é o protocolo de linha de comando criptografado por padrão (o "cofre forte"); VNC, X11 e XDMCP lidam com interface gráfica, sendo os dois últimos inseguros por padrão e dependentes de tunelamento via SSH pra serem usados com segurança.

*Aprendizado:* pedir resumo isolado gera conhecimento fragmentado. Pedir comparação entre N itens gera conhecimento que eu realmente consigo aplicar na hora de decidir o que usar.

### Cicatriz 3 — NAC e TCP Wrappers pareciam a mesma coisa

Aqui o problema não foi profundidade, foi sobreposição: depois de pedir resumos de NAC, segurança Linux e hardening, fiquei com a impressão de que NAC e TCP Wrappers faziam exatamente a mesma função: bloquear IP indesejado.

Em vez de pedir pra explicar de novo, confrontei minha própria teoria:

> `"o NAC não é responsável por proteger a rede bloqueando IPs que não pertencem a ela? o TCP wrapper faz a mesma coisa então?"`

A IA apontou onde minha teoria errava: NAC é política de rede (nível do prédio inteiro); TCP Wrapper é controle local por host, configurado em `/etc/hosts.allow` e `/etc/hosts.deny` — uma implementação pontual de parte do que o NAC cobre, não a mesma coisa em escopos diferentes.

*Aprendizado:* quando dois conceitos parecem idênticos, o prompt mais produtivo é apresentar a própria teoria (mesmo errada) e pedir correção pontual. Isso é bem mais eficiente que pedir "explica de novo, mais devagar".

### Cicatriz 4 — LXC vs Docker: "containers" não é uma coisa só

Depois de entender namespaces/cgroups (Cicatriz 1), assumi que LXC e Docker eram basicamente o mesmo conceito com nome diferente. Perguntei direto:

> `"faça um comparativo de lxc vs docker, destacando quando usar cada um"`

A resposta separou por filosofia, não só por feature: LXC trata o contêiner como um "system container", um SO completo, com init, múltiplos processos, pensado pra rodar indefinidamente como um servidor. Docker trata o contêiner como um "application container" em geral um processo só, efêmero, com todo um ecossistema de imagem/distribuição (Dockerfile, registry) em volta.

Isso me ajudou a entender por que o HTB Academy ensina LXC pra explicar isolamento de baixo nível, enquanto no meu dia a dia (RESTaurant, projetos com Docker) eu uso Docker, são ferramentas com propósitos diferentes, não uma versão "moderna" da outra.

### Cicatriz 5 — confundi kernel, distro e ambiente gráfico

Esse foi o erro de premissa mais claro de todos. Perguntei:

> `"o linux então é um kernel e sistemas como gnome os, arch os são distros?"`

A resposta corrigiu metade da minha hipótese: Linux é o kernel, certo. Arch é uma distro, certo. Mas GNOME **não é uma distro**, é um ambiente de desktop (DE), a camada gráfica que roda em cima de uma distro. Dá pra ter Ubuntu com GNOME, Arch com GNOME, Fedora com GNOME, é plugável, independente da distro.

A hierarquia correta:
```
Kernel (Linux)
 └─ Distro (Arch, Ubuntu, Debian, Pop!_OS...)
     └─ Ambiente gráfico / DE (GNOME, KDE, XFCE...)
```

*Aprendizado:* eu estava misturando dois eixos diferentes (qual distro / qual interface gráfica) como se fossem o mesmo eixo. Esse tipo de erro só aparece quando eu de fato tento formular a hipótese em voz alta, se eu só tivesse lido a definição de "distro" eu não teria percebido a confusão.

## Miniguia de Estudo (Entrega Final)

### Resumos estruturados

**Contêineres e Orquestração**
- LXC: virtualização leve em nível de SO, system container, compartilha o kernel do host.
- Docker: application container, geralmente 1 processo por contêiner, ecossistema forte de imagem/distribuição.
- Namespaces: isolam o que o processo enxerga (PID, rede, filesystem).
- cgroups: limitam quanto de CPU/RAM o processo pode consumir.
- Kubernetes: orquestra, escala e coordena contêineres em produção.

**Protocolos de Acesso Remoto**
- SSH: linha de comando, criptografado nativamente, padrão-ouro de administração remota.
- VNC: desktop remoto gráfico (TCP 5900+), seguro só se tunelado via SSH.
- X11: renderiza GUI remota localmente, tráfego não criptografado por padrão (portas 6000–6009), exige `X11Forwarding`.
- XDMCP: protocolo legado (UDP 177), altamente vulnerável a MITM.

**Redes, Firewalls e Monitoramento**
- Diagnóstico: `netstat`/`ss` (conexões ativas), `traceroute` (rota via TTL).
- iptables/Netfilter: Tabelas → Cadeias (`INPUT`, `OUTPUT`) → Regras → Alvos (`ACCEPT`, `DROP`).
- Exposição de serviço local: dentro da LAN é automático; fora da rede exige port forwarding, IP público/DNS dinâmico ou túnel reverso.
- Logs críticos pra pentest: `/var/log/auth.log` (login SSH/sudo), `/var/log/syslog` (eventos gerais), `/var/log/kern.log` (kernel, drivers).

**Hardening e Controle de Acesso**
- DAC: dono do arquivo define permissões.
- MAC: SO impõe regras rígidas via rótulos (uso militar/governamental).
- RBAC: permissões atreladas ao cargo/função.
- SELinux / AppArmor: implementam MAC no kernel, confinando processos.
- TCP Wrappers: controle local por host via `/etc/hosts.allow` e `/etc/hosts.deny`, escopo diferente do NAC.

**Hierarquia do sistema**
- Kernel (Linux) → Distro (Arch, Ubuntu, Pop!_OS...) → Ambiente gráfico (GNOME, KDE, XFCE...).

### Glossário

| Termo | Definição 

| AppArmor | LSM que restringe recursos de aplicativos via perfis. 
| cgroups | Recurso de kernel que limita/isola uso de CPU e memória de processos. 
| Daemon | Serviço em background, geralmente termina em "d" (ex: `sshd`). 
| Distro | Kernel Linux + ferramentas, gerenciador de pacotes e configurações padrão (ex: Arch, Ubuntu). 
| DE (Desktop Environment) | Camada gráfica que roda sobre a distro (ex: GNOME, KDE) — independente de qual distro está por baixo. 
| Inode | Estrutura que guarda metadados de um arquivo (permissões, tamanho) — não o nome nem o conteúdo. 
| Namespace | Abstração de kernel que isola recursos globais (PIDs, rede) — base da conteinerização. 
| NAT | Tradução de endereço que esconde IPs privados de uma rede atrás de um IP público do roteador. 
| Netfilter | Framework de kernel para filtragem de pacotes; base do `iptables`. 
| SELinux | Sistema MAC de controle de acesso em nível de kernel, alta granularidade. 

### Banco de prompts reutilizáveis

Prompts testados nesta sessão, com placeholder pra reusar em qualquer tema novo.

**Pra comparar tecnologias parecidas**
```
Compare [TECNOLOGIA A], [TECNOLOGIA B] e [TECNOLOGIA C] sob os seguintes critérios:
1) o problema específico que cada uma resolve,
2) em que cenário eu escolheria uma em vez da outra,
3) o que acontece se eu usar a errada.
Apresente em formato de tabela.
```
Validado em: SSH vs VNC vs X11 vs XDMCP; LXC vs Docker.

**Pra aprofundar em pentest/segurança**
```
Considerando [CONCEITO/FERRAMENTA], explique:
1) como um atacante exploraria isso na prática (só o racional, sem payload funcional),
2) quais logs ou evidências esse ataque deixaria no sistema,
3) como eu, como defensor, detectaria e mitigaria isso.
```
Validado em: `auth.log` e modelos NAC/MAC/RBAC.

**Pra confrontar a própria hipótese**
```
Eu acho que [CONCEITO A] e [CONCEITO B] fazem a mesma coisa, porque [SUA TEORIA].
Isso está certo? Se não, aponte exatamente onde minha teoria erra e me dê
um exemplo prático onde os dois se comportariam de forma diferente.
```
Validado em: NAC vs TCP Wrappers; kernel vs distro vs ambiente gráfico.

**Pra forçar profundidade técnica a partir de uma fonte específica**
```
Resuma especificamente o trecho abaixo, sem generalizar pro tema mais amplo:
[COLAR TRECHO DA DOCUMENTAÇÃO]
```
Validado em: documentação oficial do LXC (cgroups e namespaces).

**Pra entender comandos prontos linha por linha**
```
Quebre o comando abaixo parte por parte, explicando o que cada flag/argumento faz
e o que aconteceria se eu omitisse cada uma delas individualmente:
[COLAR COMANDO]
```

**Pra criar laboratório guiado**
```
Crie um mini-laboratório passo a passo pra eu praticar [CONCEITO/FERRAMENTA] em um
ambiente local seguro (VM, Docker, HTB Academy/PortSwigger Academy). Inclua:
pré-requisitos, passos numerados, resultado esperado em cada etapa e um
"ponto de verificação" pra eu saber se entendi certo antes de avançar.
```
