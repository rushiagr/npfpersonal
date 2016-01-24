rushiagr.com
===

This is the actual code behind [rushiagr.com](http://www.rushiagr.com). It
uses [Hugo](http://hugo.spf13.com) to generate a static HTML site. Theme is
mostly copied over from [npf.io](http://npf.io), which was inspired by Hyde theme.

Create a new post
-----------------
1. Create new post with `hugo new your-new-post.md`
2. Move it inside 'blog' directory `mv your-new-post.md blog`
3. Add `type = "post"` in the metadata section of `your-new-post.md` file
4. Remove 'menu = "main"` in the metadata section of `your-new-post.md` file
