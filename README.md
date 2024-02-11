Years of writing groff have made me tired.
This converts Linux manpages I have additions for to markdown.
New features will be documented here.

To convert an existing manpage it's literally just:

```
pandoc man2/$MANPAGE -w gfm -o $MANPAGE.md
```
