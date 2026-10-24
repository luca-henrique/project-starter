# Guia de Arquitetura: Múltiplos Projetos Docker com Cloudflare Tunnel (Sem Conflitos de Porta)

Este guia explica o fluxo conceitual e prático para hospedar múltiplas aplicações (como Next.js e Node.js) que utilizam as mesmas portas internas (ex.: 3000 e 3333) no mesmo servidor Debian, sem colisões de porta e sem expor o servidor à internet direta.

---

## 1. O Problema: Conflito de Portas no Host

Normalmente, o conflito de portas (*port already in use*) ocorre quando duas ou mais aplicações tentam reivindicar a mesma porta física da máquina Debian (ex.: tentar vincular duas portas 3000 internas à mesma porta 3000 do servidor).

Para contornar isso, **nenhuma aplicação deve ser vinculada diretamente às portas do sistema operacional**. O tráfego será intermediado internamente via Docker e Cloudflare.

---

## 2. Passo a Passo Conceitual

### Passo 1: Criar uma Rede Virtual Compartilhada
Em vez de deixar os contêineres isolados em redes autônomas geradas por cada compose, cria-se uma rede virtual comum (do tipo *bridge*) dentro do Docker.
* **Função:** Atuar como uma rede interna privada (semelhante a uma rede local de computadores), visível apenas para os contêineres conectados a ela.
* **Benefício:** Qualquer serviço conectado a essa rede consegue se comunicar com outro usando apenas o nome atribuído ao contêiner.

---

### Passo 2: Isolar o Tráfego das Portas do Sistema Operacional
Em cada projeto:
* O contêiner roda internamente na sua porta nativa (ex.: `3000` para Next.js, `3333` para API).
* O mapeamento com o host é omitido (as portas não são expostas para a máquina física Debian).
* O contêiner é conectado à rede virtual compartilhada criada no Passo 1.
* **Resultado:** Como a porta fica contida dentro do contêiner, dezenas de aplicações podem rodar simultaneamente na porta `3000` sem nenhuma colisão.

---

### Passo 3: Atribuir Identificadores Únicos (Nomes de Contêiner)
Cada aplicação (frontend e backend) recebe um nome único de contêiner.
* O Docker possui um serviço de resolução de nomes interno (DNS interno).
* O contêiner do túnel usará esse nome para saber exatamente para onde encaminhar cada requisição (ex.: `projeto-a-front`, `projeto-a-api`, `projeto-b-front`, `projeto-b-api`).

---

### Passo 4: Conectar o Agente do Cloudflare Tunnel
Um contêiner dedicado rodando o agente da Cloudflare é iniciado dentro da mesma rede virtual compartilhada.
* **Segurança de borda:** O túnel estabelece uma conexão criptografada de dentro para fora com a infraestrutura da Cloudflare. Não há necessidade de abrir portas no firewall da máquina (`ufw`) ou liberar roteamento no roteador/provedor.
* **Visibilidade:** Por estar na mesma rede virtual, o túnel enxerga todos os outros contêineres diretamente pelos nomes definidos no Passo 3.

---

### Passo 5: Mapear os Subdomínios no Painel da Cloudflare
O roteamento final é configurado no painel da Cloudflare (ou via arquivo de configuração):
* Cada domínio ou subdomínio público é associado ao nome interno do contêiner e sua respectiva porta:
  * `app1.seudominio.com` $\rightarrow$ Encaminha para `projeto-a-front:3000`
  * `api1.seudominio.com` $\rightarrow$ Encaminha para `projeto-a-api:3333`
  * `app2.seudominio.com` $\rightarrow$ Encaminha para `projeto-b-front:3000`
  * `api2.seudominio.com` $\rightarrow$ Encaminha para `projeto-b-api:3333`

---

## 3. Fluxo de Uma Requisição

1. **Usuário:** Acessa `https://app1.seudominio.com` pelo navegador.
2. **Cloudflare Edge:** Recebe a requisição, aplica certificado SSL/TLS automático e regras de segurança.
3. **Túnel Criptografado:** A Cloudflare despacha o pacote pelo túnel direto até o contêiner do túnel no seu Debian.
4. **Resolução Interna:** O contêiner do túnel entrega a requisição para `http://projeto-a-front:3000` pela rede interna.
5. **Retorno:** A aplicação processa a resposta, que faz o caminho inverso de forma transparente.

---

## 4. Vantagens Desta Abordagem

* **Escalabilidade ilimitada:** Novos projetos podem ser adicionados sem precisar gerenciar portas livres (`3001`, `3002`, `3003`...).
* **Zero portas abertas no Debian:** O servidor fica invisível a varreduras de portas comuns na internet aberta.
* **Padronização:** Todos os projetos Next.js e Node mantêm suas configurações padrão de porta de desenvolvimento e produção.