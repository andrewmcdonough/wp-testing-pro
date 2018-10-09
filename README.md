# Wordpress Sync Test

## Create a Blog post in Git

To add a new blog post to Wordpress from GitHub create a new document in the `_posts` folder with all the contents. 
You only need to add these lines in the top of your document:

```
---
post_title: <title of the blog post>
layout: post
published: true
---
```

If you want it to be a draft, just define `published: false`.
