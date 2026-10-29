# Regras do projeto para o ui-designer

## O que ler na especificação
Seções 2 (dados e imagens), 10 (site público, principalmente 10.3 identidade visual), 11 (responsividade) e 12 (backoffice).

## Marca
O logo da Maestria é **monocromático (preto e branco)**, com um "M" geométrico de traço grosso e um **corte diagonal**. A identidade parte dele: "barbearia futurista", sóbria e premium.

### Arquivos (`assets/brand/` → copiar para `web/public/brand/`)
| Arquivo | Uso |
|---|---|
| `logo-branco.png` | Logo completo, branco com fundo transparente, para o **tema escuro** |
| `logo-preto.png` | Logo completo, preto com fundo transparente, para o **tema claro** |
| `icone-original.png` | Ícone "M" no círculo: avatar padrão e e-mails |
| `favicon-32.png`, `apple-icon.png` | Ícones do site |

### Imagens provisórias (`assets/placeholders/` → `web/public/placeholders/`)
`hero.svg`, `galeria-1..6.svg` e `profissional.svg`. Devem ficar fáceis de trocar pelas fotos reais.

## Direção visual
- **Tema escuro é o padrão** (`defaultTheme="dark"`), com o claro disponível em todas as telas.
- **Paleta monocromática com toque metálico:**
  - Escuro: fundo `#0A0A0A`, superfície `#141414`, borda `#262626`, texto `#FAFAFA`, texto secundário `#A3A3A3`, **primário branco** (botão branco com texto preto).
  - Claro: fundo `#FAFAFA`, superfície `#FFFFFF`, borda `#E5E5E5`, texto `#0A0A0A`, **primário preto**.
  - Prata/cromo **só em detalhes**: gradiente `#E5E5E5 → #737373` em linhas, bordas de destaque e brilho no hover.
  - Estados: sucesso `#22C55E`, erro `#EF4444`, aviso `#F59E0B`.
- **Tipografia:** Montserrat (700–800, levemente espaçada) nos títulos, que conversa com o "MAESTRIA" do logo; Inter no texto; Michroma opcional em rótulos pequenos "tech".
- **Motivo gráfico: a diagonal do "M"** em divisores de seção, cantos cortados de cards e destaques do hover. Também grid de linhas finas ao fundo, cards com borda de 1px, header com vidro fosco e ícones de traço fino.

## Telas que são responsabilidade sua
- Home na ordem da seção 10.2: header (com menu hambúrguer no celular), hero, sobre, serviços, equipe, galeria com lightbox, avaliações, localização e horários (com selo "Aberto agora"/"Fechado agora"), CTA final, footer e **botão flutuante de WhatsApp**.
- **Stepper de agendamento:** no celular, botão fixo no rodapé (acima da área segura e sem conflitar com o botão de WhatsApp) e horários em grade de 3 colunas; no desktop, resumo lateral. O botão "Entrar com Google" segue as diretrizes visuais do Google.
- **Login do backoffice:** logo centralizado e campos de código grandes.
- **Layout do backoffice:** sidebar a partir de `lg`; abaixo disso, navegação inferior com Painel, Agenda, Avaliações e Mais, mais um drawer. Tabelas viram cards abaixo de `md`. A agenda mostra 1 profissional por vez no celular.
- E-mails: um template simples com o logo e a paleta (o backend usa).
