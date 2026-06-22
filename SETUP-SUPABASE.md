# ☁️ Ligar a nuvem (Supabase) — sincronizar a dois em tempo real

Com isso, você e a Iandra editam a **mesma lista**, ao vivo, de qualquer aparelho — e os
dados ficam salvos na nuvem (com backup automático), sem depender do navegador.

> Enquanto não fizer isso, o app roda **100% local** normalmente. A nuvem é opcional.

Leva ~10 minutos. Você só vai **colar 2 chaves** e **rodar 1 script SQL**.

---

## 1) Criar o projeto
1. Acesse <https://supabase.com> → **Start your project** (login com GitHub/Google).
2. **New project**. Dê um nome (ex.: `pedro-enxoval`), defina uma senha de banco (guarde-a) e
   crie. Espere ~2 min ficar pronto.

## 2) Criar a tabela + segurança (rode o SQL)
No menu lateral do Supabase: **SQL Editor → New query**, cole o script abaixo
**trocando os dois e-mails** pelos de vocês, e clique **Run**.

> ⚠️ Copie só as linhas que começam com `--`, `create`, `alter` etc. **NÃO** copie as três
> crases (```` ``` ````) — elas são formatação deste arquivo, não fazem parte do SQL. Se colar
> as crases, dá `syntax error at or near "```"`.

Resultado esperado ao rodar: **"Success. No rows returned"**.

```sql
-- tabela única com o estado do app (1 documento compartilhado)
create table if not exists app_state (
  id text primary key,
  data jsonb not null default '{}',
  updated_at timestamptz default now()
);

-- liga a segurança por linha
alter table app_state enable row level security;

-- >>> TROQUE pelos e-mails de vocês dois <<<
create policy "casal le"     on app_state for select
  using ( (auth.jwt() ->> 'email') in ('voce@email.com','iandra@email.com') );
create policy "casal cria"   on app_state for insert
  with check ( (auth.jwt() ->> 'email') in ('voce@email.com','iandra@email.com') );
create policy "casal edita"  on app_state for update
  using ( (auth.jwt() ->> 'email') in ('voce@email.com','iandra@email.com') );

-- liga o tempo real nessa tabela
alter publication supabase_realtime add table app_state;
```

Só esses dois e-mails conseguem ler/editar — mais ninguém, mesmo com o link.

## 3) Configurar o login por e-mail
1. **Authentication → Providers → Email**: deixe **Email** ligado (já vem). Pode deixar
   "Confirm email" como está — o link mágico funciona.
2. **Authentication → URL Configuration**:
   - **Site URL**: a URL onde o app vai ficar hospedado (ex.: `https://pedro-enxoval.netlify.app`).
   - **Redirect URLs**: adicione a mesma URL.
   > Isso é o que faz o link do e-mail voltar pro app. (Para testar local, dá pra adicionar
   > também `http://localhost:8000`, mas o ideal é testar já hospedado.)

## 4) Colar as chaves no app
No Supabase: **Project Settings → API**. Copie:
- **Project URL**
- **anon public** (a chave longa)

Abra o arquivo **`config.js`** (na pasta `enxoval-bebe`) e cole:

```js
window.SUPA = {
  url: 'https://SEU-PROJETO.supabase.co',
  anonKey: 'eyJhbGciOi...sua-anon-key...'
};
```

Salve. Pronto — o app passa a pedir login por e-mail e a sincronizar.

## 5) Hospedar
- Vá em <https://app.netlify.com/drop> e **arraste a pasta `enxoval-bebe`** inteira
  (com `index.html`, `config.js`, `vendor/`).
- Pegue a URL que o Netlify gerar e confirme que ela está em **Site URL / Redirect URLs**
  no Supabase (passo 3). Se mudar a URL, atualize lá.

---

## Como usar no dia a dia
1. Cada um abre a URL, digita o **próprio e-mail** e clica em **Enviar link de acesso**.
2. Abre o e-mail **no mesmo aparelho** e clica no link → entra direto.
3. Mexam à vontade: o que um marca aparece no outro com **"Sincronizado 🔄"**.
4. Sem internet? Continua funcionando (cache local) e sincroniza quando a conexão voltar.
5. Botão **Sair** fica na barra lateral.

## Migrar os dados que já existem
Se você já tem itens cadastrados no modo local, ao entrar pela 1ª vez na nuvem o app
**sobe automaticamente** o que estava salvo. (Garantia extra: antes de ligar a nuvem, use
**Backup → Exportar JSON**; se precisar, é só **Importar** depois.)

## Dúvidas comuns
- **"Não entra / link não volta"** → confira **Site URL / Redirect URLs** (passo 3) batendo
  com a URL hospedada.
- **"Outra pessoa tentou e não viu nada"** → correto: o RLS só libera os e-mails do passo 2.
- **Custo** → plano free do Supabase + Netlify = R$ 0. O free do Supabase pausa o projeto se
  ficar ~1 semana sem uso; basta reativar no painel (1 clique).

## Segurança (transparência)
A `anonKey` fica visível no `config.js` (é assim em todo site estático) — **isso é esperado e
seguro**, porque quem protege os dados é o **RLS por e-mail** (passo 2): sem estar logado com um
dos e-mails autorizados, a chave não lê nem grava nada.
