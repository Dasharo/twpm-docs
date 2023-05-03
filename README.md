# About

This repository contains source code for the Trustworthy Platform Module (TwPM)
documentation webpage

## Contribution

Please make sure to follow the steps below before publishing your changes as a
merge request.

### Local build

```shell
virtualenv -p $(which python3) venv
source venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

By default, it will host a local copy od documentation at:
`http://0.0.0.0:8000/`.

It is very important at this point to verify that the pages you have changed
render correctly as HTML in local preview.

### Make sure no TBD or TODO content is displayed

Find all occurrences:

```shell
grep -E "TBD|TODO" docs/**/*.md -r
```

Iterate over all occurrences and check if:

- file, where TBD or TODO occurs, is displayed (included in nav section of
  mkdocs.yml)
- TBD or TODO is visible on the website

There should be no TBD or TODO visible on the website.

### pre-commit hooks

- [Install pre-commit](https://pre-commit.com/index.html#install), if you
  followed [local build](#local-build) procedure `pre-commit` should be
  installed

- [Install go](https://go.dev/doc/install)

- Install hooks into repo:

```shell
pre-commit install --hook-type commit-msg
```

- Enjoy automatic checks on each `git commit` action!

- (Optional) Run hooks on all files (for example, when adding new hooks or
  configuring existing ones):

```shell
pre-commit run --all-files
```

#### To skip verification

In some cases, it may be needed to skip `pre-commit` tests. To do that, please
use:

```shell
git commit --no-verify
```

## Funding

This project was partially funded through the
[NGI Assure](https://nlnet.nl/assure) Fund, a fund established by
[NLnet](https://nlnet.nl/) with financial support from the European
Commission's [Next Generation Internet](https://ngi.eu/) programme, under the
aegis of DG Communications Networks, Content and Technology under grant
agreement No 957073.

<!-- markdownlint-disable-next-line MD033 -->
<p align="center">
<img src="https://nlnet.nl/logo/banner.svg" height="75">
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<img src="https://nlnet.nl/image/logos/NGIAssure_tag.svg" height="75">
</p>
<!-- markdownlint-enable-next-line MD033 -->
