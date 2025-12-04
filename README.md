# Painel de Senhas — Serviço de Radiologia HCPA

Versões dos arquivos:
- Painel (tela pública): V.3 (arquivo HTML/CSS/JS)
- Guichê (interface do operador): V7.7 (arquivo HTML/CSS/JS)

Resumo do projeto
- Sistema simples de chamada de senhas para o Serviço de Radiologia do HCPA com duas interfaces:
  - Painel público: exibe a senha atual, o tipo de atendimento (Agendar / Agendados), o guichê e um histórico das últimas chamadas. Possui animações e anúncio de voz (Web Speech API).
  - Guichê (operador): autenticação simples, geração/enchamada de senhas, rechamada, visualização do histórico, área administrativa e geração de relatórios (export para Excel).

Tecnologias utilizadas
- Frontend: HTML, CSS e JavaScript (puro).
- Realtime Database: Firebase Realtime Database (libraries compat usadas).
- Biblioteca de exportação: SheetJS (XLSX) para exportar relatórios.
- Hospedagem sugerida: Vercel (ou qualquer hosting de arquivos estáticos).
- Repositório: GitHub.

Arquivos principais e propósito
- painel.html (V.3)
  - Interface pública; escuta o nó `chamadaSenha` e `historicoChamadas` no Firebase.
  - Quando há nova chamada: atualiza tela, faz animação de destaque e usa speechSynthesis para anunciar.
  - Exibe últimas 5 chamadas no histórico do lado direito.
- guiche.html (V7.7)
  - Interface para operadores com tela de login simples.
  - Gera senhas usando transações em `contadoresSenha/<tipo>` para evitar duplicidade.
  - Atualiza `chamadaSenha`, `historicoChamadas`, `ultimoPorGuiche` e registra atendimentos em `atendimentos/<usuario>`.
  - Admin: ajustar contadores, gerenciar usuários autorizados, limpar atendimentos, visualizar / exportar relatórios.

Estrutura de dados usada no Firebase (resumo)
- chamadaSenha: objeto atual exibido no painel
  - { tipo, senha, guiche, repetir, ts }
- historicoChamadas: array/lista com registros
  - [{ tipo, senha, guiche, timestamp }, ...]
- contadoresSenha:
  - { Agendar: número, Agendados: número }
- usuariosAutorizados: array de strings (usuários)
- ultimoPorGuiche:
  - { "<guiche>": { "Agendar": { senha, timestamp }, "Agendados": { senha, timestamp } } }
- atendimentos:
  - { "<usuario>": { pushId: { timestamp, tipo, senha, guiche, data }, ... }, ... }

Como executar localmente (passo a passo)
1. Clone ou baixe o repositório com os arquivos (painel.html e guiche.html).
2. Atualize (se necessário) a configuração `firebaseConfig` nos arquivos com as credenciais do seu projeto Firebase.
   - Observação: o Firebase config (apiKey, authDomain, databaseURL, ...) normalmente fica no frontend em apps web; isto é esperado, mas as regras do Realtime Database devem proteger os dados.
3. Sirva os arquivos em um servidor local (algumas funcionalidades como speech dependem do navegador, mas não exigem HTTPS para uso básico):
   - Usando Python (na pasta com os arquivos):
     - Python 3: `python3 -m http.server 8000`
     - Acesse: http://localhost:8000/painel.html e http://localhost:8000/guiche.html
4. Abra duas janelas/navegadores (uma com painel e outra com guichê), faça login no guichê e teste chamadas.

Fluxo básico de uso
1. Operador faz login no guichê (usuário autorizado / admin).
2. Escolhe o número do guichê.
3. Clica em "Senha Agendar" ou "Senha Realizar" — o sistema incrementa o contador certo, grava dados no Firebase e atualiza o painel.
4. O painel exibe a senha, pisca e faz a leitura em voz alta; histórico é atualizado.
5. Rechamada: o botão "Rechamar" reenvia a última senha daquele guichê.

Checklist de testes para avaliação
- [ ] Abrir painel e guichê em navegadores diferentes.
- [ ] Logar no guichê com um usuário autorizado.
- [ ] Chamar senha Agendar → painel atualiza, pisca e fala.
- [ ] Chamar senha Realizar → painel atualiza corretamente.
- [ ] Rechamar → painel fala novamente e pisca.
- [ ] Conferir histórico no painel e no guichê.
- [ ] Fazer export de relatório (admin) e abrir o arquivo XLSX gerado.
- [ ] Validar que os contadores em `contadoresSenha` incrementaram corretamente.
