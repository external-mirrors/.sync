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

```sh
#!/bin/bash

TAGS_PER_PUSH=8

TAG_LIST=$(git tag -l 'v*')
readarray -t TAG_ARRAY <<< "$TAG_LIST"

TOTAL_TAGS=${#TAG_ARRAY[@]}

echo "--- Push Git tags in ${TAGS_PER_PUSH} increments (total: ${TOTAL_TAGS} tags) ---"

i=0

while [ $i -lt $TOTAL_TAGS ]; do
    START_INDEX=$i

    END_INDEX=$((i + TAGS_PER_PUSH < TOTAL_TAGS ? i + TAGS_PER_PUSH : TOTAL_TAGS))

    LENGTH=$((END_INDEX - START_INDEX))
    TAGS_TO_PUSH=${TAG_ARRAY[@]:START_INDEX:LENGTH}

    echo "--- [${START_INDEX} / ${TOTAL_TAGS}] pushing ${LENGTH} tags ---"
    echo "  tag: ${TAGS_TO_PUSH}"

    git push mirror ${TAGS_TO_PUSH} || { 
        echo "🚨 faled to push, continue..." 
    }

    i=$END_INDEX
    
    echo "--- push done ---"
    echo ""
done

echo "✅ all done"
```
