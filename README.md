# Monitoramento da Sala Vermelha — UPAs BH

Sistema baseado no PAP DAUE 018 para substituir a planilha Google Sheets atual.

## O que tem aqui

- `schema.sql` — banco de dados (Supabase): tabelas, view com as regras do PAP
  (classificação Verde/Amarelo/Vermelho/Preto, alerta de dado desatualizado,
  bloqueio de "unidade com só 1 vaga não pode ser referência") e permissões.
- `lancamento.html` — página que cada UPA usa para registrar a ocupação.
- `dashboard.html` — página de visão consolidada, em tempo real, para o
  regulador do SAMU e demais interessados.

## Passo a passo

### 1. Criar o projeto Supabase (ou reaproveitar o que você já usa nas escalas)

Se quiser manter separado do sistema de escalas, crie um projeto novo em
supabase.com. Se preferir, pode usar o mesmo projeto — as tabelas têm nomes
próprios (`unidades`, `registros_ocupacao`) e não colidem com as do sistema
de escalas.

### 2. Rodar o schema

No painel do Supabase → **SQL Editor** → cole o conteúdo de `schema.sql` →
Run. Isso já cria as 9 UPAs com valores de leitos de exemplo — **ajuste os
números de `leitos_totais` na tabela `unidades`** para os valores reais de
cada unidade antes de usar em produção.

### 3. Pegar as chaves do projeto

Em **Project Settings → API**, copie:
- `Project URL`
- `anon public key`

Cole essas duas informações em **ambos** os arquivos HTML, no topo do
`<script>`, substituindo `SEU-PROJETO` e `SUA-ANON-KEY`.

### 4. Hospedar no GitHub Pages

Suba os dois arquivos `.html` para um repositório e ative o GitHub Pages
(Settings → Pages → branch `main`, pasta raiz). Vai gerar uma URL do tipo:

```
https://seu-usuario.github.io/upa-dashboard/
```

### 5. Gerar os links de lançamento para cada UPA

Cada unidade recebe um link fixo assim:

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

Envie o link correspondente a cada UPA (WhatsApp institucional, e-mail, ou
salvo como atalho na tela do celular/computador de plantão).

### 6. Link do dashboard

Um único link, para todos que precisam visualizar (regulador do SAMU,
gerência, DAUE):

```
.../dashboard.html
```

## Sobre segurança (leia)

Na versão atual, quem tem o link de lançamento de uma UPA consegue enviar
dados para aquela UPA — não há login. Isso reproduz o mesmo nível de
confiança da planilha atual (quem tem acesso, edita). Se depois você quiser
reforçar, dá para:

- Trocar para Supabase Auth com um usuário por unidade, reaproveitando o
  mesmo padrão de autenticação/RLS que você já implementou no sistema de
  escalas; ou
- Colocar uma Edge Function no meio do caminho para validar um token por
  unidade antes de gravar.

Nenhuma das duas é necessária para começar a usar.

## O que o sistema automatiza (e o que continua manual)

**Automatiza:**
- Cálculo de % de ocupação e classificação por nível
- Alerta visual de dado desatualizado (regra do item 4.2 do PAP)
- Alerta de "não indicar como referência" quando só resta 1 vaga
- Consolidação em tempo real, visível para todos no mesmo link
- Histórico completo (toda inserção fica registrada, nunca é sobrescrita)

**Continua manual (o PAP exige contato humano, não dá pra automatizar):**
- Comunicação ao Responsável Técnico Médico (Amarelo)
- Contato telefônico com o 192 (Vermelho e Preto)
- Aviso no grupo institucional de WhatsApp (Preto)
- Acionamento da Central de Internação — CINT (Preto)

O dashboard só acelera a decisão de quem vê e liga; ele não substitui essas
comunicações previstas no procedimento.
