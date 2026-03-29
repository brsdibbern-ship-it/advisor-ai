# ADVISOR AI — Setup do Supabase

## 1. Criar projeto no Supabase
Acesse https://supabase.com → New Project → anote:
- Project URL: https://xxxx.supabase.co
- Anon key (public)
- Service Role key (secret — nunca expor no frontend)

---

## 2. Rodar o SQL abaixo no Supabase SQL Editor

```sql
-- Tabela de perfis (uma por usuário)
create table profiles (
  id uuid references auth.users on delete cascade primary key,
  name text not null,
  nivel text default 'Iniciante',
  xp integer default 0,
  xp_max integer default 2000,
  streak integer default 0,
  streak_last_date date,
  level_title text default 'Assessor Trainee',
  created_at timestamp with time zone default now()
);

-- Tabela de sessões de treino
create table sessions (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users on delete cascade not null,
  persona text,
  objetivo text,
  cenario text,
  dificuldade text,
  nivel_usuario text,
  mode text,
  score_geral integer,
  score_objetivo integer,
  score_mercado integer,
  score_escuta integer,
  score_tom integer,
  humor_final text,
  dica_principal text,
  pontos_fortes text[],
  pontos_melhorar text[],
  had_audio boolean default false,
  num_trocas integer default 0,
  created_at timestamp with time zone default now()
);

-- Habilitar Row Level Security
alter table profiles enable row level security;
alter table sessions enable row level security;

-- Políticas para profiles
create policy "usuarios veem proprio perfil"
  on profiles for select using (auth.uid() = id);
create policy "usuarios atualizam proprio perfil"
  on profiles for update using (auth.uid() = id);
create policy "usuarios criam proprio perfil"
  on profiles for insert with check (auth.uid() = id);

-- Políticas para sessions
create policy "usuarios veem proprias sessoes"
  on sessions for select using (auth.uid() = user_id);
create policy "usuarios inserem proprias sessoes"
  on sessions for insert with check (auth.uid() = user_id);

-- Criar perfil automaticamente ao cadastrar
create or replace function handle_new_user()
returns trigger as $$
begin
  insert into public.profiles (id, name, nivel)
  values (
    new.id,
    coalesce(new.raw_user_meta_data->>'name', 'Assessor'),
    coalesce(new.raw_user_meta_data->>'nivel', 'Iniciante')
  );
  return new;
end;
$$ language plpgsql security definer;

create trigger on_auth_user_created
  after insert on auth.users
  for each row execute procedure handle_new_user();
```

---

## 3. Variáveis de ambiente no Vercel

No dashboard do Vercel → Settings → Environment Variables, adicione:

| Nome | Valor |
|------|-------|
| ANTHROPIC_API_KEY | sk-ant-... |
| SUPABASE_URL | https://xxxx.supabase.co |
| SUPABASE_SERVICE_ROLE_KEY | eyJ... (service role, não anon) |

---

## 4. Preencher config no index.html e admin.html

Abra index.html e admin.html e preencha no topo:
```js
const SUPABASE_URL = 'https://xxxx.supabase.co';
const SUPABASE_ANON_KEY = 'eyJ...'; // chave pública — pode expor
const ADMIN_EMAIL = 'seu@email.com'; // seu e-mail de admin
```

---

## 5. Deploy
```bash
vercel --prod
```
