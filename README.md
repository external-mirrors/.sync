# .sync

## mesa

Mesa origin repo has a (some?) invalid object(s).

For example `feb356caff82e996ba0b898c02383fdfa3effc5f` .

- `git fsck` -> `cut -c 14-53`
- `git for-each-ref {hash}`

## omap5-sgx-ddk-um-linux (private)

The mirror is not publicly available because this repository is an unknown license.

## how to mirror large repos ex. linux

```sh
mkdir repo
cd depo
git init --bare
git remote add origin https://example.org/owner/repo.git
git remote set-url --push origin https://github.com/external-mirrors/repo.git
git fetch --tags
while (Enough times)
do
  git push origin [tag]
done
# GitHub Actions will do the rest
```
