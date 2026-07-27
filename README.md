# Monitoramento de Ocupação — UPAs BH (Sala Vermelha + Enfermaria)

Sistema baseado no PAP DAUE 018, estendido para também monitorar as
Enfermarias das mesmas 9 unidades.

## O que tem aqui

- `schema.sql` — banco de dados (Supabase). **Reescrito do zero** em
  relação à versão anterior — rode num projeto novo (recomendado: já na
  conta institucional).
- `lancamento.html` — página de registro. Agora tem um seletor no topo
  para escolher **Sala Vermelha** ou **Enfermaria** antes de preencher.
- `dashboard.html` — visão consolidada, agora com abas para os dois tipos,
  alerta de pendências no topo, e destaque visual de unidades atrasadas.
- `relatorios.html` — nova página para extrair relatórios de ocupação por
  período (semanal / mensal / semestral / anual / customizado), com
  exportação em CSV.

## O que mudou desde a versão anterior

### 1. Sala Vermelha + Enfermaria (mesma lógica, dois "tipos")
A lógica da Sala Vermelha **não foi alterada** — mesmos níveis (Verde/
Amarelo/Vermelho/Preto), mesma regra do "1 leito livre não pode ser
referência". A Enfermaria roda em paralelo, com sua própria capacidade de
leitos e seu próprio cronograma.

Cada unidade continua com **um único link** de lançamento
(`lancamento.html?u=barreiro`) — dentro da página, a pessoa escolhe qual
dos dois tipos está preenchendo.

### 2. Cronograma de atraso por tipo
- **Sala Vermelha:** mantém o padrão a cada 2h (00, 02, 04… 22h), com 15
  minutos de tolerância — igual já era.
- **Enfermaria:** cronograma próprio, só **9h e 20h**, com 45 minutos de
  tolerância (esse intervalo é ajustável na função `fn_tolerancia` do
  `schema.sql`, caso queira mais ou menos rigor).

O dashboard mostra, para cada card, se o último lançamento está **"No
horário"** ou **"Atrasado"** — e quem atrasa fica com destaque visual
(contorno vermelho no card) e um selo "⏱ atrasado". Isso deve ajudar nas
cobranças às unidades que atrasam com frequência, já que também aparece
o nome e cargo de quem fez o último lançamento.

### 3. Alerta de pendências
Um banner vermelho aparece no topo do dashboard sempre que houver
qualquer unidade (Sala Vermelha ou Enfermaria) com lançamento atrasado —
não precisa entrar na aba certa para perceber.

### 4. Lotação acima da capacidade
O campo de pacientes internados não tem mais limite máximo — dá para
lançar um número maior que os leitos cadastrados. Quando isso acontece:
- A tela de lançamento mostra um alerta imediato ("lotação acima da
  capacidade em X paciente(s)").
- O card no dashboard ganha o selo "⚠ acima da capacidade".
- Vagas aparecem como número negativo, destacado em vermelho.
- Cada ocorrência fica registrada no histórico normalmente — nada é
  bloqueado, só sinalizado.

### 5. Relatórios (`relatorios.html`)
Nova página, acessível pelo link "📊 Relatórios" no dashboard. Escolha um
período (semanal = últimos 7 dias, mensal = últimos 30, semestral = 6
meses, anual = 12 meses, ou datas customizadas) e o tipo (Sala Vermelha,
Enfermaria, ou os dois). O relatório mostra, por unidade:
- número de registros no período
- ocupação média e máxima (%)
- quantos registros ficaram acima da capacidade
- número máximo de pacientes já registrado

Dá para baixar tudo em CSV. Esse cálculo roda **dentro do banco**
(função `relatorio_ocupacao`), então funciona mesmo com um ano inteiro de
histórico sem sobrecarregar o navegador.

### 6. Nome e cargo obrigatórios
A tela de lançamento agora exige nome e cargo de quem está preenchendo —
o botão de enviar valida isso antes de gravar, e o banco também recusa
(colunas `NOT NULL`) caso algo tente pular essa validação.

### 7. Visual
A borda colorida lateral dos cards ficou mais grossa (de 5px para 10px),
como pedido.

## Passo a passo de instalação

### 1. Criar o projeto Supabase (conta institucional)

supabase.com → **New Project**. Guarde a senha do banco (não é usada no
dia a dia, só em emergências).

### 2. Rodar o schema

**SQL Editor** → cole o `schema.sql` inteiro → **Run**.

Isso cria as 9 UPAs, a capacidade de leitos de Sala Vermelha (valores já
usados antes) e de Enfermaria (**valores de exemplo, 20 para todas — vá
em Table Editor → `capacidade_leitos` e ajuste para os números reais**).

### 3. Pegar as chaves

**Project Settings → API** → copie a `Project URL` e a chave pública
(`anon` / `publishable`).

Cole essas duas informações nos **três** arquivos HTML
(`lancamento.html`, `dashboard.html`, `relatorios.html`), no topo do
`<script>`.

### 4. Hospedar no GitHub Pages

Suba os quatro arquivos (os três `.html` + este `README.md`, opcional)
para o repositório institucional e ative o GitHub Pages.

### 5. Links de lançamento (iguais a antes, um por UPA)

```
.../lancamento.html?u=barreiro
.../lancamento.html?u=centro_sul
.../lancamento.html?u=leste
.../lancamento.html?u=nordeste
.../lancamento.html?u=noroeste
.../lancamento.html?u=norte
.../lancamento.html?u=oeste
.../lancamento.html?u=pampulha
.../lancamento.html?u=venda_nova
```

Cada link agora atende tanto Sala Vermelha quanto Enfermaria — a pessoa
escolhe dentro da página.

### 6. Dashboard e relatórios

```
.../dashboard.html
.../relatorios.html
```

## Sobre a heartbeat / pausa por inatividade

Continua valendo a recomendação de configurar um "ping" automático
(GitHub Actions, 1x por dia) para o projeto Supabase não pausar por
inatividade — ainda mais importante agora que existe um segundo
cronograma (Enfermaria, só 2x/dia), que já é naturalmente mais espaçado.

## Segurança (sem mudanças em relação à versão anterior)

Continua o mesmo modelo: quem tem o link de uma unidade pode lançar dados
para ela, sem autenticação adicional — reproduz o nível de confiança da
planilha atual. Reforços possíveis (Supabase Auth por unidade, ou uma
Edge Function validando token) continuam sendo incrementos futuros, não
bloqueadores para uso.
