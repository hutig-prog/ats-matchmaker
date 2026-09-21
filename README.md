# ATS Matchmaker

## Gerador de currículo ATS friendly - Vaga viva bot

Um currículo pode estar excelente e mesmo assim ser descartado antes de chegar ao RH, porque o ATS (Applicant Tracking System) ranqueia currículos automaticamente. A aplicação resolve isso: ela compara o currículo do usuário com a descrição de uma vaga, calcula o percentual de match, aponta as palavras-chave encontradas e as que faltam, e gera uma versão do currículo ATS-friendly pronta para copiar ou baixar — aumentando as chances de o candidato passar pelo filtro automático.

### Mega-Prompt:

# App: "Currículo vs Vaga" — Analisador ATS com IA (MVP)

## 1. Visão geral
Crie um aplicativo web responsivo, em português do Brasil, **sem login e sem cadastro**, que compara o currículo do usuário com a descrição de uma vaga, calcula o percentual de match, mostra as palavras-chave encontradas e as que faltam, e gera uma versão do currículo otimizada para sistemas ATS, pronta para copiar ou baixar como arquivo.

Contexto: recrutadores usam ATS (Applicant Tracking System) para ranquear currículos automaticamente. Um bom currículo para ATS precisa das palavras-chave certas e de formatação simples, sem colunas, tabelas ou imagens.

## 2. Regras de ouro (críticas)
- **NUNCA inventar nem adicionar** experiência profissional, formação, datas, empresas, certificações, idiomas ou habilidades que não existam no currículo enviado pelo usuário.
- A IA pode **reformular, reorganizar e reposicionar** informações que JÁ existem no currículo, inclusive usando a terminologia da vaga quando a competência equivalente já está presente.
- Toda sugestão de melhoria deve ser claramente marcada como sugestão ("Sugestão:", "Dica:"). Nada de conteúdo inventado dentro do currículo final.
- A versão ATS-friendly gerada deve conter **APENAS fatos do currículo original**, com redação otimizada e ordem profissional.

## 3. Fluxo do usuário (página única com estados)
**Etapa 1 — Entrada:**
- Área para colar ou enviar a descrição da vaga (arquivo PDF/DOCX/TXT ou texto colado).
- Área para colar ou enviar o currículo (arquivo PDF/DOCX/TXT ou texto colado).
- Botão principal "Analisar match".

**Etapa 2 — Analisando:**
- Estado de carregamento elegante ("Analisando seu currículo…"), 2 a 5 segundos.

**Etapa 3 — Resultado:**
- Score de match (0–100%) com barra ou anel suave e rótulo qualitativo (Alto / Médio / Baixo).
- Palavras-chave **ENCONTRADAS** (chips preenchidos suaves).
- Palavras-chave **FALTANTES** (chips de contorno discreto), cada uma com orientação: "se já tem a competência, use o termo da vaga no currículo" / "se não tem, prepare-se para a entrevista".
- Dicas de melhoria por seção (Resumo, Experiência, Habilidades, Formação) — sempre como sugestão, nunca inventando.
- Botão "Gerar versão ATS-friendly".

**Etapa 4 — Versão ATS-friendly:**
- Texto otimizado exibido como documento limpo, formato monocoluna, sem colunas, sem tabelas, sem imagens e sem gráficos (formatação compatível com ATS).
- Botão "Copiar texto" (copia para a área de transferência).
- Botão "Baixar DOCX" (gera arquivo .docx simples e ATS-safe).

## 4. Entrada de arquivos
- Upload com clique **e** arrastar e soltar (drag & drop).
- Aceitar .PDF, .DOCX e .TXT: PDF via extração de texto client-side; DOCX via extração de texto; TXT direto.
- Se o arquivo não tiver texto extraível (PDF escaneado ou imagem), mostrar mensagem amigável pedindo para colar o texto manualmente.
- Limite de ~2MB por arquivo, com aviso amigável se exceder.
- Ao usar arquivo, exibir um resumo do texto extraído para o usuário confirmar que a leitura ficou correta.

## 5. IA — Edge Function "ats-analyzer"
- Crie uma Supabase Edge Function chamada "ats-analyzer" que recebe `{ jobDescription, resumeText }` e retorna JSON.
- Use a API do Google Gemini (modelo gratuito, ex.: gemini-2.0-flash ou o equivalente disponível) com a chave da variável de ambiente `GOOGLE_AI_STUDIO_API_KEY` (secrets da Edge Function). A chave **nunca** deve aparecer no código do cliente.
- Use este system prompt:
"Você é um especialista em currículos e sistemas ATS. Você recebe a descrição de uma vaga e o texto de um currículo. Retorne um JSON válido com:
1. matchScore: número de 0 a 100 representando a compatibilidade.
2. keywordsFound: lista de palavras-chave da vaga presentes no currículo.
3. keywordsMissing: lista de palavras-chave da vaga ausentes no currículo, cada uma com uma sugestão breve de como agir (usar o termo já tendo a competência, ou se preparar para a entrevista).
4. tips: lista de 3 a 6 dicas objetivas de melhoria por seção, sempre como sugestões.
5. atsResumeText: o currículo original reescrito em formato otimizado para ATS — monocoluna, sem tabelas, com seções claras (Resumo, Experiência, Habilidades, Formação), usando a terminologia da vaga APENAS para competências já presentes no currículo original, e NUNCA inventando experiência, datas, empresas, certificações ou habilidades. Se uma palavra-chave da vaga não estiver coberta pelo currículo, NÃO a inclua no texto — ela já aparece em keywordsMissing.
Responda somente com o JSON, sem texto extra."
- Se a API falhar ou a chave não existir, retorne a resposta amigável: "Não consegui analisar agora. Tente novamente em instantes." (nunca tela de erro).
- (Opcional, se for simples) Mantenha um fallback por regras de palavras-chave para quando a API estiver indisponível.

## 6. Telas e navegação
- Página única em /, com estados de etapa (Entrada → Analisando → Resultado → Versão ATS), sem login e sem rotas complexas; os resultados aparecem na mesma página com rolagem.
- Barra leve de progresso entre etapas (1 Vaga, 2 Currículo, 3 Análise, 4 ATS), se ajudar na clareza.
- Cabeçalho com o nome do app ("Currículo vs Vaga" ou "Analisador de Currículo ATS") e subtítulo curto.
- Rodapé discreto: "Desafio DIO — Vibe Coding".

## 7. Design (crítico)
- Visual **simples, minimalista, intuitivo e elegante**.
- **Cores suaves e neutras**: fundo off-white (#FAFAF8), superfícies brancas (#FFFFFF), bordas finas em tons claros (#E6E2DC), texto em cinza-escuro suave (#2F2A26 / #6B6560), e um único tom de destaque discreto verde-sálvia (#8FAE9B) para botões, acentos e o score, com variações suaves para estados (âmbar-claro e vermelho-claro discretos para score médio/baixo).
- Tipografia limpa e legível (Inter ou fontes do sistema), hierarquia clara de títulos.
- Cantos levemente arredondados, sombras muito suaves, espaçamento generoso (bastante respiro).
- Sem poluição visual: uma ação principal por tela; botões com rótulos claros ("Analisar match", "Copiar texto", "Baixar DOCX").
- Chips/etiquetas: encontradas = preenchidas suaves; faltantes = contorno fino discreto.
- Responsivo: coluna única no celular; duas colunas lado a lado no desktop para vaga e currículo.
- Estado vazio amigável na entrada: "Cole a vaga e seu currículo para ver o match".

## 8. Detalhes técnicos (organização)
- Componentes em src/components/: JobInput, ResumeInput, MatchScore, KeywordsList, TipsList, ATSResumeView, ExportButtons.
- Lógica isolada em src/lib/: extract-text.ts (extração de texto dos arquivos), analyze.ts (chamada à Edge Function e parse do JSON), ats-utils.ts (geração do DOCX e cópia para a área de transferência).
- Tokens de cor como variáveis CSS em src/styles.css; componentes usam SOMENTE tokens semânticos (--color-bg, --color-surface, --color-text, --color-accent, etc.).
- head() com título e descrição próprios.
- Estado de carregamento com feedback visual suave.
- Nada de autenticação, nada de planos pagos, nada de telas de cadastro.
- O app deve ser publicável pelo próprio Lovable com endereço próprio (deploy gratuito) e pronto para exportar o código para o GitHub.

## 9. Exemplo de resposta da API (para validar o front)
{ "matchScore": 72, "keywordsFound": ["React", "TypeScript", "APIs REST"], "keywordsMissing": [{"keyword":"AWS","suggestion":"Se você já usou nuvem, mencione o termo AWS no currículo; senão, prepare uma resposta para a entrevista."}], "tips": ["Reescreva o resumo com os verbos da vaga", "Ordene as experiências da mais recente para a mais antiga"], "atsResumeText": "..." }

##  Como a análise funciona, da vaga colada até o currículo ajustado?

O usuário cola ou envia (PDF/DOCX/TXT) a descrição da vaga e o currículo;
A IA (via Edge Function ats-analyzer com Google Gemini) processa os dois textos e retorna: matchScore (0–100%), palavras-chave encontradas e faltantes (com sugestões de ação), dicas de melhoria por seção e o texto do currículo reescrito;
O app exibe o score com rótulo qualitativo (Alto/Médio/Baixo), separa as keywords em chips e mostra as dicas;
Ao clicar em "Gerar versão ATS-friendly", o currículo original é reformulado em formato monocoluna, sem tabelas, imagens ou colunas — compatível com ATS — e o usuário pode copiar o texto ou baixar DOCX.

## Ajustes pedidos depois da primeira geração, e por quê?

Regra de nunca inventar informações — a primeira versão precisou garantir que a IA jamais adicionasse experiência, formação, datas ou habilidades que não existem no currículo original; tudo que é sugestão deve vir marcado como "Sugestão:"/"Dica:". Motivo: veracidade — um currículo com dados falsos elimina o candidato e compromete a credibilidade;
Formato ATS-safe na versão ajustada — monocoluna, sem tabelas/imagens, usando a terminologia da vaga apenas para competências que já existem no currículo. Motivo: é o que garante que o arquivo seja lido corretamente pelo ATS;
Entrada por upload além da colagem (PDF/DOCX/TXT, drag & drop, limite ~2MB, com validação de texto extraível). Motivo: usabilidade, o usuário nem sempre tem o texto pronto para colar;
Design minimalista com cores suaves e neutras (off-white, verde-sálvia, tipografia limpa) e fluxo por etapas com estado de carregamento. Motivo: clareza visual e UX elegante, sem poluição;
Exportação dupla (copiar texto / baixar DOCX) e página única sem login. Motivo: uso direto e livre, sem barreiras de cadastro;
Tratamento de erro amigável da API ("Não consegui analisar agora. Tente novamente em instantes."). Motivo: robustez — nunca mostrar tela de erro ao usuário.

## Os testes registrados (vaga com CV pouco compatível e CV bastante compatível em .docx) serviram para validar que a análise funciona nos dois cenários: baixo e alto match. E funções de entrada, tanto por texto quanto por upload, assim como a saída por .docx, funcionam corretamente.

#### publicado em:
https://vaga-viva-bot.lovable.app
