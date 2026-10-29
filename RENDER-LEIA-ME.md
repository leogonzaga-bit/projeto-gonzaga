# Implantação no Render

1. Suba todos estes arquivos na **raiz de um repositório GitHub privado**. Não suba `.env` ou credenciais Google.
2. Em https://dashboard.render.com/blueprint/new conecte o repositório e aplique o `render.yaml`.
3. Preencha MASTER_EMAIL, MASTER_PASSWORD e DRIVE_ROOT_FOLDER_ID na criação do Blueprint.
4. Nas variáveis de ambiente do serviço, configure a integração Google Drive via OAuth (GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET, GOOGLE_REFRESH_TOKEN, GOOGLE_REDIRECT_URI) ou conta de serviço com arquivo secreto montado e GOOGLE_SERVICE_ACCOUNT_JSON apontando para o caminho do arquivo.
5. Verifique o deploy e acesse a URL https://...onrender.com que o Render atribuir. Sem Google configurado, as operações do Drive não funcionarão.

**Custo**: o Blueprint usa serviço pago com disco persistente de 1 GB; confira o preço exibido no Render antes de aceitar. Não escolha uma instância gratuita para este SQLite sem adaptar a persistência.
**Segurança**: MVP não homologado para dados jurídicos reais, com sessões na memória e sem CSRF/MFA. Use apenas dados fictícios para testar. Não faça deploy público com credenciais de produção sem endurecimento.
