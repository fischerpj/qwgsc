# quarto-giscus
Minimal site demonstrating giscus functionality

#!/usr/bin/env bash
set -euo pipefail

###############################################
# CONFIGURATION (EDIT THESE)
###############################################

# URL of the foreign/original repo you want to replicate
FOREIGN_REPO_URL="https://github.com/r-leyshon/quarto-giscus.git"
## git@github.com:r-leyshon/quarto-giscus.git

# Name of the new repo you want in your GitHub account
NEW_REPO_NAME="qwgsc"

# Your GitHub username
GITHUB_USER="fischerpj"

# Your GitHub personal access token (classic or fine-grained)
# You can also export it before running:
export GITHUB_TOKEN="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"
##GITHUB_TOKEN="${GITHUB_TOKEN:-}"

###############################################
# SAFETY CHECKS
###############################################

if [ -z "$GITHUB_TOKEN" ]; then
  echo "ERROR: GITHUB_TOKEN is not set."
  echo "Export it first:  export GITHUB_TOKEN=your_token"
  exit 1
fi

###############################################
# STEP 1 — Clone the foreign repo
###############################################

echo "Cloning foreign repo..."
git clone "$FOREIGN_REPO_URL" temp-repo
cd temp-repo

###############################################
# STEP 2 — Remove the original remote
# This breaks all ties with the foreign repo.
###############################################

echo "Removing original remote..."
git remote remove origin || true

###############################################
# STEP 3 — Create a new empty repo on GitHub
# Using GitHub API to create a clean repo
###############################################

echo "Creating new repo '$NEW_REPO_NAME' on GitHub..."

curl -s \
  -H "Authorization: token $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  https://api.github.com/user/repos \
  -d "{\"name\":\"$NEW_REPO_NAME\",\"private\":false}"

echo "New repo created."

###############################################
# STEP 4 — Add your new repo as origin
###############################################

NEW_REPO_URL="https://github.com/$GITHUB_USER/$NEW_REPO_NAME.git"

echo "Adding new origin: $NEW_REPO_URL"
git remote add origin "$NEW_REPO_URL"

###############################################
# STEP 5 — Push everything to your new repo
###############################################

# Detect main branch name
BRANCH=$(git symbolic-ref --short HEAD)

echo "Pushing branch '$BRANCH' to new repo..."
git push -u origin "$BRANCH"

###############################################
# DONE
###############################################

echo "Replica successfully created!"
echo "You now fully own: https://github.com/$GITHUB_USER/$NEW_REPO_NAME"
echo "You can enable GitHub Pages in Settings → Pages."
