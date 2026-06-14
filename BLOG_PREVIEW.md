# BenchEvolver blog — editing & preview guide

This repo is a working copy of the [RDI website](https://github.com/rdi-berkeley/rdi-berkeley.github.io)
used to draft the **BenchEvolver** blog post before opening a PR upstream.

## Where the blog lives

```
blog/benchevolver/
├── index.md          ← the post (Markdown + a little inline HTML)
├── Example.png       ← the two worked examples figure
├── framework.png     ← method / cover figure
├── lcb_difficulty.png, scicode_difficulty.png, algo_shift.png
└── RL_curve1.png     ← training curves
```

The post is also registered in `_data/blogs.yml` (top entry) so it appears on the blog index.

**Editing rules**
- Keep every image/asset inside `blog/benchevolver/`.
- Reference images by filename only inside `index.md` (e.g. `src="framework.png"`), not by absolute path.
- The front matter `layout: blog` + `title:` must stay at the top.

## Preview locally

The site is built with Jekyll. Two ways to render it:

### Option A — Ruby + Bundler (standard)
```bash
bundle install
bundle exec jekyll serve
# open http://localhost:4000/blog/benchevolver/
```

### Option B — Docker (no Ruby install needed)
```bash
docker run --rm -it -v "$PWD":/srv/jekyll -w /srv/jekyll -p 4000:4000 \
  -u root -e HOME=/tmp ruby:3.1 \
  bash -c "gem install bundler -q && bundle install && bundle exec jekyll serve --host 0.0.0.0"
# open http://localhost:4000/blog/benchevolver/
```
> Note: on **rootless Docker**, avoid the `jekyll/jekyll` image — its entrypoint drops to a uid that
> can't write the bind-mounted repo. The plain `ruby:3.1` image above avoids that.

## Publishing to the real RDI site

When the post is ready, open a PR from this repo's `blog/benchevolver/` + `_data/blogs.yml`
against `rdi-berkeley/rdi-berkeley.github.io` (`main`), following the cybergym example structure.
