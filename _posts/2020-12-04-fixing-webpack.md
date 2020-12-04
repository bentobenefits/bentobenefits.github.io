---
layout: post
title: "Fixing Webpack"
---

# Fixing Webpack

Author: ...Paul

I @#$%ing hate Javascript.

I used `brew` to install an unrelated package and it updated `node` to
15.3.0.  And that broke everything Javascript/webpack related in our Bento
application.  When the server would start up, the webpack process would fail:

```
ERROR in ./css/app.scss
Module build failed (from ./node_modules/mini-css-extract-plugin/dist/loader.js):
ModuleBuildError: Module build failed (from ./node_modules/sass-loader/dist/cjs.js):
Error: Node Sass does not yet support your current environment: OS X 64-bit with Unsupported runtime (88)
```

Now, at this point, MAYBE it would've worked if I'd remembered `npm update` and
not just trying to run `npm install` repeatedly.  But I didn't, so this is what
happened:

### Switch to nvm

A friend of mine said he preferred nvm for managing node in the first place,
so maybe that would help.  Time to `brew uninstall node` and then head over
to [https://nvm.sh](https://nvm.sh) to follow the easy directions there.
Restart the terminal to get all the shell goodies and...

### Now fix nvm

Terminal window starts up, and instead of a command prompt, it prompts me to
answer a question

```
zsh compinit: insecure directories, run compaudit for list.
Ignore insecure directories and continue [y] or abort compinit [n]?
```

Ugh.  Okay, so Googling for the answer to that question.  Get different
answers, likely because Apple keeps screwing around with how they do stuff,
but ultimately this seems to do the trick:

```sh
$ sudo chmod -R 755 /usr/local/share/zsh
$ sudo chown -R root:staff /usr/local/share/zsh
```

Now that runs clean.  Great.  `nvm install node` and that installs the exact
same version I just had.  Which generates the exact same error I had.

### Back to fixing Node

The error is in node-sass, and on the
[Elixir Slack](https://elixir-lang.slack.com/) I get a suggestion that
`node-sass` has been replaced by `dart-sass`.  But I find it hard to believe
that `node-sass` was completely deprecated.  Sure enough, though I see that
the version in my `package.json` file, 
[4.14.1 doesn't support node 15](https://github.com/sass/node-sass/releases/tag/v4.14.1) but [5.0.0](https://github.com/sass/node-sass/releases/tag/v5.0.0)
does.  So I try to bump the version in `package.json` to `^5.0.0`.  But that
doesn't work, becuase other stuff doesn't support that.

### Start over

I blow away the `node_modules` directory to try to start clean.  I go look up
webpack docs and it mentions installing `node-sass` or `dart-sass` first,
manually.  So I change the `package.json` file to use `dart-sass` instead of
`node-sass` BUT -- the former has completely reset the versioning from the
latter, so I have to figure out/reemmber that the version should be `^1.0.0`
because `dart-sass` hasn't gotten past v1, let alone v4.

Now, `npm install`.

### WTF?

Nothing changed.  In fact, I still see node-sass being used...

Oh riiight...  The `package-lock.json` file hasn't changed, and that's what the
install subcommand is looking at.  `npm update` and things start looking
better.  Seems like everything's fixed now.

### Conclusion

I @#$%ing hate Javascript.  Its ecosystem is horrific; people are creating a
ton of things that packages depend on, and then just abandon them in favor of
a different name, different numbering scheme, and effectively rewriting
everything.  Probably because the language is so godawful that it's not really
maintainable after a certain size.  It's easier to rebuild from scratch with
"lessons learned" than to try to rearchitect the existing library and maintain
compatibility.

And, always remember to try to `npm update` first.



