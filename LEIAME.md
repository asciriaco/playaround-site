# site/ — as páginas públicas que a Play Store exige

Três ficheiros estáticos. Sem código a correr, sem base de dados, sem nada para
manter em pé.

| | |
|---|---|
| `index.html` | entrada mínima, para a raiz não devolver 404 |
| `privacidade.html` | a Política de Privacidade — **gerada**, não editar aqui |
| `apagar-conta.html` | como pedir a eliminação da conta (pt + en) |
| `estilo.css` | a folha de estilo, partilhada pelas três |

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

## Onde publicar

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
