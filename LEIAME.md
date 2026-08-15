# site/ — as páginas públicas e a app no browser

| | |
|---|---|
| `index.html` | entrada mínima, para a raiz não devolver 404 |
| `privacidade.html` | a Política de Privacidade — **gerada**, não editar aqui |
| `apagar-conta.html` | como pedir a eliminação da conta (pt + en) |
| `estilo.css` | a folha de estilo, partilhada pelas três |
| `webapp/` | **a app a correr no browser** — ver o fundo desta página |

## Porque é que isto existe

A Google Play exige **dois URLs públicos** antes de aceitar uma submissão: o da
política de privacidade e o do pedido de eliminação de conta. Nenhum dos dois
tinha onde ser servido:

- ⚠ **O Supabase Storage não serve HTML.** Um bucket público devolve
  `text/plain` para ficheiros HTML, de propósito, para ninguém alojar páginas
  em conteúdo de utilizador. A política aparecia no browser como um muro de
  código. Não adianta reenviar com `Content-Type: text/html` — foi tentado por
  upsert e por apagar-e-criar.
- **O `playaround.pt` está registado mas não resolve** — tem registos de email
  e mais nada, e o site fica para mais tarde.

## Publicado

**No ar desde 11 de agosto de 2026**, em GitHub Pages a partir do repositório
público `asciriaco/playaround-site`:

| | |
|---|---|
| Política de privacidade | https://asciriaco.github.io/playaround-site/privacidade.html |
| Eliminação de conta | https://asciriaco.github.io/playaround-site/apagar-conta.html |
| **A app no browser** | https://asciriaco.github.io/playaround-site/webapp/ |

⚠ **O `site/` vive AQUI, neste repositório.** O `playaround-site` é só o
espelho publicado — não se edita lá, senão as duas cópias divergem. Depois de
mudar alguma coisa:

```powershell
git subtree split --prefix=site -b _site_publicacao
git push site _site_publicacao:main
git branch -D _site_publicacao
```

(o remoto `site` aponta para `https://github.com/asciriaco/playaround-site.git`)

## Como foi publicado

Qualquer alojamento estático serve. O caminho mais curto é o **GitHub Pages a
partir de um repositório público** — os ficheiros aqui não têm nada de privado,
e não é preciso domínio nenhum:

```powershell
gh repo create playaround-site --public --source=site --push
# depois: Settings → Pages → Deploy from a branch → main → / (root)
```

Fica em `https://<utilizador>.github.io/playaround-site/`, e é esse endereço
que se põe na Play Console.

⚠ **Quando o `playaround.pt` existir**, aponta-se um CNAME para o mesmo sítio e
os URLs mudam. **Actualizar os dois campos na Play Console nesse dia** — um URL
de política que deixa de responder é motivo de suspensão da app, não é um aviso.

## Ao mudar a política

O `privacidade.html` é gerado a partir do `POLITICA_PRIVACIDADE.md` pelo
`gerar_pagina_privacidade.py`. Quem muda a política muda o `.md` e volta a
gerar — escrever à mão no HTML divergiu da fonte à terceira revisão.

A cópia aqui tem uma diferença face à original: a folha de estilo saiu de
dentro do `<style>` para o `estilo.css`, para as três páginas a partilharem.

## `webapp/` — a app no browser

É **a mesma app**, o mesmo código de `playaround/app`, compilado para browser.
Existe porque metade dos primeiros testadores tem iPhone e a versão iOS ainda
não existe: no browser entra toda a gente, e no iPhone dá para «Adicionar ao
ecrã principal» e ficar com um ícone como se fosse instalada.

Para voltar a publicar depois de mexer na app:

```powershell
cd C:\phc_sql\playaround\app
& C:\flutter\bin\flutter.bat build web --release --base-href /playaround-site/webapp/
Remove-Item C:\phc_sql\playaround\site\webapp -Recurse -Force
Copy-Item C:\phc_sql\playaround\app\build\web C:\phc_sql\playaround\site\webapp -Recurse
# e depois o subtree split/push de cima
```

⚠ **O `--base-href` não é opcional.** Sem ele a app fica à procura dos ficheiros
na raiz do domínio (`/main.dart.js`) e o que aparece é um ecrã branco, sem erro
nenhum visível a não ser na consola.

⚠ **Só funciona porque as imagens passaram a ser desenhadas em `<img>`** — ver
`app/lib/custom_code/widgets/imagem_da_rede.dart`. Dez dos quinze sites das
fontes não enviam CORS, e o Flutter web, ao contrário do Android, obedece à
política de mesma origem: antes disto a app aparecia no browser com um buraco
cinzento onde devia estar cada fotografia.

**O que não funciona no browser**, e é bom saber antes de alguém perguntar:

- **Os links `playaround://evento/123`** — são de app instalada. No browser um
  evento partilha-se pelo endereço da própria página.
- **A localização é menos precisa.** O browser dá a posição por wi-fi/IP em vez
  do GPS, e pergunta sempre que se abre a página.
- **A primeira visita puxa ~10 MB** (o motor do Flutter). Depois fica em cache e
  abre num instante. Numa rede móvel fraca, a primeira vez custa.

**A pasta pesa 33 MB no repositório** (o `canvaskit` traz várias variantes e o
browser só descarrega uma). É muito para um repositório git, e cada publicação
acrescenta mais uma cópia dos ficheiros grandes. Se um dia incomodar, o sítio
para isto é um alojamento estático a sério, não o git.

## Porque é que a página de eliminação não tem um botão que apague

Uma caixa pública onde se escreve um endereço e se carrega em «apagar» deixaria
**qualquer pessoa apagar a conta de qualquer outra** — bastava saber-lhe o
email. Um formulário assim só é seguro com confirmação por email (um link com
um código que só chega a quem é dono da caixa), e isso exige um serviço de
envio a funcionar em produção, que ainda não está montado.

Enquanto não estiver, a página documenta o caminho honesto: o botão dentro da
app, que já apaga tudo na hora e sabe quem está a pedir porque a pessoa está
autenticada, e um pedido por email a partir do endereço registado para quem já
desinstalou. É isto que a Google pede — uma página que explique **como** se
pede, **o que** é apagado e **quanto tempo** demora.
