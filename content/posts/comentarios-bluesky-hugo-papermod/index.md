---
title: "Adicionando comentários via Bluesky ao Hugo PaperMod"
date: 2024-11-27
# weight: 1
# aliases: ["/posts/comentarios-bsky-hugo"]
tags: ["bluesky", "hugo", "papermod", "tutoriais"]
author: "SHRMP0" # ["Me", "You"] multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
bsky: "<bsky post url>" # link to your bsky post
description: "Um tutorial simples para aprender a transformar skeets na seção de comentários do seu site ou blog."
canonicalURL: "https://shrmp0.com.br/posts/comentarios-bluesky-hugo-papermod"
disableShare: true
disableHLJS: false # to disable highlightjs
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
images: ["images/bluesky-hugo.png"] # link or path of image for opengraph, twitter-cards
cover:
    image: "images/bluesky-hugo.png" # image path/url
    alt: "Logos do Hugo e Bluesky." # alt text
    caption: "Logos do Hugo e Bluesky." # display caption under cover
    relative: true # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/SHRMP0/SHRMP0.github.io/tree/main/content"
    Text: "Sugerir mudanças" # edit text
    appendFilePath: true # to append file path to Edit link
---

Alguns dias atrás, uma das desenvolvedoras do Bluesky escreveu sobre os benefícios do Bluesky ser uma rede aberta [em seu blog](https://emilyliu.me/blog/open-network). Até aí tudo normal, a dev tá vendendo o peixe dela, o que tem demais nisso? Bom, acontece que a seção de comentários é um pouco diferente...

{{< bluesky link="https://bsky.app/profile/emilyliu.me/post/3lbq7mxsiek2v" >}}

Após o :exploding_head: inicial, as pessoas queriam saber comofas pra utilizar skeets[^1] como seção de comentários em sites e blogs. Sem perder tempo, ela postou [um guia](https://emilyliu.me/blog/comments) próprio baseado [nesse outro guia](https://graysky.app/blog/2024-02-05-adding-blog-comments) ~~que existe desde Fevereiro mas passou batido pra muita gente~~. A partir daí surgiram [mais guias](https://www.coryzue.com/writing/bluesky-comments/) com implementações diferentes, a exemplo [dessa versão](https://gist.github.com/basyliq/ca50ae5442dce9e4f01b1821de7d973d) para sites feitos com o Hugo.

## Demonstração

Quer ver ao vivo como isso funciona? Fácil demais, vai lá no final dessa página. Que doideira da porra, né? E se quiser testar é só responder a [esse skeet aqui]() no Bluesky, recarregar a página... e voilà!

A essa altura você deve estar pensando se vale mesmo a pena esse trabalho todo ou se é apenas firula, e a resposta, na minha opinião, é que compensa sim. Além de simplificar as coisas, como você está utilizando as APIs do próprio Bluesky e [Protocolo AT](https://atproto.com/), isso significa que:

1. Os usuários que você bloquear no Bluesky não aparecerão na seção de comentários; e
2. O serviço de moderação do Bluesky fornece cobertura adicional contra spam e conteúdo abusivo.

## Tutorial

Primeiramente, eu **não sou um programador** nem nada assim, então é claro que não teria a capacidade de formular isso tudo do zero. Sendo assim, 99% do código utilizado aqui veio [desse tutorial](https://gist.github.com/basyliq/ca50ae5442dce9e4f01b1821de7d973d) também linkado acima. As alterações que eu fiz se resumem a traduzir parte do texto para português e adaptar uma besteira ou outra pra funcionar melhor com o tema [PaperMod](https://github.com/adityatelange/hugo-PaperMod/) sem precisar editá-lo diretamente e/ou conforme minhas preferências pessoais.

Para começar, crie um arquivo chamado `comments.html` dentro da pasta `/layouts/partials`:

```html
<div id="comments-section" data-bsky-uri="{{ .Params.bsky }}"></div>
{{ $comments := resources.Get "js/comments.js" }}
<script src="{{ $comments.RelPermalink }}"></script>
```

Depois disso crie um arquivo chamado `comments.js` dentro da pasta `/assets/js`:

```javascript
  const handleSpan = document.createElement("span");
  handleSpan.textContent = ` (@${author.handle})`;
  authorDiv.appendChild(handleSpan);

  commentDiv.appendChild(authorDiv);

  const contentP = document.createElement("p");
  contentP.textContent = post.record.text;
  commentDiv.appendChild(contentP);

  const actionsDiv = document.createElement("div");
  actionsDiv.className = "actions";
  actionsDiv.textContent = `${post.replyCount ?? 0} respostas | ${post.repostCount ?? 0} repostagens | ${post.likeCount ?? 0} curtidas`;
  commentDiv.appendChild(actionsDiv);

  if (comment.replies && comment.replies.length > 0) {
    const nestedRepliesDiv = document.createElement("div");
    nestedRepliesDiv.className = "nested-replies";

    const sortedReplies = comment.replies.sort(sortByLikes);
    for (const reply of sortedReplies) {
      if (isThreadViewPost(reply)) {
        nestedRepliesDiv.appendChild(renderComment(reply));
      }
    }

    commentDiv.appendChild(nestedRepliesDiv);
  }

  return commentDiv;
}

function sortByLikes(a, b) {
  if (!isThreadViewPost(a) || !isThreadViewPost(b)) {
    return 0;
  }
  return (b.post.likeCount ?? 0) - (a.post.likeCount ?? 0);
}

function isThreadViewPost(obj) {
  return obj && obj.$type === "app.bsky.feed.defs#threadViewPost";
}
```

Após isso, adicione o seguinte ao arquivo `custom.css` (crie-o se ainda não existir), localizado na pasta `/assets/css/extended`:

```css
.comment {
    padding: 10px 0;
}

.author {
    font-weight: bold;
}

.avatar {
    width: 24px;
    height: 24px;
    border-radius: 50%;
    vertical-align: middle;
    margin-right: 8px;
}

.nested-replies {
    margin-left: 20px;
    padding-left: 10px;
}

.actions {
    font-size: 12px;
    color: #666;
}

.nested-replies .comment {
    margin-bottom: 0;
}
```

Por fim, basta adicionar isso aqui ao cabeçalho dos posts em que deseja habilitar comentários:

```yaml
---
comments: true
bsky: "<bsky post url>" # link to your bsky post
---
```

**Pronto, agora é só testar se está tudo funcionando e ser feliz!**

## Limitações

Obviamente, por ser algo recente e implementado em tão pouco tempo pela comunidade, a adição de comentários via Bluesky ainda é um processo *artesanal*, por assim dizer. Tem outras coisinhas menores que eu acho irrelevante e não citei, mas as principais limitações que já percebi no código que utilizei são:

* Novos comentários não são carregados em tempo real;
* Não é possível adicionar ou responder a comentários diretamente da página em que eles são mostrados;
* GIFs, fotos, vídeos e miniaturas de links não são exibidos corretamente; e
* É preciso linkar o skeet que será usado como seção de comentários em cada página manualmente.

Muito provavelmente essas coisas serão melhoradas no futuro (e devo tentar atualizar o esquema aqui para contemplá-las), mas ainda assim já considero o estado atual dessa integração bom o suficiente para minhas necessidades pelo menos.

*Se ficou com alguma dúvida, deixa um comentário no post e, com fé, eu ou alguém que entenda melhor do assunto aparecerá pra te ajudar. Se tiver sugestões de como aprimorar algo, pode mandar também.*

[^1]: Caso não saiba o motivo de chamarem posts no Bluesky de skeets, [clique aqui](https://knowyourmeme.com/memes/skeet-bluesky-slang).