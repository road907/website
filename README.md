# About Road907

Road907 is a community organization build by three friends in a bear
bar on road 907 who want to share the real interesting things about
the science, technology and philosophy.

We don't want to change this world, just make it fun.

## Init repo

`public` 和 `themes` 是两个submodule。 `public` 关联的是`git@github.com:road907/road907.github.io.git`， 它是网站build之后的输出。 `themes` 里放主题包，当前用的是`ghostwriter`。 在写文章之前，要先初始化两个submodule, 并让`public`这个目录处在可编辑状态。

1. `git clone git@github.com:road907/website.git`
2. `git submodule init`
3. `git submodule update`
4. `cd public && git checkout master  && git pull && cd ..` 把public切换到road907的主分支

## How to public to road907
- Edit post

- Run `hugo server` to preview the website after your post finished.

- Run `./deploy_github_pages.sh` to generate github pages website and push to `road907.github.io`.

- Visit `https://road907.github.io/` .
- commit && push this repo