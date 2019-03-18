---
title: Hexo tutorial
tag: [Hexo,Tutorial]
categories: 
- Web
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```
OR
``` bash
$ hexo s
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```
OR
``` bash
$ hexo g
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```
OR
``` bash
$ hexo d
```

### Generate Categories

``` bash
$ hexo new page categories
```
Then open `source/categories/index.md`, Add `type: "categories"`
```
---
title: categories
date: 2019-03-09 13:31:42
type: "categories"
comments: false
---
```

### Generate Tags

``` bash
$ hexo new page tags
```
Then open `source/tags/index.md`, Add `type: "tags"`
```
---
title: categories
date: 2019-03-09 13:31:42
type: "tags"
comments: false
---
```

### Using tags or categories

Adding categories or tags description on page title
```
---
title: Hexo tutorial
tag: [Hexo,Tutorial]
categories: 
- Web
---
```

### Clean caches and regenerated

```
$ hexo clean
$ hexo g
```

More info: [Deployment](https://hexo.io/docs/deployment.html)
