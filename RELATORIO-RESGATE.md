# Relatório de Resgate
- Discente: Lucio Aguiar de Souza
- Branch de trabalho: `resgate/discente-lucio`
- Data: 27/10/2026

> A atividade foi feita individualmente, então fiz sozinho os quatro papéis sugeridos
> (investigador Git, responsável técnico, recuperação e relator).

## Diagnóstico
Descreva os problemas encontrados e suas evidências.

Antes de mexer em qualquer coisa eu rodei o `git status` e o `git log --oneline --all --graph --decorate`
pra entender como o repositório estava. O que encontrei:

- **Branch ativa no começo:** `release-dev` (e não a `main`). Tinha também duas alterações
  locais não commitadas no `DESAFIO.md` e neste relatório, que eram só ajustes do enunciado
  (trocar "equipe" por "discente"), então mantive e commitei junto com o relatório.
- **Branches existentes:** 7 — `main`, `release-dev`, `backup-antigo`, `docs-readme`,
  `feature-cadastro`, `hotfix-login` e `teste-login-descartavel`. Com a minha
  `resgate/discente-lucio` ficaram 8.
- **Tags:** `v1.0.0-funcional` (a última versão que funcionava, mesma posição da `main`
  e da `backup-antigo`) e `commit-perigoso`.
- **Branches que guardavam coisa útil:** `docs-readme` (README completo),
  `hotfix-login` (uma nota dizendo como o login deveria ser corrigido) e a tag
  `v1.0.0-funcional` (código que compilava). A `feature-cadastro` tem uma validação
  de status da mercadoria, mas como não fazia parte do resgate eu não integrei.

Problemas encontrados, um por porta:

### Porta 1 – O projeto não compilava
O `mvn clean package` falhava com:

```
[ERROR] EntregaService.java:[6,28] package br.edu.entregas.util does not exist
[ERROR] EntregaService.java:[14,9] cannot find symbol  symbol: variable Validador
[ERROR] EntregaService.java:[19,33] constructor Mercadoria in class br.edu.entregas.model.Mercadoria cannot be applied to given types;
[ERROR] EntregaService.java:[20,19] cannot find symbol  (método gravar)
```

Na verdade eram dois problemas juntos. Comparando com `git diff v1.0.0-funcional` vi que:

1. O commit `a70ee84` (Henrique Nunes, "feat: acelerar cadastro de mercadorias") mudou
   `repository.salvar(...)` para `repository.gravar(...)`, só que esse método não existe no
   `MercadoriaRepository`. Ele também passou a criar a `Mercadoria` sem o endereço
   (6 parâmetros em vez de 7), ou seja, mesmo que compilasse, a mercadoria ficaria sem endereço de entrega.
2. O pacote `util` não existia mais porque a classe `Validador` tinha sido apagada (Porta 2).

### Porta 2 – Arquivo excluído
Com `git log --diff-filter=D --summary` achei a exclusão:

- Arquivo: `src/main/java/br/edu/entregas/util/Validador.java`
- Commit: `64f88f6f058bf6279244314b3b444d1c0fef0d23`
- Autor: Igor Reis — mensagem "refactor: remover classe aparentemente sem uso"

O nome do commit já diz muito: a classe foi apagada porque *parecia* sem uso, mas o
`EntregaService` usa ela pra validar nome, descrição, peso e valor.

### Porta 3 – Login com erro de lógica
O commit `9a6d3b0` (Felipe Rocha, "fix: liberar login para homologacao") trocou:

```java
// antes (v1.0.0-funcional)
return USUARIO.equals(usuario) && SENHA.equals(senha);
// depois
return USUARIO.equals(usuario) || senha == SENHA; // alteração urgente
```

Com o `||`, bastava acertar o usuário `admin` pra entrar com **qualquer senha**. E o `==`
compara referência de objeto em Java, não o conteúdo da String, então a senha digitada no
console nunca seria considerada igual. Esse commit está na branch `teste-login-descartavel`,
que pelo nome era pra ser descartada, mas acabou indo parar na linha da `release-dev`.
A branch `hotfix-login` (Eva Martins, `a6f356d`) só tem um `NOTA-HOTFIX.txt` explicando que o
certo é usar `equals` e exigir usuário **e** senha — a correção mesmo nunca foi feita.

### Porta 4 – README inútil
O README atual só dizia "Sistema interno. Pergunte ao desenvolvedor como executar."
O commit `0cd80f6` (Gustavo Melo, "docs: simplificar readme") apagou 18 linhas de
documentação. Com `git log --all -- README.md` encontrei a branch `docs-readme`
(commit `7fe8faa`, Carla Souza), que tem a versão completa e ainda acrescenta o fluxo de contribuição.

### Porta 5 – Informação sensível versionada
O commit `6572d8a` (Felipe Rocha, "chore: salvar credenciais para facilitar testes"),
marcado com a tag `commit-perigoso`, criou o `config/application.properties` com:

```
db.user=admin
db.password=SuperSenha123
api.token=TOKEN-NAO-DEVERIA-ESTAR-NO-GIT
```

Senha de banco e token de API não podem ficar no repositório. Removi o arquivo da versão atual
e coloquei no `.gitignore`. **Importante:** apagar o arquivo agora não apaga o segredo, porque o
Git guarda o conteúdo de todos os commits anteriores. Qualquer pessoa com uma cópia do repositório
consegue ver a senha com `git show commit-perigoso` ou `git show 6572d8a:config/application.properties`.
Como o Git é distribuído, cada clone tem o histórico inteiro. Por isso o certo seria:

1. Considerar a senha e o token como vazados e **trocar os dois** (isso é o mais importante);
2. Se fosse um repositório real, reescrever o histórico com `git filter-repo` ou BFG para tirar o
   arquivo de todos os commits e depois pedir pra todo mundo clonar de novo;
3. Daqui pra frente, usar variáveis de ambiente ou um arquivo de exemplo sem valores reais.

Não reescrevi o histórico aqui porque a atividade pede pra preservar as evidências.

## Comandos Git utilizados
Liste os comandos e explique a finalidade de cada um.

| Comando | Pra que usei |
|---|---|
| `git status` | Ver em qual branch eu estava e se tinha alteração pendente. |
| `git branch -a` / `git tag` | Listar as branches e as tags do repositório. |
| `git log --oneline --all --graph --decorate` | Ver o gráfico completo de commits, branches e tags. |
| `git log --stat` | Ver quais arquivos cada commit alterou. |
| `git log --all --format='%h %an %ad %s'` | Levantar autor e data de cada commit. |
| `git switch -c resgate/discente-lucio` | Criar a minha branch de resgate sem mexer na `main`. |
| `mvn clean package` | Compilar e ver o erro de compilação (e depois confirmar a correção). |
| `git log -- src/main/java/br/edu/entregas/service/EntregaService.java` | Ver o histórico do arquivo que não compilava. |
| `git diff v1.0.0-funcional -- src/main/java/br/edu/entregas/service/EntregaService.java` | Comparar o arquivo com a última versão que funcionava. |
| `git show a70ee84` | Ver exatamente o que o commit do Henrique alterou. |
| `git restore --source=v1.0.0-funcional -- .../EntregaService.java` | Restaurar o arquivo a partir da tag funcional. |
| `git log --diff-filter=D --summary` | Encontrar commits que apagaram arquivos. |
| `git log --all -- src/main/java` | Ver tudo que mexeu no código-fonte em todas as branches. |
| `git restore --source=64f88f6^ -- .../util/Validador.java` | Recuperar o `Validador.java` do commit anterior à exclusão. |
| `git log -- .../LoginService.java` e `git diff v1.0.0-funcional -- .../LoginService.java` | Achar e comparar a alteração do login. |
| `git show hotfix-login:NOTA-HOTFIX.txt` | Ler a nota do hotfix que confirmava a correção certa. |
| `git restore --source=v1.0.0-funcional -- .../LoginService.java` | Voltar o login para a versão correta. |
| `git log --all -- README.md` e `git show docs-readme:README.md` | Achar a versão boa do README em outra branch. |
| `git restore --source=docs-readme -- README.md` | Recuperar o README pelo histórico (sem reescrever). |
| `git log --all --oneline -- config/application.properties` | Achar o commit que versionou as credenciais. |
| `git show commit-perigoso` | Ver o conteúdo e o autor do commit com o segredo. |
| `git rm config/application.properties` | Remover o arquivo da versão atual. |
| `git add` / `git commit` | Registrar cada correção em um commit separado. |
| `git switch main` | Voltar para a `main` para integrar. |
| `git merge --no-ff resgate/discente-lucio -m "merge: integrar resgate do projeto"` | Integrar o resgate na `main` mantendo um commit de merge no histórico. |

## Commits relevantes
Informe os hashes investigados, revertidos ou recuperados.

**Investigados (onde os problemas entraram):**

| Hash | Autor | Mensagem | Problema |
|---|---|---|---|
| `356ec6f` (tag `v1.0.0-funcional`) | Bruno Costa | feat: implementar sistema de entregas funcional | Versão boa, usada como referência |
| `9a6d3b0` | Felipe Rocha | fix: liberar login para homologacao | Quebrou o login (`\|\|` e `==`) |
| `6572d8a` (tag `commit-perigoso`) | Felipe Rocha | chore: salvar credenciais para facilitar testes | Versionou senha e token |
| `0cd80f6` | Gustavo Melo | docs: simplificar readme | Apagou a documentação |
| `a70ee84` | Henrique Nunes | feat: acelerar cadastro de mercadorias | Quebrou a compilação e tirou o endereço |
| `64f88f6` | Igor Reis | refactor: remover classe aparentemente sem uso | Apagou o `Validador.java` |
| `7fe8faa` (branch `docs-readme`) | Carla Souza | docs: detalhar fluxo de contribuição | Fonte do README recuperado |
| `a6f356d` (branch `hotfix-login`) | Eva Martins | fix: documentar correção validada para autenticação | Só documentava a correção do login |

**Meus commits na branch `resgate/discente-lucio`:**

| Hash | Mensagem |
|---|---|
| `8a0c8d8` | fix(build): restaurar EntregaService da v1.0.0-funcional |
| `3beeae5` | fix: recuperar Validador.java removido em 64f88f6 |
| `306d08f` | fix(login): exigir usuário E senha corretos na autenticação |
| `d36aef9` | docs: recuperar README completo da branch docs-readme |
| `0b727e1` | security: remover application.properties com credenciais |
| `65a666f` | chore: ignorar arquivos de configuração do Eclipse |
| (este commit) | docs: preencher relatório de resgate |

Depois disso fiz o merge na `main` com `--no-ff`, então o histórico mostra o commit
`merge: integrar resgate do projeto` (dá pra ver com `git log --oneline --graph main`).

## Validação final
Registre como você confirmou compilação, login, cadastro, README e segurança.

Todos os testes foram feitos na branch de resgate e repetidos na `main` depois do merge.

| Teste | Como testei | Resultado |
|---|---|---|
| Compilação | `mvn clean package` | `BUILD SUCCESS` |
| Login correto | `admin` / `12345678` | `Acesso autorizado.` ✅ |
| Senha errada | `admin` / `errada` | `Acesso negado.` ✅ |
| Usuário errado | `joao` / `12345678` | `Acesso negado.` ✅ |
| Cadastro com endereço | Saída do sistema após o login | `#1 \| Notebook \| Notebook corporativo \| 2.1 kg \| R$ 4500,00 \| AGUARDANDO ENVIO \| Entrega: Av. Goiás, 1000 - Sala 8, Goiânia/GO - CEP: 74000-000` ✅ |
| README | Abri o `README.md` | Tem requisitos, como compilar, como executar e as credenciais de demonstração ✅ |
| Segurança | `ls config` e `git ls-files config` | O `application.properties` não existe mais na versão atual e está no `.gitignore` ✅ |

Pra rodar usei:

```bash
mvn clean package
java -cp target/classes br.edu.entregas.Main
```

## Pergunta de encerramento
**Como o uso adequado do controle de configuração reduz riscos técnicos, facilita auditorias e
permite recuperar um projeto mesmo depois de alterações incorretas?**

Nessa atividade ficou bem claro na prática o que a gente viu na aula 9. O sistema chegou quebrado
de quatro formas diferentes, mas como tudo estava versionado eu consegui descobrir **quem** alterou,
**quando** alterou e **o que** foi alterado em cada caso, sem precisar perguntar pra ninguém nem
reescrever nada do zero. A tag `v1.0.0-funcional` funcionou como uma linha de base: bastou comparar
com ela pra achar as regressões. O arquivo apagado e o README voltaram direto do histórico.

Isso reduz risco técnico porque nenhuma alteração é definitiva — sempre dá pra voltar a uma versão
estável. Facilita a auditoria porque o histórico é a própria evidência (hash, autor, data e diff), que
é justamente o que a auditoria de configuração precisa verificar. E as branches permitem testar coisas
separadas sem afetar a versão principal.

Por outro lado, a atividade também mostrou que a ferramenta sozinha não resolve: os problemas
vieram de falta de disciplina — commit com mensagem que não condiz com o que foi feito ("acelerar
cadastro" que quebra a compilação), branch "descartável" que acabou indo pra frente, classe apagada
sem verificar se era usada e senha commitada "pra facilitar". Se o time usasse revisão por Pull
Request e rodasse o build antes do merge (como o próprio README da `docs-readme` recomenda), quase
tudo isso teria sido barrado. E no caso da senha, o histórico que ajuda a recuperar o projeto é o
mesmo que mantém o segredo exposto, então segurança tem que ser pensada antes do commit, não depois.
