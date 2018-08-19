---
title: naming_rule
tags: tips
---

## Rule for creating new file 

E.g: 

``` bash
$ hexo new "mypost" --category others
```
So the url for that post will be like this:
> mysite_url/:lang/:category/:year/:month/:day/:title/

> E.g: http://localhost:4000/vi/others/2018/08/11/mypost

And the directory that contains the file will be: 
> \_post/vi/other/mypost.md

If you don't provide the parameter for lang and category, it will be automatically set to undefined, so url for above post and directory of source file will be:
> E.g: http://localhost:4000/undefined/undefined/2018/08/11/mypost

> \_post/undefined/undefined/mypost.md

### List categories:

1. japansese
2. it
3. digital-marketing
4. life
5. others

_So, don't forget to give parameter for language and category :D_

__More info__: https://hexo.io/docs/permalinks