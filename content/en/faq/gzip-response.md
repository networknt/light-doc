---
title: "Gzip Response"
date: 2017-11-06T16:10:17-05:00
description: ""
categories: []
keywords: []
slug: ""
aliases: []
toc: false
draft: false
reviewed: true
---

### How to compress the response with gzip

For some of the API endpoints, the response bodies are big and it might be necessary
to compress it before return it to the client. Undertow has a built-in EncodingHandler
that can be utilize for this.


Here is an example with customized path provider.

```
public` HttpHandler getHandler() {

    return new EncodingHandler(new ContentEncodingRepository()
            .addEncodingHandler("gzip",
                    new GzipEncodingProvider(), 50,
                    Predicates.parse("max-content-size[150]")))
            .setNext(
                    Handlers.routing()

                            .add(Methods.GET, "/v2/fronts/{front_id}", new FrontGetHandler(Config.getInstance().getMapper()))

                            .add(Methods.GET, "/v2/articles/{article_id}", new ArticleGetHandler(Config.getInstance().getMapper()))

                            .add(Methods.GET,"/v2/medias/{media_id}", new MediaGetHandler(Config.getInstance().getMapper()))

                            .add(Methods.GET,"/v2/menus/{site_id}", new MenuGetHandler(Config.getInstance().getMapper()))

                            .add(Methods.GET,"/v2/sections/{section_id}", new SectionGetHandler(Config.getInstance().getMapper()))

            );
}
```

For more information about the implementation, please refer to https://github.com/networknt/light-4j/issues/88


Thanks @samek to confirm it works and provide the example.
