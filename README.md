Local dev:

```
hugo server
```

Build:

```
rm -rf public/* && hugo
```

Push:

```
rsync -avzr --delete ./public/* <user>@<server>:<folder>
```
