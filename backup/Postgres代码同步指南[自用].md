以IvorySQL IVORYSQL_REL_1_STABLE分支同步pg 14.18代码为例：
1. 基于IVORYSQL_REL_1_STABLE新建sync_pg_14.18分支
2. 
git remote add pg https://git.postgresql.org/git/postgresql.git
git fetch pg
git checkout -b pg_14_branch REL_14_18/git switch -c pg_14_branch REL_14_18
git checkout sync_pg_14.18

./configure --prefix=/home/himmel/Dev/Ivory1 --enable-debug --enable-depend --enable-cassert CFLAGS=-O0


3. get commits id list
```shell
#!/bin/bash

# ==============================================================================
# Script to analyze commits between two PostgreSQL release tags.
#
# It calculates the total number of commits and exports the commit IDs 
# in chronological order (oldest first) to a file.
# ==============================================================================

# --- Configuration ---
# The old tag (the starting point, exclusive)
OLD_TAG="REL_14_17"

# The new tag (the ending point, inclusive)
NEW_TAG="REL_14_18"

# The output file where commit IDs will be saved
OUTPUT_FILE="commits_${OLD_TAG}_to_${NEW_TAG}_chronological.txt"

# --- Main Logic ---

echo "Analyzing commits for PostgreSQL upgrade from $OLD_TAG to $NEW_TAG..."
echo "------------------------------------------------------------"

# Check if the tags exist locally.
# The '2>/dev/null' part suppresses error messages if the tag doesn't exist.
if ! git rev-parse --verify $OLD_TAG >/dev/null 2>&1; then
    echo "Error: Tag '$OLD_TAG' not found. Please run 'git fetch' first."
    exit 1
fi

if ! git rev-parse --verify $NEW_TAG >/dev/null 2>&1; then
    echo "Error: Tag '$NEW_TAG' not found. Please run 'git fetch' first."
    exit 1
fi

# 1. Calculate the total number of commits.
# The order doesn't matter for 'wc -l', but we add --reverse for consistency.
COMMIT_COUNT=$(git log --oneline --reverse $OLD_TAG..$NEW_TAG | wc -l)

echo "Total commits found: $COMMIT_COUNT"
echo ""

# 2. Export the commit IDs in chronological order (oldest first).
#    THIS IS THE KEY CHANGE: Added the --reverse flag.
echo "Exporting commit IDs to '$OUTPUT_FILE' in chronological order (oldest first)..."
git log --format="%H" --reverse $OLD_TAG..$NEW_TAG > $OUTPUT_FILE

echo "------------------------------------------------------------"
echo "✅ Done. The list of commit IDs has been saved to '$OUTPUT_FILE'."
``` 

4. Cherry pick