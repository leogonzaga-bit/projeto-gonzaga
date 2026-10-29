# Portal de Arquivos e Tarefas — protótipo funcional (v0.1)

Aplicação web em Node.js/Express. **Os documentos permanecem no Google Drive.** O banco SQLite guarda usuários, ACL de pastas, tarefas, auditoria e estado da lixeira, mas **não** guarda o conteúdo dos arquivos.

## O que já existe

- Login por e-mail/senha, sessão HTTP-only; senha armazenada com bcrypt.
- Três papéis: master (acesso integral e auditoria), admin (gerencia usuários e permissões), comum (apenas pastas autorizadas).
- Funções: advogada, advogada parceira, estagiário, financeiro, colaborador, outro.
- Cadastro com nome, telefone, e-mail, papel e função; desativação/restauração de contas.
- Permissões por ID de pasta do Google Drive, incluindo as subpastas; verificação **no backend**, não apenas na interface.
- Navegação, criação de pastas, upload até 25 MB e download de arquivos binários.
- Lixeira reversível do **portal**, impedindo exclusão permanente pela interface. O arquivo permanece no Google Drive original; a lixeira não corresponde à lixeira nativa do Google Drive.
- Calendário mensal, criação de tarefas e histórico de tarefas, marcação de concluídas, tarefas vencidas.
- E-mail para tarefas do próprio dia às **07:00, horário de Brasília**, desde que SMTP configurado e servidor ligado.
- Logs de login, tentativas malsucedidas, upload, download, lixeira, restauração, criação/alteração de usuário, permissões e tarefas; visualização exclusiva do master.

## Instruções rápidas

1. Instale Node.js **20 ou superior** no servidor; execute `npm install` nesta pasta.
2. Copie `.env.example` para `.env` e configure uma senha master forte, e-mail master, `SESSION_SECRET` de pelo menos 64 caracteres aleatórios e `DRIVE_ROOT_FOLDER_ID`.
3. Crie um projeto no [Google Cloud Console](https://console.cloud.google.com/), habilite a **Google Drive API** e configure **uma** das duas opções:
   - **Workspace e Drive compartilhado (preferível):** crie uma conta de serviço, baixe a credencial JSON para fora da pasta pública e configure `GOOGLE_SERVICE_ACCOUNT_JSON`. Conceda à conta de serviço acesso ao Drive compartilhado/pasta raiz, com direitos suficientes para criar arquivos. Conforme regras do Workspace, a administração pode exigir outra estratégia de autenticação.
   - **Google Drive pessoal / Meu Drive:** crie credenciais OAuth 2.0, obtenha com consentimento do responsável o `refresh_token` com escopo `https://www.googleapis.com/auth/drive`, e configure `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `GOOGLE_REFRESH_TOKEN`, `GOOGLE_REDIRECT_URI`. Este protótipo não implementa assistente OAuth de consentimento; o token deve ser obtido durante a configuração administrativa.
4. Configure SMTP de saída (`SMTP_HOST`, `SMTP_USER`, `SMTP_PASS`, `SMTP_FROM` etc.) para lembretes. Sem SMTP, tarefas funcionam, mas **nenhum e-mail é enviado**.
5. Execute `npm start`. Publique via proxy reverso com **HTTPS**. Configure `NODE_ENV=production` após instalar certificado TLS e proxy, para cookies seguros.
6. Acesse com `MASTER_EMAIL` / `MASTER_PASSWORD`. A primeira conta master só é criada quando o banco ainda está vazio.
7. Para autorizar uma pessoa a uma pasta, copie o ID dela na URL do Google Drive, vá a **Usuários → Pastas** e informe os IDs separados por vírgula. O usuário verá apenas as pastas autorizadas e descendentes; o administrador vê todas as pastas da raiz configurada.

## Regras e avisos importantes

- **Controle de acesso não altera as permissões de compartilhamento que já existem no Google Drive.** Se um usuário possuir acesso direto ao Drive ou links públicos, ele poderá contornar o portal. Antes da implantação, restrinja acesso direto, links públicos e permissões externas conforme política institucional. A conta de integração deve ter acesso apenas à árvore definida como raiz.
- O upload de arquivos é limitado a 25 MB por arquivo e passa pela memória do servidor. Para arquivos grandes, implementar upload em streaming/resumível.
- Documentos nativos do Google Docs, Sheets e Slides exigem implementação extra de visualização/edição ou exportação. A API de download deste protótipo foi projetada para documentos binários (PDF, DOCX etc.).
- A lixeira é controlada **somente pela aplicação**. Ações feitas diretamente no Drive podem excluir ou mover arquivos à revelia do sistema. Para retenção robusta, versões e preservação jurídica, considerar Google Workspace com políticas de retenção/Google Vault e backups independentes.
- SQLite local e sessões em memória são adequados a um protótipo de **uma instância**. Para produção empresarial: PostgreSQL, sessões persistentes, jobs idempotentes, filas, backups automáticos, controle de recuperação de senha, MFA, proteções CSRF, monitoramento e testes de integração/end-to-end.
- Auditoria não é imutável e tem limite de exibição de 500 entradas; logs também precisam ser enviados a armazenamento externo com política de retenção.
- O frontend é um **MVP inicial**, sem a qualidade final de um produto comercial. O editor de permissões utiliza IDs das pastas, a versão final deve ter um seletor visual com árvore hierárquica e busca.
- Para conformidade com LGPD e uso em escritório jurídico, planejar classificação de documentos, confidencialidade, perfis por caso/processo, regras de compartilhamento, prazo de retenção e resposta a incidentes.

## Estrutura

- `server.js` — API, autenticação, Drive, ACL, banco, cron e SMTP.
- `public/index.html` — interface web responsiva inicial.
- `.env.example` — variáveis de instalação (nenhum segredo real incluído).
- `portal.sqlite` — criado na primeira execução; deve ser mantido fora da web pública, com backup.

**Estado:** código disponibilizado para instalação e testes. Não foi conectado à conta Google do solicitante, não foi publicado, e não passou por homologação de segurança. Não usar com documentos sensíveis reais antes de revisão técnica.
