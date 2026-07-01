# PRD — Bolão Copa 2026 (Fase de Grupos)

## 1. Visão Geral
Aplicativo web para um grupo fechado de amigos registrarem palpites dos jogos da fase de grupos da Copa do Mundo 2026, acompanharem o ranking em tempo real e conversarem em um chat único. Mobile-first, com painel administrativo completo.

**URL publicada:** https://bolaocopa2026fase1.lovable.app

---

## 2. Objetivos
- Centralizar palpites e resultados dos jogos da 1ª fase.
- Calcular pontuação de forma **não cumulativa** e transparente.
- Fornecer ranking com critérios de desempate claros.
- Permitir gestão total pelos administradores (jogos, regras, participantes).
- Promover engajamento via chat em tempo real.

---

## 3. Perfis de Usuário
| Perfil | Permissões |
|---|---|
| **Participante** | Cadastrar/alterar palpites até o prazo, ver ranking, consultar palpites de todos, usar o chat, editar o próprio perfil. |
| **Administrador** | Tudo do participante + cadastrar jogos, lançar/corrigir resultados, processar pontuação, editar regras, cadastrar/excluir participantes, gerar relatórios, compartilhar ranking, moderar chat. |

- Primeiro usuário cadastrado vira admin automaticamente.
- Limite de **5 administradores** (garantido por trigger no banco).
- Admins iniciais definidos: **Matheus Melo** e **Eldonor Mario F. Coelho**.

---

## 4. Autenticação
- Login por e-mail + senha, com opção **mostrar/ocultar senha**.
- Mensagens de erro traduzidas para **pt-BR**.
- Aviso sobre necessidade de validar o e-mail recebido.
- **Esqueci minha senha** com fluxo em `/reset-password`.
- Cadastro de participantes pelo admin: nome, e-mail e **senha numérica de 9 dígitos** (padrão telefone); conta criada já confirmada. Dados de confirmação exibidos ao admin criador.

---

## 5. Regras de Pontuação (não cumulativas)
As regras são **configuráveis** pelo admin em ordem de prioridade — a primeira que casar define a pontuação.

Padrão inicial:
1. **Placar Exato** — 5 pontos
2. **Vencedor ou Empate** — 3 pontos
3. **Placar Invertido** (não vale para empates) — 2 pontos
4. **Gols de 1 Time** — 1 ponto
5. Caso contrário — 0 pontos

Implementação espelhada em `src/lib/scoring.ts` (UI) e função SQL `public.calculate_points` (banco). Admins podem criar, editar, ativar/desativar, reordenar e excluir regras. Após mudanças, jogos finalizados podem ser **reprocessados**.

---

## 6. Palpites
- **Prazo único:** até **10/06/2026 às 19:00 (BRT)** — banner com contagem regressiva.
- Preenchimento **parcial** permitido; inclusão e alteração até o prazo.
- Após o prazo, palpites ficam somente leitura.
- Badges por palpite indicam a pontuação obtida.

---

## 7. Ranking (Home)
- Soma dos pontos de todos os participantes (com paginação para não perder registros).
- **Destaque visual para os 5 primeiros colocados.**
- Pódio para 1º, 2º e 3º.
- Textos "exatos"/"vencedores" **ocultos** na listagem.
- Botões: **Regulamento** e **Análise Detalhada** (modal com palpites e regras aplicadas por jogo).

### Critérios de desempate (nesta ordem)
1. Maior quantidade de palpites com **8 pontos**
2. Maior quantidade com **4 pontos**
3. Maior quantidade com **3 pontos**
4. Maior quantidade com **2 pontos**

---

## 8. Regulamento (acessível na Home)
### Disposições Preliminares
Integração dos participantes, entretenimento e premiação por precisão dos palpites.

### Premiação (5 primeiros colocados)
| Lugar | Prêmio |
|---|---|
| 1º | R$ 1.500,00 |
| 2º | R$ 600,00 |
| 3º | R$ 450,00 |
| 4º | R$ 300,00 |
| 5º | R$ 150,00 |

### Regra de Anulação
Partida(s) não realizada(s) ou com resultado não homologado são **desconsideradas** (pontuação zero para o jogo).

---

## 9. Chat
- Canal **único** para todos os participantes.
- Mensagens em tempo real (Supabase Realtime), autoscroll, estilo mensageiro.
- Usuário exclui as próprias mensagens; admin pode excluir qualquer uma.
- Admin pode limpar todo o histórico.

---

## 10. Painel Administrativo
Abas disponíveis:

1. **Jogos** — cadastro individual e em lote (Time A, Time B, Grupo, Data/Hora), lançamento e correção de resultado, **Processar / Reprocessar** pontuação.
2. **Regras** — CRUD das regras de pontuação com prioridade e ativação.
3. **Administradores** — listar (X/5), promover e remover admins.
4. **Cadastrar Participante** — criar conta com senha numérica de 9 dígitos.
5. **Consulta de Palpites** — busca por nome (≥ 2 letras), exibe palpites do participante e permite **excluir participante** (exceto admins).
6. **Relatório CSV** — exporta **todos** os palpites (paginação a 1.000 por vez para incluir os mais recentes): nome, jogo, palpite, resultado real, status, pontos.
7. **Compartilhar Ranking** — gera bloco de código monoespaçado (Posição, Nome, Pts, 8, 4, 3, 2) com botões **Copiar**, **Baixar .txt** e **WhatsApp** (Web Share API + fallback `wa.me`).

---

## 11. Arquitetura Técnica
- **Front-end:** TanStack Start (React 19) + Vite 7 + Tailwind v4 + shadcn/ui.
- **Back-end:** Lovable Cloud (Supabase) — Auth, Postgres, Realtime.
- **Server logic:** `createServerFn` (`@tanstack/react-start`) para operações privilegiadas (ex.: cadastro de participante, exclusão).
- **RLS** habilitar em todas as tabelas públicas com `GRANT` explícitos.
- Papéis em tabela separada `user_roles` + função `has_role` (SECURITY DEFINER) — sem risco de escalonamento.
- Deploy: Cloudflare Workers.

### Tabelas principais
`profiles`, `user_roles`, `matches`, `guesses`, `scoring_rules`, `chat_messages`.

### Funções SQL
`calculate_points`, `process_match`, `can_modify_guess`, `has_role`.

---

## 12. Design
- **Mobile-first**, esportivo e moderno.
- Paleta inspirada na Copa (verde, amarelo, azul) com gradientes suaves.
- Tokens semânticos em `src/styles.css`.
- Componentes: cards de jogo, badges de pontuação, tabelas limpas, banners de prazo.

---

## 13. Segurança
- Palpites visíveis apenas ao próprio usuário (exceto admin/consulta pública de ranking).
- Realtime protegido por RLS.
- Funções internas sem acesso anônimo.
- Segredos e chaves de serviço nunca expostos ao cliente.
