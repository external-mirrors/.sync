<img width="885" height="260" alt="image" src="https://github.com/user-attachments/assets/d7932059-ded6-41f9-9682-a83aaa1e35a6" /># .sync

## mesa

Mesa origin repo has a (some?) invalid object(s).

For example `feb356caff82e996ba0b898c02383fdfa3effc5f` .

- `git fsck` -> `cut -c 14-53`
- `git for-each-ref {hash}`

## omap5-sgx-ddk-um-linux (private)

The mirror is not publicly available because this repository is an unknown license.

## how to mirror large repos ex. linux

### simple, slow

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

### fast (by Gemini 2.5 Flash)

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

### crazy fast (by Gemini 2.5 Flash)

```sh
#!/bin/bash

TAGS_PER_PUSH=10
NUM_THREADS=5

TAG_LIST=$(git tag -l)
readarray -t TAG_ARRAY <<< "$TAG_LIST"

TOTAL_TAGS=${#TAG_ARRAY[@]}

echo "--- Push Git tags with ${NUM_THREADS} threads (total: ${TOTAL_TAGS} tags) ---"
echo "--- Tags per push: ${TAGS_PER_PUSH} ---"

GROUP_SIZE=$((TOTAL_TAGS / NUM_THREADS))
REMAINDER=$((TOTAL_TAGS % NUM_THREADS))

push_tags_thread() {
    THREAD_ID=$1
    START_INDEX=$2
    END_INDEX=$3
    THREAD_TAGS_COUNT=$((END_INDEX - START_INDEX))

    echo "--- 🧵 Thread ${THREAD_ID}: Tags ${START_INDEX} to ${END_INDEX} (total: ${THREAD_TAGS_COUNT}) ---"

    i=0
    while [ $i -lt $THREAD_TAGS_COUNT ]; do
        CURRENT_OFFSET=$i
        
        ACTUAL_START_INDEX=$((START_INDEX + CURRENT_OFFSET))
        LENGTH=$((TAGS_PER_PUSH < THREAD_TAGS_COUNT - CURRENT_OFFSET ? TAGS_PER_PUSH : THREAD_TAGS_COUNT - CURRENT_OFFSET))
        TAGS_TO_PUSH=${TAG_ARRAY[@]:ACTUAL_START_INDEX:LENGTH}

        echo "--- 🧵${THREAD_ID} [${ACTUAL_START_INDEX} / ${TOTAL_TAGS}] pushing ${LENGTH} tags ---"
        echo "  tag: ${TAGS_TO_PUSH}"

        git push mirror ${TAGS_TO_PUSH} || {
            echo "🚨 Thread ${THREAD_ID} failed to push, continue..."
        }
        
        echo "✅ 🧵${THREAD_ID} push done"

        i=$((i + LENGTH))
    done
    
    echo "--- 🧵 Thread ${THREAD_ID}: all its group tags pushed ---"
}

CURRENT_INDEX=0
for ((t=0; t<NUM_THREADS; t++)); do
    THREAD_GROUP_SIZE=$((GROUP_SIZE + (t < REMAINDER ? 1 : 0)))
    THREAD_END_INDEX=$((CURRENT_INDEX + THREAD_GROUP_SIZE))
    
    push_tags_thread $t $CURRENT_INDEX $THREAD_END_INDEX &

    CURRENT_INDEX=$THREAD_END_INDEX
done

wait

echo "✅ all done"
```
