# Regras do projeto para o qa-tester

## O que ler na especificação
Seções 5 (autenticação), 7 (agendamento e casos de borda), 9 (avaliações), 11 (responsividade) e 16 (critérios de aceite).

## Cenários obrigatórios no backend

### Disponibilidade (`availability.service.ts`), prioridade máxima
- Expediente com intervalo de almoço: o serviço não atravessa o almoço.
- Faixa do profissional recortada pelo horário de funcionamento.
- Dia fechado (`BusinessHour.isOpen = false`) e `ClosedDay` → nenhum horário.
- Serviço terminando exatamente no fim do expediente → permitido.
- Conflito com agendamento `CONFIRMED` bloqueia; com `CANCELLED`, não bloqueia.
- Folga de dia inteiro e folga parcial.
- Antecedência mínima e limite máximo de dias.
- Dia sem `WorkingHour` → nenhum horário.
- "Sem preferência" juntando horários de vários profissionais, e escolha do profissional com menos agendamentos no dia.
- Datas tratadas em `America/Sao_Paulo`.

### Agendamento
- Horário inválido → 400; horário ocupado → 409.
- **Dois pedidos simultâneos para o mesmo horário → só um criado** (integração contra o banco de teste).
- Cliente sem telefone não agenda. Limite de 3 agendamentos futuros ativos.
- Cancelamento dentro e fora do `cancelLimitHours`; cliente não cancela agendamento de outro.
- Falha no envio de notificação não impede a criação.

### Login do backoffice por código
- Expira em 10 min; uso único; bloqueia após 5 tentativas.
- Um novo pedido invalida o anterior; limite de 1 envio a cada 60 s e 5 por hora.
- Resposta idêntica para contato existente e inexistente.
- Usuário inativo não recebe código. Só o hash fica no banco.

### Login Google
- Token inválido, de outra `audience` ou com e-mail não verificado → 401 (mock do `verifyIdToken`).
- Primeiro login cria o `Customer`; os seguintes atualizam pelo `googleId`.

### Avaliações
- Só com atendimento `COMPLETED` do próprio cliente; uma por atendimento.
- Nasce `PENDING`; `/public/reviews` não lista pendentes e a média só considera aprovadas.
- O admin aprova, apaga e remove as de exemplo; o profissional não modera.

### Permissões
- O profissional não lê nem altera dados de outro profissional.
- `/admin/*` recusa PROFESSIONAL; `/customer/*` recusa `staff_token` e vice-versa; sem token → 401.

## Checklists manuais do frontend
- `/agendar` de ponta a ponta no celular (375px) e no desktop, nos dois temas, **incluindo o login Google no meio do fluxo sem perder a seleção**.
- Tratamento do 409 (abrir duas abas e confirmar o mesmo horário nas duas).
- Login do backoffice com o código recebido no Mailpit (http://localhost:8025).
- Todas as rotas do site e do backoffice em 375px, 768px e 1280px, sem rolagem horizontal e com o logo correto em cada tema.
- Alterar o horário de funcionamento ou a duração de um serviço no backoffice e conferir que os horários do site mudam na hora.
