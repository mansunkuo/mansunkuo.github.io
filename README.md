# blog
https://mansunkuo.github.io/blog/

## Dev
Start Hugo’s development server:
```bash
hugo server --disableFastRender --buildDrafts
```

New post:
```bash
hugo new content -k post -c content/tech $(date -I)-test.md
hugo new content -k post -c content/tech $(date -I)-test.zh-tw.md
```

## References
- [hugo-PaperMod](https://github.com/adityatelange/hugo-PaperMod)
- [Host on GitHub Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/)