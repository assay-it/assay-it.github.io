# assay.it

Web-Site https://assay.it

```bash
podman build -t assay-it .


podman run -it \
  -v $GOPATH/github.com/assay-it/assay-it.github.io:/usr/src/app \
  -p 4000:4000 \
  assay-it \
  bundle exec jekyll serve -H 0.0.0.0 -t
```
