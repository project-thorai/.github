# Org rulesets for Thor graphs

The branch rules a Thor graph relies on, as org rulesets for every repository in project-thorai, including ones created later (`repository_name: ~ALL`).

| File | Branches | Rules | Who bypasses |
| --- | --- | --- | --- |
| `thorai-branches-thor-only.json` | `thorai/**` | create, update (so push and merge) | the Thor App `thorai-app` (App ID 5505187) |
| `thorai-branches-kept.json` | `thorai/**` | delete, force-push | nobody |
| `default-branch-people-merge.json` | each repository's default branch | update | org owners and people with write, maintain or admin, through a pull request only |

GitHub has no rule for "nobody but X" on a default branch, so the third ruleset restricts updates and lets people bypass it through pull requests; the Thor App holds no repository role, so it can't. It sits beside the existing "Protect main and dev", "Require 1 approval on main" and "Thor approves" rulesets, which still apply to people.

Rulesets can't make deletion depend on an open pull request, so `thorai/` branches can't be deleted at all while the second ruleset is active.

Reviews and comments are not rules, so people can still review and comment on pull requests into `thorai/` branches.

## Apply

Org rulesets need an org owner. With `gh` signed in as one, from this directory:

```sh
gh auth refresh -h github.com -s admin:org
for f in *.json; do gh api orgs/project-thorai/rulesets --method POST --input "$f" --jq '.id, .name'; done
```

To change one later, edit its file and replace it by ID:

```sh
id=$(gh api orgs/project-thorai/rulesets --jq '.[] | select(.name == "<name>") | .id')
gh api "orgs/project-thorai/rulesets/$id" --method PUT --input <file>.json
```

## Check

1. The rules on a `thorai/` branch and on `main`, readable by anyone with access to the repository:

   ```sh
   gh api repos/project-thorai/thor-tag/rules/branches/thorai/eng-1155-graph --jq '.[].type'   # creation, update, deletion, non_fast_forward
   gh api repos/project-thorai/thor-tag/rules/branches/main --jq '.[].type'                    # includes update
   ```

2. As a person, merge a pull request into a `thorai/` branch: refused, and push to one: refused.
3. As Thor, `thor-graph mark <key> done` on a node: its pull request merges into the graph's `thorai/` branch.
4. As Thor, merge a pull request into `main` (`gh pr merge` with Thor's token): refused.
5. As a person, merge an approved pull request into `main`: allowed, as before.
6. Create a new repository and run step 1 against it: the same rules, with no setup.
