# Regras do projeto para o mentor

- Notas de aprendizado em `docs/aprendizado/fase-N.md`, uma por fase.
- Antes de escrever uma nota, leia as anteriores para **não repetir** conceitos já explicados. Faça referência a elas ("visto na fase 1").
- Conceitos que este projeto tem e que merecem explicação quando aparecerem:
  - dois guards de autenticação diferentes (cliente e equipe) e como o Nest decide qual usar;
  - login sem senha (código por e-mail): por que guardar só o hash e como funciona o rate limit;
  - validação do ID token do Google no backend;
  - `prisma.$transaction` com isolamento `Serializable` e por que ele evita agendamento duplo;
  - fuso horário: UTC no banco e `America/Sao_Paulo` na regra de negócio;
  - Server Components buscando dados da API versus Client Components no stepper;
  - `proxy.ts` versus a validação real na API;
  - next-themes e tokens de tema no Tailwind 4.
- Exercícios sugeridos devem usar o domínio da barbearia (ex.: "adicione o campo `instagram` no profissional, do schema até o card da equipe").
