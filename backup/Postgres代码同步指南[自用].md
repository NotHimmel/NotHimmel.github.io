以IvorySQL IVORYSQL_REL_1_STABLE分支同步pg 14.18代码为例：
1. 基于IVORYSQL_REL_1_STABLE新建sync_pg_14.18分支
```
git remote add pg https://git.postgresql.org/git/postgresql.git
git fetch pg
git checkout -b pg_14_branch REL_14_18/git switch -c pg_14_branch REL_14_18
git checkout sync_pg_14.18

./configure --prefix=/home/himmel/Dev/Ivory1 --enable-debug --enable-depend --enable-cassert CFLAGS=-O0
```

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
```
#!/bin/bash
# ==============================================================================
# Script to sequentially cherry-pick commits from a file.
#
# USAGE:
# 1. Copy your commit list to `commits_to_pick.txt`.
# 2. Run this script: ./run_cherry_pick.sh
#
# BEHAVIOR:
# - It's RESUMABLE. It modifies the input file.
# - On SUCCESS, it removes the picked commit from the top of the file and
#   continues to the next one.
# - On FAILURE (e.g., a merge conflict), it stops and gives you instructions.
#   After you resolve the conflict and run `git cherry-pick --continue`,
#   you can simply re-run this script to proceed with the remaining commits.
# - It will ALWAYS report the number of remaining commits after each attempt.
# ==============================================================================

# --- Configuration ---
# The file containing the list of commits to pick.
# IMPORTANT: This file will be MODIFIED. Use a copy of your original list.
COMMIT_LIST_FILE="commits_to_pick.txt"

# --- Colors for better output ---
COLOR_GREEN='\033[0;32m'
COLOR_RED='\033[0;31m'
COLOR_YELLOW='\033[1;33m'
COLOR_NC='\033[0m' # No Color

# --- Pre-flight Checks ---

# 1. Check if the input file exists.
if [ ! -f "$COMMIT_LIST_FILE" ]; then
    echo -e "${COLOR_RED}❌ Error: Input file '$COMMIT_LIST_FILE' not found.${COLOR_NC}"
    echo "Please create this file or change the 'COMMIT_LIST_FILE' variable in the script."
    exit 1
fi

# 2. Check if the git working directory is clean.
if ! git diff-index --quiet HEAD --; then
    echo -e "${COLOR_RED}❌ Error: Your git working directory is not clean.${COLOR_NC}"
    echo "Please commit or stash your changes before running this script."
    exit 1
fi

echo -e "${COLOR_YELLOW}Starting cherry-pick process from file: $COMMIT_LIST_FILE${COLOR_NC}"
echo "------------------------------------------------------------"


# --- Main Logic ---

# Loop as long as the file is not empty
while [ -s "$COMMIT_LIST_FILE" ]; do
    # Get the first line of the file, which contains the next commit to pick
    LINE=$(head -n 1 "$COMMIT_LIST_FILE")
    
    # Extract the commit ID (the first word on the line)
    COMMIT_ID=$(echo "$LINE" | awk '{print $1}')
    
    echo -e "Attempting to cherry-pick: ${COLOR_YELLOW}$LINE${COLOR_NC}"

    # Attempt to cherry-pick the commit.
    # The `if` statement checks the exit code of the `git` command.
    if git cherry-pick $COMMIT_ID; then
        # --- SUCCESS ---
        echo -e "${COLOR_GREEN}✅ Successfully cherry-picked $COMMIT_ID${COLOR_NC}"
        
        # Remove the successfully picked line (the first line) from the file
        sed -i '1d' "$COMMIT_LIST_FILE"
        
        # **MODIFIED LINE** - Report remaining commits after successful removal
        echo "   Commits remaining: $(wc -l < "$COMMIT_LIST_FILE")"
        echo "------------------------------------------------------------"

    else
        # --- FAILURE ---
        echo -e "${COLOR_RED}❌ FAILED to cherry-pick $COMMIT_ID.${COLOR_NC}"
        
        # **NEW LINE** - Report remaining commits before exiting
        echo "   Commits remaining: $(wc -l < "$COMMIT_LIST_FILE")"

        echo -e "${COLOR_YELLOW}ACTION REQUIRED: A merge conflict has likely occurred.${COLOR_NC}"
        echo ""
        echo "Please follow these steps:"
        echo "  1. Open the conflicting files and resolve the issues manually."
        echo "  2. Stage the resolved files using: git add <file1> <file2> ..."
        echo "  3. Finalize the cherry-pick with: git cherry-pick --continue"
        echo ""
        echo -e "After the cherry-pick is successfully continued, ${COLOR_GREEN}re-run this script${COLOR_NC} to proceed with the remaining commits."
        exit 1
    fi
done

echo -e "${COLOR_GREEN}🎉 All commits from '$COMMIT_LIST_FILE' have been successfully cherry-picked!${COLOR_NC}"
```