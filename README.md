# Dicionário Macuxi Multimídia

Aplicação web para consulta, documentação e difusão do vocabulário da língua **Macuxi** — língua indígena da família Carib, falada por cerca de 5.800 pessoas no Brasil (principalmente em Roraima) e também na República Cooperativa da Guiana.

O dicionário reúne palavras, traduções para o português e o inglês, classe gramatical, exemplos de uso e registros de pronúncia em áudio, com o objetivo de tornar o conhecimento linguístico acessível a membros da comunidade, professores e pesquisadores.

🔗 **Aplicação no ar:** [project-macuxi-dicionario.vercel.app](https://project-macuxi-dicionario.vercel.app)

---

## Sumário

- [Sobre o projeto](#sobre-o-projeto)
- [Equipe](#equipe)
- [Funcionalidades](#funcionalidades)
- [Tecnologias](#tecnologias)
- [Arquitetura](#arquitetura)
- [Banco de dados](#banco-de-dados)
- [Como rodar localmente](#como-rodar-localmente)
- [Deploy](#deploy)
- [Estrutura do código](#estrutura-do-código)
- [Área administrativa](#área-administrativa)
- [Roadmap](#roadmap)
- [Créditos e apoio](#créditos-e-apoio)

---

## Sobre o projeto

O **Dicionário Macuxi Multimídia** é desenvolvido no âmbito do **Laboratório de Estudo e Ensino de Línguas e Literaturas Indígenas (LEeLLI/PPGL)** da Universidade Federal de Roraima (UFRR), como parte do projeto **Inventário Nacional da Diversidade Linguística Indígena de Roraima**.

O projeto teve origem em um **curso de extensão** realizado de forma online em 2021, sob a condução do Prof. Vitor Francisco Juvêncio e coordenação da Prof.ª Ananda Machado. Em 2025, foi retomado e expandido como **iniciação científica**, contemplando o desenvolvimento desta aplicação web.

## Equipe

| Papel | Nome | Vínculo |
|---|---|---|
| Coordenação | Prof.ª Ananda Machado | Pesquisadora PQ CNPq |
| Docente (extensão 2021) | Prof. Vitor Francisco Juvêncio | — |
| Desenvolvimento / IC 2025 | Rafael da Silva | Bolsista CNPq |
| Revisão linguística / Mestrado | Rosilda da Silva e Silva | Bolsista Capes |

A validação do conteúdo linguístico (grafia, traduções, classes gramaticais) é feita por **corretores falantes da língua Macuxi**.

## Funcionalidades

**Área pública**

- Consulta ao acervo completo (~6.200 verbetes)
- Busca simultânea em Macuxi e Português
- Navegação pelo alfabeto Macuxi (A, E, F, H, I, Î, K, M, N, O, P, S, T, U, V, W, X, Y)
- Reprodução de áudio de pronúncia, quando disponível
- Exibição de classe gramatical, tradução em inglês e exemplos de uso
- Página de apresentação institucional integrada à home
- Tema claro/escuro

**Área administrativa** (restrita)

- Login com e-mail e senha
- Recuperação de senha por e-mail
- Cadastro, edição e exclusão de verbetes (CRUD)
- Navegação por letra inicial e busca textual
- Edição em duas colunas (lista + formulário) para revisão em série

## Tecnologias

| Camada | Tecnologia |
|---|---|
| Interface | React 18 (via CDN, com Babel no navegador) |
| Estilização | Tailwind CSS (via CDN) |
| Backend / Banco | Supabase (PostgreSQL) |
| Autenticação | Supabase Auth |
| Armazenamento de mídia | Supabase Storage |
| Hospedagem | Vercel |
| Versionamento | Git + GitHub |

> **Nota sobre a arquitetura atual:** o projeto é, nesta fase, um **único arquivo `index.html`** que carrega React, Babel e Supabase por CDN e transpila o JSX no próprio navegador. É uma abordagem simples e sem etapa de build, adequada ao estágio inicial do projeto. A migração para **Vite + React** (com build real e código modularizado) está planejada — veja o [Roadmap](#roadmap).

### Versões fixadas das dependências (CDN)

As versões abaixo estão **fixadas propositalmente**. Uma atualização automática do Supabase para um formato de módulo incompatível com o Babel no navegador chegou a derrubar a aplicação em produção; fixar as versões evita que isso se repita.

```
react@18.2.0
react-dom@18.2.0
@babel/standalone@7.23.5
@supabase/supabase-js@2.39.3  (build UMD)
```

## Arquitetura

```
┌──────────────────────────────┐
│         Navegador            │
│  index.html (React + Babel)  │
│  • busca e filtros no front  │
└──────────────┬───────────────┘
               │  HTTPS (SDK supabase-js)
               ▼
┌──────────────────────────────┐
│           Supabase           │
│  • PostgreSQL (verbetes)     │
│  • Auth (login admin)        │
│  • Storage (áudios/imagens)  │
│  • RLS (regras de acesso)    │
└──────────────────────────────┘
```

A busca e a filtragem acontecem **no navegador**: a aplicação carrega os verbetes em blocos de 1.000 registros (limite por requisição do Supabase) e filtra localmente. Para um acervo desse tamanho, essa abordagem é simples e eficiente.

## Banco de dados

### Tabela `tabela_verbetes`

Tabela principal, com aproximadamente **6.200 registros**.

| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | int | Identificador único (PK) |
| `macuxi` | text | Palavra em Macuxi (entrada do verbete) |
| `traducao_portugues` | text | Tradução em português |
| `ingles` | text | Tradução em inglês |
| `classe_gramatical` | text | Classe gramatical |
| `exemplo_macuxi` | text | Frase de exemplo em Macuxi |
| `exemplo_portugues` | text | Tradução da frase de exemplo |
| `audio_url` | text | URL do áudio de pronúncia (Storage) |
| `categoria_id` | int | Categoria temática (em implantação) |

### Tabela `admins`

Controla quem tem permissão de administrador.

| Coluna | Tipo | Descrição |
|---|---|---|
| `user_id` | uuid | Referência ao usuário em `auth.users` |
| `email` | text | E-mail do administrador |

### Segurança (RLS)

O **Row Level Security** está ativo. As políticas garantem que:

- **Leitura** dos verbetes é pública (qualquer visitante consulta o dicionário).
- **Escrita** (inserir, editar, excluir) é permitida apenas a usuários autenticados que constem na tabela `admins`.

A aplicação usa a **chave anônima** (`anon key`) do Supabase, que é segura para exposição no front-end justamente porque as permissões são controladas pela RLS no banco. A chave de serviço (`service_role`) **nunca** deve ser usada no navegador.

## Como rodar localmente

Por ser um único arquivo estático, não há etapa de instalação nem build.

### Opção 1 — Live Server (recomendado)

1. Abra a pasta do projeto no **VS Code**.
2. Instale a extensão **Live Server**.
3. Clique com o botão direito em `index.html` → **Open with Live Server**.
4. O navegador abre em `http://127.0.0.1:5500`.

### Opção 2 — Abrir o arquivo diretamente

Basta abrir o `index.html` no navegador. O Live Server é preferível porque recarrega automaticamente a cada alteração.

> **Requisito:** conexão com a internet, já que React, Tailwind, Babel e o SDK do Supabase são carregados por CDN.

### Configuração

As credenciais de conexão com o Supabase (URL do projeto e chave anônima) ficam no objeto `config`, dentro do próprio `index.html`:

```js
const config = {
  supabaseUrl: 'https://<seu-projeto>.supabase.co',
  supabaseAnonKey: '<sua-anon-key>',
  tableName: 'tabela_verbetes'
};
```

## Deploy

O deploy é feito pela **Vercel**, conectada ao repositório no GitHub. Cada `git push` para o branch principal dispara uma nova publicação automaticamente.

```bash
git add .
git commit -m "descrição da alteração"
git push origin main
```

Em cerca de 30 segundos, a Vercel publica a nova versão. Como o projeto é estático, não há configuração de build necessária.

## Estrutura do código

Todo o código vive em `index.html`, organizado em componentes React declarados no mesmo arquivo:

| Componente | Responsabilidade |
|---|---|
| `App` | Roteamento por estado (landing / dicionário / admin / reset de senha) e tema |
| `LandingPage` | Página inicial: boas-vindas, alfabeto e apresentação do projeto |
| `DictionaryPage` | Tela de consulta: busca, lista de palavras e detalhe |
| `WordList` / `WordDetail` | Lista de resultados e ficha da palavra selecionada |
| `AdminLogin` | Login e recuperação de senha |
| `AdminPanel` | Painel de gestão de verbetes (CRUD) |
| `ResetPasswordPage` | Redefinição de senha |
| `SobreProjeto`, `FichaProjeto`, `Objetivos`, `Equipe` | Seções da apresentação |
| `Header`, `Button`, `Icon`, `ThemeToggle`, `Avatar` | Componentes reutilizáveis |
| `useTheme` | Hook do tema claro/escuro |

## Área administrativa

O acesso fica no botão **Admin**, no cabeçalho da página inicial.

**Para conceder acesso a um novo administrador:**

1. A pessoa cria uma conta (ou é criada em **Supabase → Authentication → Users**).
2. Copie o `user_id` (UUID) desse usuário.
3. Insira uma linha na tabela `admins` com esse `user_id` e o e-mail correspondente.

Somente usuários presentes na tabela `admins` conseguem efetuar login na área administrativa; os demais são deslogados automaticamente.

> **Observação:** o plano gratuito do Supabase limita o envio de e-mails (por exemplo, de recuperação de senha) a uma pequena quantidade por hora. A mensagem "email rate limit exceeded" nesse contexto é esperada.

## Roadmap

Funcionalidades e melhorias planejadas:

- [ ] Filtro por área temática (Animais, Plantas, Seres Encantados) — requer a criação da tabela `categorias` e o preenchimento de `categoria_id`
- [ ] Filtro por classe gramatical
- [ ] Padronização das classes gramaticais no acervo
- [ ] Normalização do caractere `î` na busca
- [ ] Cadastro de novos administradores pelo próprio painel
- [ ] Upload de áudio e imagem diretamente pela área administrativa
- [ ] Persistência da preferência de tema (claro/escuro) entre visitas
- [ ] **Migração para Vite + React** (build real, código modularizado, sem transpilação no navegador)

## Créditos e apoio

Projeto desenvolvido no **LEeLLI/PPGL — Universidade Federal de Roraima (UFRR)**, no âmbito do Inventário Nacional da Diversidade Linguística Indígena de Roraima.

**Apoio institucional:** CNPq · Capes · LEeLLI · PPGL · Insikiran · UFRR · OPIRR

---

<p align="center">
  <strong>Morîîpe Aisakon</strong> — Sejam bem-vindos!
</p>