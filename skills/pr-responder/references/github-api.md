# GitHub API Reference for PR Comments

## Get PR for Current Branch

```bash
gh pr view --json number,title,url,state,baseRefName,headRefName
```

## Get Review Comments (Inline on Code)

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  --jq '.[] | {id, path, line, body, user: .user.login, created_at, in_reply_to_id}'
```

## Get Issue Comments (General Discussion)

```bash
gh api repos/{owner}/{repo}/issues/{pr_number}/comments \
  --jq '.[] | {id, body, user: .user.login, created_at}'
```

## Get Repo Owner and Name

```bash
gh repo view --json owner,name --jq '"\(.owner.login)/\(.name)"'
```

## Get Review Threads with Resolution Status

```bash
gh api graphql -f query='
  query($owner: String!, $repo: String!, $pr: Int!) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $pr) {
        reviewThreads(first: 100) {
          nodes {
            isResolved
            comments(first: 10) {
              nodes {
                body
                author { login }
                path
                line
              }
            }
          }
        }
      }
    }
  }
' -f owner=OWNER -f repo=REPO -F pr=PR_NUMBER
```

## Reply to a Review Comment

```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  -f body="Your reply here" \
  -f in_reply_to={comment_id}
```

## Resolve a Review Thread

```bash
gh api graphql -f query='
  mutation($threadId: ID!) {
    resolveReviewThread(input: {threadId: $threadId}) {
      thread { isResolved }
    }
  }
' -f threadId=THREAD_NODE_ID
```

## Full Workflow Example

```bash
# 1. Get PR info
PR_JSON=$(gh pr view --json number,title,headRefName)
PR_NUM=$(echo "$PR_JSON" | jq -r '.number')

# 2. Get repo info
REPO=$(gh repo view --json owner,name --jq '"\(.owner.login)/\(.name)"')
OWNER=$(echo "$REPO" | cut -d'/' -f1)
REPO_NAME=$(echo "$REPO" | cut -d'/' -f2)

# 3. Fetch all review comments
gh api "repos/$OWNER/$REPO_NAME/pulls/$PR_NUM/comments" \
  --jq '.[] | {path, line, body, user: .user.login}'

# 4. Fetch general comments
gh api "repos/$OWNER/$REPO_NAME/issues/$PR_NUM/comments" \
  --jq '.[] | {body, user: .user.login}'
```
