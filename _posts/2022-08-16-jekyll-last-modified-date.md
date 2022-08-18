---
layout: post
title: "Jekyll : automatically add a lastModified attribute to your posts"
date:   2022-08-18 16:18:00 +0200
categories: github pages jekyll bash git hook commit
lastModifiedDate: 2022-08-18 17:43:20 +0200
---

**[Jekyll](https://jekyllrb.com/)** is an amazing tool to generate static websites & blogs from plain text files!

With very little configuration, you can run a website using the theme of your dreams, and host it from free using [github pages](https://pages.github.com/) or [gitlab pages](https://docs.gitlab.com/ee/user/project/pages/)!

Actually, this very blog has been generated and deployed with jekyll and github pages!

Though, this solution would not feature a "last modified date" attribute on blog posts, that updates automatically.<br>
I wanted it, so I added it using git hooks!

Here's how I did the trick!

## First considerations

The first things to think about is: 
* How can I modelize the timestamp of the last modification of a blog post ?
* How can I save it ?
* How can I trigger it ?

### What is a "last modified date"?

Well, the answer is quite obvious: The last modified date should represent the timestamp when I save the files I am modifying.

As committing to git is the most reliable way to save my work, I should use the timestamp corresponding to the moment I commit!

### How can I represent it?

The Jekyll syntax let the user add whatever metata they want. Basically, there are four basic properties initialised:

>* layout: The name of the layout to use for the post
>* title: The title of the post
>* date: The date the post is created
>* categories: A bunch of keywords

So, I am just going add one more there! The final output should look like this:
```diff
  ---
  layout: post
  title: "My amazing blog post"
  date: 2022-08-07 16:18:00 +0200
  categories: bleh blah blue
+ lastModifiedDate: 2022-08-16 12:15:00 +0200
  ---
```

### How can I trigger it?

Since the last modified date is a timestamp took from the commit, we can add a pre-commit hook to update the files.

>ℹ️ A git hook is a script which is executed on specific steps of the git flow. The pre-commit hook is triggered. [Read more here](https://delicious-insights.com/fr/articles/git-hooks/)

![Git flow & hooks *by delicious-insights*](https://delicious-insights.com/assets/images/articles/git-hooks.png)

## Write the hook

> ℹ️ You can find the [source code here 🤓](https://github.com/Phitrapy/phitrapy.github.io/blob/main/git_hooks/pre-commit)

When you are happy with the hook you have written, you have to copy it in the folder `.git/hooks`, with the name `pre-commit`.

Now, you can test it all with the command `git commit`, and you are all set!