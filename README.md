# Entrelinhas — biblioteca pessoal de textos

Projeto de biblioteca pessoal para anotações, livros, filmes/séries e projetos, com autenticação por e-mail, editor de texto rico, capas e revisão de texto por IA sob demanda. Inclui base para redirecionamento automático de administradores.

## Tecnologias

- HTML, CSS e JavaScript (ES modules)
- Supabase Auth e PostgreSQL com Row Level Security (RLS)
- Supabase Storage com bucket privado para capas
- Quill para edição de texto rico
- Supabase Edge Function + OpenAI API para revisão

## 1. Configurar o Supabase

1. Crie um projeto em https://supabase.com/.
2. Edite `js/config.js` e informe a URL do projeto e a chave pública `publishable` ou `anon` do Supabase.
3. **Nunca** coloque `service_role`, senha do banco ou chave privada da OpenAI no frontend.
4. No SQL Editor, execute primeiro `supabase/schema.sql` e depois `supabase/admin-setup.sql`.
5. Em Authentication → URL Configuration, configure Site URL e Redirect URLs para seu servidor local e domínio publicado.
6. Em Authentication → Providers → Email, habilite a confirmação de e-mail. O usuário precisará clicar no link recebido antes de entrar quando a confirmação estiver ativa.
7. Use um servidor local (VS Code + Live Server ou `npx serve .`). Não abra o HTML diretamente por `file://`.

## 2. Criar administradores

Todos usam a mesma tela de login/cadastro em `index.html`. O acesso administrativo é atribuído separadamente pelo banco.

1. Cadastre a conta pela tela normal e confirme o e-mail.
2. No SQL Editor, consulte a conta:

```sql
select id, email from auth.users where email = 'admin@exemplo.com';
```

3. Confira o endereço e copie o UUID retornado. Insira-o:

```sql
insert into public.admin_users (user_id)
values ('COLE-AQUI-O-UUID-REAL')
on conflict (user_id) do nothing;
```

4. Repita para cada administrador autorizado. Para remover o acesso, use `delete from public.admin_users where user_id = 'UUID-REAL';` no SQL Editor.

Após o login, o site consulta `public.is_admin()`: administradores vão para `admin.html`; contas comuns ficam na biblioteca. O painel verifica a sessão e o papel novamente. Se o SQL de administração não tiver sido executado, o redirecionamento administrativo não estará disponível.

**Limite atual:** `admin.html` é um painel inicial de acesso verificado, não um sistema completo de gestão de usuários/moderação. Cada ação administrativa que você adicionar deve validar permissões no servidor. Ser admin não dá acesso automático aos documentos privados dos usuários.

## 3. Revisão por IA

A chave da OpenAI deve ficar como segredo da Edge Function, nunca no navegador.

1. Instale o Supabase CLI e faça login.
2. Na raiz do projeto, vincule o projeto com `supabase link --project-ref SEU_PROJECT_REF`.
3. Cadastre o segredo: `supabase secrets set OPENAI_API_KEY=sua_chave`.
4. Publique a função: `supabase functions deploy revisar-texto`.

A API pode gerar custos. Antes de produção, configure limites de uso, monitoramento e CORS restrito ao domínio real.

## Arquivos importantes

- `index.html`: autenticação e biblioteca.
- `documento.html`: editor.
- `admin.html`: painel administrativo inicial.
- `js/app.js`: autenticação, verificação do papel e biblioteca.
- `js/admin.js`: valida sessão e papel administrativo.
- `js/config.js`: URL e chave pública do Supabase.
- `supabase/schema.sql`: tabelas, RLS e bucket privado.
- `supabase/admin-setup.sql`: papel de administrador e função de verificação.
- `supabase/functions/revisar-texto/index.ts`: revisão com OpenAI.

## Checklist de testes

- [ ] Preencher `js/config.js` com URL e chave pública corretas.
- [ ] Executar os dois SQLs na ordem indicada.
- [ ] Testar cadastro e confirmação de e-mail.
- [ ] Testar login, logout e recuperação de senha.
- [ ] Testar uma conta comum e uma conta promovida a administradora.
- [ ] Confirmar que usuários comuns não leem nem alteram documentos de outras contas.
- [ ] Manter as chaves secretas somente no servidor/Edge Functions.
- [ ] Configurar limites e segurança antes de publicar o site.
