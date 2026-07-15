---
description: Planeja o calendário mensal de posts do cliente e executa a geração em lote via /criar-post-instagram e /criar-carrossel-instagram
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

Antes de executar, confirmar que `{cliente}/brand-profile.md` está acessível (schema completo em `_sistema/referencias/brand-profile-conteudo-visual.md`) — identidade visual, vocabulário do público, CTA padrão e restrições de marca estão neste arquivo. As skills `/criar-post-instagram` e `/criar-carrossel-instagram`, chamadas no Passo 4, carregam o brand-profile como pré-requisito próprio.

---

## Gatilho

Usuário quer planejar vários posts de um cliente de uma vez. Variações: "planeja os posts da semana", "faz o calendário de julho", "cria 5 posts", "programação mensal de conteúdo", "posts em lote".

---

## Por que esta skill existe

Criar posts um a um é lento. Esta skill automatiza o planejamento mensal completo: mapeia datas, sugere temas e layouts, apresenta o plano para aprovação e executa a geração de todos os posts em lote — cada um com PNG pronto, sem ferramenta externa.

---

## Acesso ao vault

- **Lê:** `{cliente}/brand-profile.md`
- **Lê:** `_sistema/referencias/templates-post-instagram.md` (matriz de escolha de layout — tipos A–H/T — e "Família Carrossel" pra papéis de slide)
- **Lê:** `{cliente}/keywords-seo-gmn.md` (seções Tendências + Cauda Longa — Passo 2)
- **Chama:** `/criar-post-instagram` como skill (briefing pré-preenchido — sem perguntar ao usuário), para posts de slide único
- **Chama:** `/criar-carrossel-instagram` como skill (briefing pré-preenchido — sem perguntar ao usuário), para posts marcados como carrossel no plano
- **Escreve:** `{cliente}/Posts/CALENDÁRIO OFICIAL – [MÊS].md`
- **Cria:** `{cliente}/Posts/AAAA-MM/` + arquivos `.png` de cada post único, ou subpasta com `slide_XX.png` de cada carrossel

---

## Fluxo de execução

### Passo 1 — Coleta

Perguntar em uma única mensagem (não picotar):

- Período: semana, quinzena ou mês? (qual exatamente — ex: "julho 2026")
- Volume: quantos posts por semana? (padrão: 3)
- Eventos especiais: alguma conquista de cliente? Data comemorativa? Campanha ativa?
- Formatos predominantes: post único (feed 4:5), carrossel, quadrado ou stories? (padrão: feed 4:5 — carrossel é recomendado pra temas educativos/listicle com mais de 2–3 pontos, ver "Quantidade recomendada de slides" em `templates-post-instagram.md`)

### Passo 2 — Geração do plano

Com as respostas:

1. Identificar datas comemorativas relevantes ao segmento do cliente (ver "Contexto do cliente" no brand-profile — ex: para uma autoescola, datas de trânsito; para uma clínica, datas de saúde)
2. Consultar `{cliente}/keywords-seo-gmn.md` — seções "Tendências em Crescimento" e "Cauda Longa — Prioridade de Conversão" — para priorizar temas com alta intenção de busca:
   - Incluir ao menos 1 tema de tendência por quinzena
   - Incluir ao menos 1 tema de cauda longa por quinzena
   - Marcar esses temas com ⭐ na tabela para o usuário identificar rapidamente
3. Mapear mix de tipos de post por semana:
   - **Post único** — usar as 4 categorias de sempre: Conquista/prova social (protagonista nomeado), Educativo (dica, segurança, legislação), Institucional/campanha (data especial), Engajamento (reflexão, curiosidade, interação)
   - **Carrossel** — usar as 5 categorias próprias de `/criar-carrossel-instagram` (não as 4 acima, são taxonomias diferentes): Educativo, Depoimento, Dicas/Listicle, Marco Numérico, Data Comemorativa. Nem todo tema de post único vira carrossel de forma direta: "Conquista/prova social" isolada normalmente fica melhor como post único (Layout D); só considerar carrossel se o conteúdo render pra uma dessas 5 categorias (ex: depoimento longo → Depoimento; vários erros/dicas → Dicas/Listicle)
4. Para posts de **formato único**, consultar a "Matriz de escolha de layout" em `_sistema/referencias/templates-post-instagram.md` e sugerir o tipo adequado (A, A2, B, C, D, E, E2, F, G, H ou T) para cada post. Para posts de **formato carrossel**, sugerir o número de slides e o papel de cada um (Capa / Desenvolvimento / Exemplo / Checklist / Fechamento-CTA — ver "Família Carrossel" no mesmo doc), em vez de um único layout
5. Apresentar tabela para aprovação:

```
| # | Data | Tema | Tipo | Formato | Layout/Slides | Protagonista |
|---|------|------|------|---------|----------------|--------------|
| 1 | 22/07 | Conquista de Maria Silva | Conquista | Post único | D — Conquista/Prova social | Maria Silva |
| 2 | 24/07 | Dica de segurança | Educativo | Post único | B — Educativo | — |
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

Para cada post na tabela aprovada:

1. Montar briefing completo a partir da linha da tabela — contexto, protagonista, formato, layout/slides, copy já definidos na etapa de planejamento
2. Executar a skill correspondente ao **Formato** da linha, sem perguntar ao usuário (o briefing da tabela já responde todas as perguntas de coleta da skill atômica):
   - **Post único** → `/criar-post-instagram`, passando o layout (A–H/T) definido no plano
   - **Carrossel** → `/criar-carrossel-instagram`, passando tipo de conteúdo (uma das 5 categorias do Passo 2), tema e número de slides definidos no plano. **CTA:** usar o CTA padrão do brand-profile por padrão (a tabela não tem coluna própria de CTA); só passar um CTA diferente se o usuário tiver indicado um específico para aquela linha na revisão do Passo 3
3. Ao final de cada item informar:
   - Post único: `Post [#] gerado → {cliente}/Posts/AAAA-MM/nome.png`
   - Carrossel: `Post [#] gerado (carrossel, N slides) → {cliente}/Posts/AAAA-MM/<data>-<tema>/`
4. Continuar para o próximo
5. Se um post falhar: registrar "❌ FALHOU" e continuar os demais — não interromper o lote

### Passo 5 — Output final

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

| # | Data | Tema | Tipo | Formato | Layout/Slides | Protagonista | Arquivo |
|---|------|------|------|---------|----------------|--------------|---------|
| 1 | 22/07 | Conquista de Maria Silva | Conquista | Post único | D | Maria Silva | Posts/2026-07/post-01-conquista-maria.png |
| 2 | 24/07 | Dica de segurança | Educativo | Post único | B | — | Posts/2026-07/post-02-educativo-dica.png |
| 3 | 26/07 | 5 erros comuns | Dicas/Listicle | Carrossel (6 slides) | Capa→4 itens→CTA | — | Posts/2026-07/2026-07-26-5erros/ |

## Legendas

### Post 1 — 22/07 — Conquista de Maria Silva

[copiado de Posts/2026-07/post-01-conquista-maria/legenda.md]

### Post 2 — 24/07 — Dica de segurança

[copiado de Posts/2026-07/post-02-educativo-dica/legenda.md]
```

**Nunca gerar o texto da legenda de novo nesta etapa.** `/criar-post-instagram` e `/criar-carrossel-instagram` já criam `legenda.md` na pasta de cada post/carrossel (Passo 2.7 e Passo 1.10 delas, respectivamente) — a seção "Legendas" deste calendário é uma cópia literal do conteúdo desses arquivos, não uma nova geração. Isso evita que a legenda do calendário e a da pasta do post divirjam com o tempo.

3. Apresentar tabela final com status (✅ gerado / ❌ falhou)
4. Informar: "[N] posts gerados em `{cliente}/Posts/AAAA-MM/`"

---

## Restrições

- Respeitar a Regra de fundo e a paleta definidas no `{cliente}/brand-profile.md` — não assumir um padrão fixo de cor nem de fundo, cada cliente define a própria regra
- Se o formato escolhido for carrossel, respeitar a quantidade recomendada de slides por tipo (ver "Quantidade recomendada de slides" em `templates-post-instagram.md` — evitar acima de 8)
- Não avançar para Passo 4 sem aprovação explícita do plano no Passo 3
- Se um post falhar na geração, registrar e continuar — não interromper o lote inteiro
