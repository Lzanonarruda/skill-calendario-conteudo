---
description: Planeja o calendário mensal de posts do cliente e executa a geração em lote via /criar-post-instagram, /criar-carrossel-instagram, /criar-copy-gmn e /criar-thumbnail-gmn
when_to_use: calendário de conteúdo, planejar posts do mês, lote de posts, semana de conteúdo, posts da semana, programação mensal
---

# Skill: Calendário de Conteúdo

## Pré-requisito 0 — Qual cliente

1. Se o usuário nomear o cliente/pasta explicitamente na mensagem → usar essa pasta como `{cliente}`.
2. Senão, procurar as pastas de topo do vault que contêm `brand-profile.md` na raiz:
   - Exatamente 1 encontrada → usar essa, sem perguntar.
   - 0 encontradas → perguntar ao usuário o nome do cliente/pasta (esperado: `{pasta}/brand-profile.md`).
   - 2+ encontradas → perguntar qual cliente usar antes de prosseguir.
3. Todos os paths e exemplos deste documento usam `{cliente}` como placeholder.

## Pré-requisito — Brand Profile

Antes de executar, confirmar que `{cliente}/brand-profile.md` está acessível (schema completo em `_sistema/referencias/brand-profile-conteudo-visual.md`) — identidade visual, vocabulário do público, CTA padrão e restrições de marca estão neste arquivo. As skills `/criar-post-instagram`, `/criar-carrossel-instagram`, `/criar-copy-gmn` e `/criar-thumbnail-gmn`, chamadas no Passo 4, carregam o brand-profile como pré-requisito próprio.

---

## Gatilho

Usuário quer planejar vários posts de um cliente de uma vez. Variações: "planeja os posts da semana", "faz o calendário de julho", "cria 5 posts", "programação mensal de conteúdo", "posts em lote".

---

## Por que esta skill existe

Criar posts um a um é lento. Esta skill automatiza o planejamento mensal completo: mapeia datas, sugere temas e layouts, apresenta o plano para aprovação e executa a geração de todos os posts em lote — cada um já com PNG de Instagram, copy de GMN e thumbnail de GMN prontos, sem ferramenta externa.

---

## Acesso ao vault

- **Lê:** `{cliente}/brand-profile.md`
- **Lê:** `_sistema/referencias/templates-post-instagram.md` (matriz de escolha de layout — tipos A–H/T — e "Família Carrossel" pra papéis de slide)
- **Lê:** `{cliente}/keywords-seo-gmn.md` (seções Tendências + Cauda Longa — Passo 2)
- **Chama:** `/criar-post-instagram` como skill (briefing pré-preenchido — sem perguntar ao usuário), para posts de slide único
- **Chama:** `/criar-carrossel-instagram` como skill (briefing pré-preenchido — sem perguntar ao usuário), para posts marcados como carrossel no plano
- **Chama:** `/criar-copy-gmn` como skill, pra todo post do lote, depois da arte de Instagram gerada
- **Chama:** `/criar-thumbnail-gmn` como skill, pra todo post do lote, depois da copy de GMN gerada
- **Lê:** `{cliente}/Posts/CALENDÁRIO OFICIAL – [MÊS].md` do mês corrente e, quando a janela de 15 dias cruzar a virada do mês, também o do mês anterior (Passo 2.0 — anti-repetição de tema)
- **Escreve:** `{cliente}/Posts/CALENDÁRIO OFICIAL – [MÊS].md` — inclui a seção `## Google Meu Negócio`, escrita automaticamente por `/criar-copy-gmn` durante o lote, não por esta skill diretamente
- **Cria:** `{cliente}/Posts/AAAA-MM/` + arquivos `.png` de cada post único, ou subpasta com `slide_XX.png` de cada carrossel; `{cliente}/GMN/Thumbnails/<AAAA-MM-DD>-<tema>/` + `copy.md` e `thumbnail.png` de cada post

---

## Fluxo de execução

### Passo 1 — Coleta

Perguntar em uma única mensagem (não picotar):

- Objetivo do período: qual é o foco principal agora — captação de clientes novos, fidelização de quem já compra, lançamento de produto/serviço, ou institucional/manutenção de presença? Pergunta aberta, não múltipla escolha fechada: se a resposta for híbrida ou não encaixar limpo em nenhuma dessas, perguntar como o usuário quer ponderar a distribuição de categorias em vez de forçar encaixe. Nunca assumir um objetivo padrão sem perguntar
- Período: semana, quinzena, mês ou intervalo de dias específico? (qual exatamente — ex: "julho 2026", "22 a 27/07", "próximos 6 dias")
- Volume: quantos posts no total, ou por semana? (padrão: 3 por semana corrida). Se o período informado não for uma semana/quinzena/mês inteiros, calcular o volume proporcional ao padrão semanal (3 posts/semana) arredondando pro inteiro mais próximo e informar esse cálculo ao usuário antes de seguir — a menos que ele já tenha dito quantos posts quer no total
- Eventos especiais: alguma conquista de cliente? Data comemorativa? Campanha ativa?
- Formatos predominantes: post único (feed 4:5), carrossel, quadrado ou stories? (padrão: feed 4:5 — carrossel é recomendado pra temas educativos/listicle com mais de 2–3 pontos, ver "Quantidade recomendada de slides" em `templates-post-instagram.md`)

### Passo 2 — Geração do plano

**Passo 2.0 — Verificar temas recentes (anti-repetição, janela de 15 dias).** Antes de sugerir qualquer tema:

1. Calcular a janela: pegar a data mais antiga do lote que está sendo planejado (Passo 1) e voltar 15 dias — esse é o limite inferior
2. Ler `{cliente}/Posts/CALENDÁRIO OFICIAL – [MÊS].md` do mês corrente. Se o limite inferior cair no mês anterior (ex: planejando 1–6/08, a janela volta até 17/07), ler também o calendário desse mês anterior
3. Se nenhum desses arquivos existir ainda (cliente novo, ou primeiro calendário gerado para ele), pular esta verificação silenciosamente — sem erro, sem bloquear o fluxo
4. Extrair as colunas Data + Tema de cada linha, mantendo só as linhas cuja Data cai dentro da janela de 15 dias — essa é a lista de "temas recentes"
5. Ao montar cada tema novo no restante do Passo 2, avaliar semanticamente (não por comparação literal de texto) se ele cobre o mesmo assunto/ângulo de algum item da lista de temas recentes. Pergunta-guia: *"um seguidor que viu os dois posts sentiria que é o mesmo conteúdo reciclado com outras palavras?"* Se sim, é repetição — descartar esse tema e escolher outro. Mesmo tópico geral abordado por ângulo genuinamente diferente (ex: "dica de manutenção preventiva" vs. "erro comum que causa multa" no mesmo nicho) não conta como repetição
6. Se não sobrar alternativa plausível pra alguma categoria depois de aplicar a exclusão, não forçar a repetição em silêncio — sinalizar isso ao usuário na tabela do Passo 3, junto com o tema recente que colidiria, para ele decidir

Com as respostas:

1. Identificar datas comemorativas relevantes ao segmento do cliente (ver "Contexto do cliente" no brand-profile)
2. Consultar `{cliente}/keywords-seo-gmn.md` — seções "Tendências em Crescimento" e "Cauda Longa — Prioridade de Conversão" — para priorizar temas com alta intenção de busca:
   - Essas duas listas abastecem preferencialmente as categorias que naturalmente comportam keyword: **Educativo** (post único e carrossel) e **Dicas/Listicle** (carrossel). Nelas, priorizar sempre um tema de tendência ou cauda longa ainda não usado no período corrente, em vez de criar um tema livre — só cair para tema livre nessas categorias quando as duas listas se esgotarem
   - Categorias que não combinam com keyword por natureza — Conquista/prova social (protagonista nomeado), Institucional/campanha (data especial), Engajamento (reflexão/curiosidade) — continuam com tema livre, sem obrigação de keyword
   - Marcar todo tema de tendência/cauda longa efetivamente usado com ⭐ na tabela, para o usuário identificar rapidamente
3. Mapear mix de tipos de post por semana:
   - **Post único** — usar as 4 categorias de sempre: Conquista/prova social (protagonista nomeado), Educativo (dica, boas práticas, informação relevante ao segmento), Institucional/campanha (data especial), Engajamento (reflexão, curiosidade, interação)
   - **Carrossel** — usar as 5 categorias próprias de `/criar-carrossel-instagram` (não as 4 acima, são taxonomias diferentes): Educativo, Depoimento, Dicas/Listicle, Marco Numérico, Data Comemorativa. Nem todo tema de post único vira carrossel de forma direta: "Conquista/prova social" isolada normalmente fica melhor como post único (Layout D); só considerar carrossel se o conteúdo render pra uma dessas 5 categorias (ex: depoimento longo → Depoimento; vários erros/dicas → Dicas/Listicle)
   - **Nota de classificação — homonímia entre taxonomias:** "Educativo" existe nas duas listas acima com o mesmo nome, mas não é a mesma categoria — post único Educativo vira Layout B; carrossel Educativo vira uma sequência própria de slides. Mesmo rótulo, execução diferente. Sempre checar qual das duas listas está em uso (definida pelo Formato da linha) antes de interpretar o valor de Tipo
   - **Ponderar pelo objetivo do período (Passo 1):** ajustar a proporção das categorias conforme o objetivo informado — captação: priorizar Educativo + Oferta; fidelização: priorizar Depoimento/Conquista + Engajamento; lançamento: priorizar Institucional/Campanha + Oferta; institucional/manutenção: manter o mix padrão equilibrado. Isso é peso de sugestão pro plano inicial, não regra rígida — o usuário ainda pode reequilibrar linha por linha no Passo 3
   - **Cadência entre categorias:** ao ordenar os posts por Data, evitar 2 posts consecutivos com o mesmo Tipo **e** mesmo Formato (ex: dois post único Educativo seguidos) — mudar o Formato já quebra a sensação de repetição mesmo com Tipo igual. Se o mix indicar repetição do mesmo par Tipo+Formato, intercalar com pelo menos 1 post diferente entre eles. Exceção: períodos muito curtos (2–3 posts) onde a repetição é inevitável dado o mix necessário — sinalizar isso ao usuário na tabela do Passo 3 em vez de forçar uma categoria artificial só pra quebrar a sequência
4. Para posts de **formato único**, consultar a "Matriz de escolha de layout" em `_sistema/referencias/templates-post-instagram.md` e sugerir o tipo adequado (A, A2, B, C, D, E, E2, F, G, H ou T) para cada post. Para posts de **formato carrossel**, sugerir o número de slides e o papel de cada um (Capa / Desenvolvimento / Exemplo / Checklist / Fechamento-CTA — ver "Família Carrossel" no mesmo doc), em vez de um único layout
5. Apresentar tabela para aprovação:

```
| # | Data | Tema | Tipo | Formato | Layout/Slides | Protagonista |
|---|------|------|------|---------|----------------|--------------|
| 1 | 22/07 | Conquista de Maria Silva | Conquista | Post único | D — Conquista/Prova social | Maria Silva |
| 2 | 24/07 | Dica prática do segmento | Educativo | Post único | B — Educativo | — |
| 3 | 26/07 | 5 erros comuns | Dicas/Listicle | Carrossel (6 slides) | Capa → 4 itens → Fechamento/CTA | — |
| 4 | 28/07 | Data comemorativa | Institucional | Post único | F — Data comemorativa | — |
```

### Passo 3 — Revisão do plano

Aguardar o usuário:
- Aprovar o plano completo → avançar para Passo 4
- Editar linhas (mudar tema, data, tipo, formato, layout/slides, protagonista)
- Remover ou adicionar posts

Não avançar sem confirmação explícita.

### Passo 4 — Execução em lote

**Passo 4.0 — Criar checklist de execução (obrigatório):** antes de iniciar o lote, criar uma lista TodoWrite com 1 item por post da tabela aprovada, cobrindo as 3 etapas daquele post (arte de Instagram + copy GMN + thumbnail GMN). Marcar `completed` só quando as 3 etapas do post tiverem rodado com sucesso — um post com qualquer etapa "❌ FALHOU" (ver item 7 abaixo) fica com o item pendente, não `completed`. Ver [[auditoria-execucao-skills]].

Para cada post na tabela aprovada, nesta ordem — arte de Instagram → copy GMN → thumbnail GMN — antes de passar pro próximo post da lista:

1. Montar briefing completo a partir da linha da tabela — contexto, protagonista, formato, layout/slides, copy já definidos na etapa de planejamento
2. **Arte de Instagram.** Executar a skill correspondente ao **Formato** da linha, sem perguntar ao usuário (o briefing da tabela já responde todas as perguntas de coleta da skill atômica):
   - **Post único** → `/criar-post-instagram`, passando o layout (A–H/T) definido no plano
   - **Carrossel** → `/criar-carrossel-instagram`, passando tipo de conteúdo (uma das 5 categorias do Passo 2), tema e número de slides definidos no plano. **CTA:** usar o CTA padrão do brand-profile por padrão (a tabela não tem coluna própria de CTA); só passar um CTA diferente se o usuário tiver indicado um específico para aquela linha na revisão do Passo 3
3. **Copy GMN.** Executar `/criar-copy-gmn` (Fluxo 1 dela, post específico) passando o mesmo tema e data da linha, sem perguntar ao usuário — sempre, pra todo post do lote, independente do tipo. Isso já escreve a seção `## Google Meu Negócio` no calendário e salva `copy.md` em `{cliente}/GMN/Thumbnails/<AAAA-MM-DD>-<tema>/`
4. **Thumbnail GMN.** Executar `/criar-thumbnail-gmn` passando o mesmo tema — sempre, pra todo post do lote, independente do tipo. Salva `thumbnail.png` na mesma pasta da copy
5. Ao final de cada item informar:
   - Post único: `Post [#] gerado → {cliente}/Posts/AAAA-MM/nome.png` + `GMN → {cliente}/GMN/Thumbnails/<data>-<tema>/`
   - Carrossel: `Post [#] gerado (carrossel, N slides) → {cliente}/Posts/AAAA-MM/<data>-<tema>/` + `GMN → {cliente}/GMN/Thumbnails/<data>-<tema>/`
6. Continuar para o próximo post
7. Se qualquer uma das 3 etapas (arte, copy, thumbnail) falhar: registrar "❌ FALHOU" só naquela etapa, seguir com as demais etapas do mesmo post se possível, e continuar pro próximo post — nunca interromper o lote inteiro por causa de uma etapa

### Passo 5 — Output final

**Auditoria antes de apresentar (obrigatório):** reconferir a lista TodoWrite do Passo 4.0 — se algum post ficou com item pendente (etapa "❌ FALHOU"), reexecutar a etapa faltante daquele post antes de declarar o lote concluído. Nunca apresentar o output final com posts incompletos sem sinalizar isso explicitamente ao usuário (ver [[auditoria-execucao-skills]]).

Ao concluir todos os posts:

1. Criar pasta `{cliente}/Posts/AAAA-MM/` se ainda não existir
2. Salvar tabela-resumo como `{cliente}/Posts/CALENDÁRIO OFICIAL – [MÊS].md`:

```markdown
---
tags: [calendario, conteudo]
periodo: [MÊS AAAA]
gerado: AAAA-MM-DD
---

# Calendário Oficial — [MÊS AAAA]

| # | Data | Tema | Tipo | Formato | Layout/Slides | Protagonista | Arquivo | GMN |
|---|------|------|------|---------|----------------|--------------|---------|-----|
| 1 | 22/07 | Conquista de Maria Silva | Conquista | Post único | D | Maria Silva | Posts/2026-07/post-01-conquista-maria.png | ✅ |
| 2 | 24/07 | Dica prática do segmento | Educativo | Post único | B | — | Posts/2026-07/post-02-educativo-dica.png | ✅ |
| 3 | 26/07 | 5 erros comuns | Dicas/Listicle | Carrossel (6 slides) | Capa→4 itens→CTA | — | Posts/2026-07/2026-07-26-5erros/ | ✅ |

## Legendas

### Post 1 — 22/07 — Conquista de Maria Silva

[copiado de Posts/2026-07/post-01-conquista-maria/legenda.md]

### Post 2 — 24/07 — Dica prática do segmento

[copiado de Posts/2026-07/post-02-educativo-dica/legenda.md]

## Google Meu Negócio

[esta seção é escrita por `/criar-copy-gmn` durante o Passo 4, não por esta skill — cada bloco `### Post N — Data — Tema` é inserido por ela mesma, no formato já documentado na skill]
```

**Nunca gerar o texto da legenda nem da copy GMN de novo nesta etapa.** `/criar-post-instagram` e `/criar-carrossel-instagram` já criam `legenda.md` na pasta de cada post/carrossel (Passo 2.7 e Passo 1.10 delas, respectivamente), e `/criar-copy-gmn` já escreve a seção `## Google Meu Negócio` sozinha durante o Passo 4. A seção "Legendas" deste calendário é uma cópia literal do conteúdo dos arquivos `legenda.md`, não uma nova geração — isso evita que a legenda do calendário e a da pasta do post divirjam com o tempo. A seção GMN nunca precisa dessa cópia manual porque já é escrita direto no lugar certo pela própria skill.

3. Apresentar tabela final com status por post e por etapa (✅ gerado / ❌ falhou) — a coluna **GMN** cobre copy + thumbnail juntos; se só uma das duas falhar, marcar "⚠️ parcial" e detalhar qual etapa faltou no texto de confirmação
4. Informar: "[N] posts gerados em `{cliente}/Posts/AAAA-MM/`, com copy e thumbnail de GMN em `{cliente}/GMN/Thumbnails/`"

---

## Restrições

- Respeitar a Regra de fundo e a paleta definidas no `{cliente}/brand-profile.md` — não assumir um padrão fixo de cor nem de fundo, cada cliente define a própria regra
- Se o formato escolhido for carrossel, respeitar a quantidade recomendada de slides por tipo (ver "Quantidade recomendada de slides" em `templates-post-instagram.md` — evitar acima de 8)
- Não avançar para Passo 4 sem aprovação explícita do plano no Passo 3
- Se um post falhar na geração (arte, copy GMN ou thumbnail GMN), registrar e continuar — não interromper o lote inteiro
- Copy GMN e thumbnail GMN são geradas pra todo post do lote, sem exceção por tipo — não pular essas duas etapas mesmo em posts onde pareçam menos óbvias (ex: avaliação/depoimento — nesse caso `/criar-copy-gmn` já sabe transformar o depoimento num ângulo educativo em vez de narrar a cena, conforme o caso especial documentado nela)
