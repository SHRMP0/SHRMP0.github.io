---
title: "Adding Bluesky comments to Hugo PaperMod"
date: 2024-11-27
# weight: 1
slug: bluesky-comments-hugo-papermod
# aliases: ["/en/posts/bsky-comments-hugo"]
tags: ["bluesky", "hugo", "papermod", "tutorials"]
author: "SHRMP0" # ["Me", "You"] multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: true
bsky: "https://bsky.app/profile/shrmp0.com.br/post/3lbwuvfwyuk2w" # link to your bsky post
description: "A simple tutorial to learn how to turn skeets into your website or blog's comments section."
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
# images: ["/posts/comentarios-bluesky-hugo-papermod/images/bluesky-hugo.png"] # link or path of image for opengraph, twitter-cards
cover:
    image: "/posts/comentarios-bluesky-hugo-papermod/images/bluesky-hugo.png" # image path/url
    alt: "Hugo and Bluesky logos." # alt text
    caption: "Hugo and Bluesky logos." # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
editPost:
    URL: "https://github.com/SHRMP0/SHRMP0.github.io/tree/main/content"
    Text: "Suggest changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

A few days ago, one of Bluesky's developers wrote about the benefits of Bluesky being an open network [on her blog](https://emilyliu.me/blog/open-network). So far, so good; the dev is just marketing her own product, what's so big about that? Well, it turns out the comments section is a little different...

{{< bluesky link="https://bsky.app/profile/emilyliu.me/post/3lbq7mxsiek2v" >}}

After the initial :exploding_head:, people wanted to know how to use skeets[^1] as a website or blog's comments section. Wasting no time, she posted [her own guide](https://emilyliu.me/blog/comments) based on [this other guide](https://graysky.app/blog/2024-02-05-adding-blog-comments) ~~which has been around since February but had gone unnoticed by most folks~~. Since then, [more guides](https://www.coryzue.com/writing/bluesky-comments/) have emerged with different implementations, like [this version](https://gist.github.com/basyliq/ca50ae5442dce9e4f01b1821de7d973d) for sites made with Hugo.

## Demonstration

Want to see how this works? It's super easy, just go to the bottom of this page. Pretty crazy, right? And if you want to try it out, just reply to [this skeet](https://bsky.app/profile/shrmp0.com.br/post/3lbwuvfwyuk2w) on Bluesky, reload the page... and voilà!

At this point, you might be wondering if all this work is really worth it or if it's just a fluff, and the answer, in my opinion, is that it is. Besides simplifying things, since you're using Bluesky and [AT Protocol](https://atproto.com)'s own APIs, this means that:

1. Users you block on Bluesky won't appear in the comments section; and
2. Bluesky's moderation service provides additional protection against spam and abusive content.

## Tutorial

First of all, I'm **not a programmer** or anything like that, so I obviously wouldn't have the ability to write all of this from scratch. Therefore, 99% of the code used here came from [this tutorial](https://gist.github.com/basyliq/ca50ae5442dce9e4f01b1821de7d973d), also linked above. The changes I made are limited to adapting a few minor tweaks to better match the [PaperMod](https://github.com/adityatelange/hugo-PaperMod/) theme without having to edit it directly and/or according to my personal preferences.

To begin, create a file called `comments.html` inside the `/layouts/partials` folder:

```html
<div id="comments-section" data-bsky-uri="{{ .Params.bsky }}"></div>
{{ $comments := resources.Get "js/comments.js" }}
<script src="{{ $comments.RelPermalink }}"></script>
```

After that create a file called `comments.js` inside the `/assets/js` folder:

```javascript
document.addEventListener("DOMContentLoaded", () => {
  const commentsSection = document.getElementById("comments-section");
  const bskyWebUrl = commentsSection?.getAttribute("data-bsky-uri");

  if (!bskyWebUrl) return;

  (async () => {
    try {
      const atUri = await extractAtUri(bskyWebUrl);
      console.log("Extracted AT URI:", atUri);

      const thread = await getPostThread(atUri);

      if (thread && thread.$type === "app.bsky.feed.defs#threadViewPost") {
        renderComments(thread, commentsSection);
      } else {
        commentsSection.textContent = "Error fetching comments.";
      }
    } catch (error) {
      console.error("Error loading comments:", error);
      commentsSection.textContent = "Error loading comments.";
    }
  })();
});

async function extractAtUri(webUrl) {
  try {
    const url = new URL(webUrl);
    const pathSegments = url.pathname.split("/").filter(Boolean);

    if (
      pathSegments.length < 4 ||
      pathSegments[0] !== "profile" ||
      pathSegments[2] !== "post"
    ) {
      throw new Error("Invalid URL format");
    }

    const handleOrDid = pathSegments[1];
    const postID = pathSegments[3];
    let did = handleOrDid;

    if (!did.startsWith("did:")) {
      const resolveHandleURL = `https://bsky.social/xrpc/com.atproto.identity.resolveHandle?handle=${encodeURIComponent(
        handleOrDid,
      )}`;
      const res = await fetch(resolveHandleURL);
      if (!res.ok) {
        const errorText = await res.text();
        throw new Error(`Failed to resolve handle to DID: ${errorText}`);
      }
      const data = await res.json();
      if (!data.did) {
        throw new Error("DID not found in response");
      }
      did = data.did;
    }

    return `at://${did}/app.bsky.feed.post/${postID}`;
  } catch (error) {
    console.error("Error extracting AT URI:", error);
    throw error;
  }
}

async function getPostThread(atUri) {
  console.log("getPostThread called with atUri:", atUri);
  const params = new URLSearchParams({ uri: atUri });
  const apiUrl = `https://public.api.bsky.app/xrpc/app.bsky.feed.getPostThread?${params.toString()}`;

  console.log("API URL:", apiUrl);

  const res = await fetch(apiUrl, {
    method: "GET",
    headers: {
      Accept: "application/json",
    },
    cache: "no-store",
  });

  if (!res.ok) {
    const errorText = await res.text();
    console.error("API Error:", errorText);
    throw new Error(`Failed to fetch post thread: ${errorText}`);
  }

  const data = await res.json();

  if (
    !data.thread ||
    data.thread.$type !== "app.bsky.feed.defs#threadViewPost"
  ) {
    throw new Error("Could not find thread");
  }

  return data.thread;
}

function renderComments(thread, container) {
  container.innerHTML = "";

  const postUrl = `https://bsky.app/profile/${thread.post.author.did}/post/${thread.post.uri.split("/").pop()}`;

  const metaDiv = document.createElement("div");
  const link = document.createElement("a");
  link.href = postUrl;
  link.target = "_blank";
  link.textContent = `${thread.post.likeCount ?? 0} likes | ${thread.post.repostCount ?? 0} reposts | ${thread.post.replyCount ?? 0} replies`;
  metaDiv.appendChild(link);

  container.appendChild(metaDiv);

  const commentsHeader = document.createElement("h2");
  commentsHeader.textContent = "Comments";
  container.appendChild(commentsHeader);

  const replyText = document.createElement("p");
  replyText.textContent = "To comment, ";
  const replyLink = document.createElement("a");
  replyLink.href = postUrl;
  replyLink.target = "_blank";
  replyLink.style.textDecoration = "underline";
  replyLink.textContent = "reply to this post on Bluesky";
  replyText.appendChild(replyLink);
  container.appendChild(replyText);

  const divider = document.createElement("hr");
  container.appendChild(divider);

  if (thread.replies && thread.replies.length > 0) {
    const commentsContainer = document.createElement("div");
    commentsContainer.id = "comments-container";

    const sortedReplies = thread.replies.sort(sortByLikes);
    for (const reply of sortedReplies) {
      if (isThreadViewPost(reply)) {
        commentsContainer.appendChild(renderComment(reply));
      }
    }

    container.appendChild(commentsContainer);
  } else {
    const noComments = document.createElement("p");
    noComments.textContent = "No comments available.";
    container.appendChild(noComments);
  }
}

function renderComment(comment) {
  const { post } = comment;
  const { author } = post;

  const commentDiv = document.createElement("div");
  commentDiv.className = "comment";

  const authorDiv = document.createElement("div");
  authorDiv.className = "author";

  if (author.avatar) {
    const avatarImg = document.createElement("img");
    avatarImg.src = author.avatar;
    avatarImg.alt = "avatar";
    avatarImg.className = "avatar";
    authorDiv.appendChild(avatarImg);
  }

  const authorLink = document.createElement("a");
  authorLink.href = `https://bsky.app/profile/${author.did}`;
  authorLink.target = "_blank";
  authorLink.textContent = author.displayName ?? author.handle;
  authorDiv.appendChild(authorLink);

  const handleSpan = document.createElement("span");
  handleSpan.textContent = ` (@${author.handle})`;
  authorDiv.appendChild(handleSpan);

  commentDiv.appendChild(authorDiv);

  const contentP = document.createElement("p");
  contentP.textContent = post.record.text;
  commentDiv.appendChild(contentP);

  const actionsDiv = document.createElement("div");
  actionsDiv.className = "actions";
  actionsDiv.textContent = `${post.replyCount ?? 0} replies | ${post.repostCount ?? 0} reposts | ${post.likeCount ?? 0} likes`;
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

After that, add the following to the `custom.css` file (create it if it doesn't already exist), located in the `/assets/css/extended` folder:

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

Finally, just add this to the front matter of the posts you want to enable comments on:

```yaml
---
comments: true
bsky: "<bsky post url>" # link to your bsky post
---
```

**Done, now just test if everything is working and enjoy!**

## Limitations

Obviously, because it's something new and implemented so quickly by the community, adding comments via Bluesky is still a *rustic* process, so to speak. There are other minor things that I consider irrelevant and haven't mentioned, but the main limitations I've already noticed in the code I used are:

* New comments aren't loaded in real time;
* It's not possible to add or reply to comments directly from the page where they're displayed;
* GIFs, photos, videos, and link thumbnails aren't displayed correctly; and
* You need to manually link the skeet that will be used as the comments section on each page.

These things will most likely be improved in the future (and I'll try to update stuff here to accommodate them), but I still consider the current state of this integration to be good enough for my needs, at least.

*If you have any questions, leave a comment on the post and, hopefully, I or someone who understands the topic better will show up to help you. If you have any suggestions for improvements, feel free to send them too.*

[^1]: If you don't know why people call posts on Bluesky "skeets", [click here](https://knowyourmeme.com/memes/skeet-bluesky-slang).
