# Mirror Repo GitHub Action

A simple GitHub Actions workflow to mirror (sync) your repository to another git host or repo.
Handles commits, branches, tags, etc. Finishes with a clean job summary.

Why? Because backups are important. Decentralization is healthy. And vendor lock-in is bad.

## Quick start

1) Add a deploy key with write access on your target host (e.g. Codeberg).
2) Save the private key in your GitHub repo as a secret: **`MIRROR_SSH`**.
3) Add a workflow, in your repo, at: `.github/workflows/mirror.yml`

```yaml
name: Mirror
on:
  schedule: [{ cron: '30 01 * * 0' }] # Choose a frequency (e.g. mirror every Sunday at 01:30)
  push: { tags: ['v*'] }
  workflow_dispatch:

jobs:
  mirror:
    runs-on: ubuntu-latest
    steps:
      - uses: lissy93/mirror@v1
        with:
          ssh_key: ${{ secrets.MIRROR_SSH }}  # required! Your SSH private key as a secret
          # host: git@codeberg.org            # optional, defaults to Codeberg (git@codeberg.org)
          # user: your-user-or-org            # optional, defaults to your repo owner
          # repo: target-repo-name            # optional, defaults to your repo name
          # force_push: false                 # optional, force push to overwrite remote (default: false)
```

That’s it! The action will:
- set up SSH,
- add `mirror` remote,
- `git push mirror --all --tags`,
- and post a summary with logs.

## Inputs

| name         | required | default             | meaning                                          |
| ------------ | -------- | ------------------- | ------------------------------------------------ |
| `ssh_key`    | ✅        | —                   | Private key with write access                   |
| `host`       | ❌        | `git@codeberg.org`  | SSH host for the target git server               |
| `user`       | ❌        | caller's repo owner | Target user/org (derived from context)           |
| `repo`       | ❌        | caller's repo name  | Target repo name (derived from context)          |
| `force_push` | ❌        | `false`             | Force push to overwrite remote history (caution) |

## Outputs

| name         | description                              | example                             |
| ------------ | ---------------------------------------- | ----------------------------------- |
| `target_url` | The target repository URL                | `git@codeberg.org:user/repo.git`    |
| `success`    | Whether the mirror operation succeeded   | `true`                              |


---

## How it works

Git mirroring copies your entire repository to another location. It's useful for backups, avoiding vendor lock-in, or keeping repos in sync across platforms.

The action runs on a schedule (like weekly) or when triggered (like on new tags). It checks out your repo with full history, authenticates, then safely pushes everything to your target git host. All branches, commits, and tags from your source repo get pushed to the mirror destination. No changes will be made to your source repo.

---

## Step-by-Step Example

First time mirroring a repo? We got you! Below is a step-by-step guide for mirroring a repo from GitHub to Codeberg (but will work the same for any git server).

### 1. Create a Destination Repo
If you've not already got one, head over to Codeberg, create an account and then a new repo.

### 2. Create an SSH key

Use `ssh-keygen` or a tool of your choice, to generate a new SSH key, and then note down the public and private key.

```bash
# Generate new SSH key
ssh-keygen -t ed25519 -a 64 -C "github-mirror-to-codeberg" -f ./codeberg_mirror_ed25519 -N ""

# Show the public Key (for destination repo, e.g. Codeberg)
cat ./codeberg_mirror_ed25519.pub

# Show the private key (for source repo, e.g. GitHub)
cat ./codeberg_mirror_ed25519
```

<details>
<summary>Show me ℹ️</summary>

![screenshot of creating ssh key](https://pixelflare.cc/alicia/screenshots/mirror-create-ssh-key)

<sub>(this demo key was invalidated, deleted and never used!)</sub>

</details>

### 3. Add Public Key to new Git Host
Log into Codeberg, and click your account (in the top-right) --> Settings --> SSH / GPG Keys --> Manage SSH --> Add Key.

Pick a good name, and then add your newly generated public key (e.g. from `./codeberg_mirror_ed25519.pub`)

<details>
<summary>Show me ℹ️</summary>

![screenshot of adding a new public SSH key into CodeBerg](https://pixelflare.cc/alicia/screenshots/mirror-add-public-key)

</details>

### 4. Add Private Key to Source Repo

Next, head back to your GitHub repo, and under the repo Settings --> Secrets --> Actions --> New repository secret.

Name it `MIRROR_SSH` (or whatever), then paste your private key (e.g. from `./codeberg_mirror_ed25519`)

<details>
<summary>Show me ℹ️</summary>

![screenshot of adding a your private SSH key into GitHub as a repository secret](https://pixelflare.cc/alicia/screenshots/mirror-add-private-key)

</details>

### 5. Create the Workflow

Create a new file, within the `.github/workflows` directory. E.g. `.github/workflows/mirror.yml`.


```yaml
name: Mirror
on:
  schedule: [{ cron: '30 01 * * 0' }]
  push: { tags: ['v*'] }
  workflow_dispatch:

jobs:
  mirror:
    runs-on: ubuntu-latest
    steps:
      - uses: lissy93/repo-mirror-action@main
        with:
          ssh_key: ${{ secrets.MIRROR_SSH }}
          host: git@codeberg.org
          user: alicia
          repo: pixelflare
```

<details>
<summary>Show me ℹ️</summary>

![screenshot of creating a new workflow](https://pixelflare.cc/alicia/screenshots/mirror-create-workflow)

</details>


### 6. Run the Workflow

Head to your repository's Actions tab. Click your new workflow (on the left), then in the top-right, click `Run workflow`

<details>
<summary>Show me ℹ️</summary>

![screenshot of running the workflow](https://pixelflare.cc/alicia/screenshots/mirror-run-workflow)

</details>

---

## More Examples

### Force push to fix out-of-sync mirrors

If your mirror gets out of sync, you can force push to overwrite the remote:

```yaml
- uses: lissy93/mirror@v1
  with:
    ssh_key: ${{ secrets.MIRROR_SSH }}
    force_push: true  # ⚠️ Use with caution - overwrites remote history
```

### Using outputs

Chain the mirror action with other steps:

```yaml
- uses: lissy93/mirror@v1
  id: mirror
  with:
    ssh_key: ${{ secrets.MIRROR_SSH }}
- run: echo "Mirrored to ${{ steps.mirror.outputs.target_url }}"
- name: Notify on success
  if: steps.mirror.outputs.success == 'true'
  run: curl -X POST ${{ secrets.WEBHOOK_URL }} -d "Mirror complete"
```

### Mirror to GitLab

```yaml
- uses: lissy93/mirror@v1
  with:
    ssh_key: ${{ secrets.GITLAB_SSH }}
    host: git@gitlab.com
    user: your-gitlab-username
    repo: your-gitlab-project
```

### Mirror to self-hosted Gitea

```yaml
- uses: lissy93/mirror@v1
  with:
    ssh_key: ${{ secrets.GITEA_SSH }}
    host: git@git.example.com
    user: myorg
    repo: myproject
```

### Mirror with user-specified settings

When you run the following workflow, GitHub will popup with a little form, which you can then fill in to specify the target repo settings.

```yaml
name: Mirror to Somewhere

on:
  workflow_dispatch:
    inputs:
        host:
          description: "SSH host (leave empty for Codeberg)"
          required: false
        user:
          description: "Target user/org (leave empty for current repo owner)"
          required: false
        repo:
          description: "Target repo name (leave empty for current repo name)"
          required: false
        force_push:
          description: "Force push to overwrite remote"
          required: false
          type: boolean

jobs:
  test-mirror:
    runs-on: ubuntu-latest
    steps:
      - name: Mirror
        uses: lissy93/repo-mirror-action@main
        with:
          ssh_key: ${{ secrets.MIRROR_SSH_TEST }}
          host: ${{ inputs.host || 'git@codeberg.org' }}
          user: ${{ inputs.user || github.repository_owner }}
          repo: ${{ inputs.repo || github.repository }}
          force_push: ${{ inputs.force_push || false }}
```

---

## Versioning

Use a major tag for stability:

```yaml
- uses: lissy93/mirror@v1
```

I'll keep `v1` pointing to the latest compatible release.

---

## Contributing

See [Contributing Guide](CONTRIBUTING.md)

---

## Security

See [Security Guide](SECURITY.md)

---

## Troubleshooting

<details>
<summary>Permission denied (publickey)</summary>

- Ensure the SSH public key is added to the target host with write access
- Verify the private key matches the public key
- Check the target repository exists
</details>

<details>
<summary>Permission denied (publickey)"</summary>

- Ensure the SSH public key is added to the target host with write access
- Verify the private key matches the public key
- Check the target repository exists
</details>


<details>
<summary>Failed to retrieve SSH host keys</summary>

- Network issue or invalid hostname
- Try manually: `ssh-keyscan your-host.com`
- Check firewall/proxy settings
</details>


<details>
<summary>Repository not found</summary>
- Create the destination repository first on the target host
- Verify user/org and repo names are correct

</details>

<details>
<summary>fatal: refusing to merge unrelated histories</summary>

- The repos have diverged
- Use `force_push: true` to overwrite (⚠️ caution)
</details>


<details>
<summary>Out of sync" errors</summary>

- Enable `force_push: true` to reset the mirror
- Or manually fix the remote repository
</details>

---

## Limitations

- Only supports SSH authentication (not HTTPS/tokens)
- Requires target repository to exist beforehand
- Large repositories (>1GB) may take longer to mirror
- Rate limits depend on your target git host
- Cannot mirror GitHub-specific features (Issues, PRs, Actions)

---

## Attributions

##### Contributors

![contributors](https://readme-contribs.as93.net/contributors/lissy93/repo-mirror-action)

##### Sponsors

![sponsors](https://readme-contribs.as93.net/sponsors/lissy93)

---

## License


> _**[Lissy93/repo-mirror-action](https://github.com/Lissy93/repo-mirror-action)** is licensed under [MIT](https://github.com/Lissy93/repo-mirror-action/blob/HEAD/LICENSE) © [Alicia Sykes](https://aliciasykes.com) 2026._<br>
> <sup align="right">For information, see <a href="https://tldrlegal.com/license/mit-license">TLDR Legal > MIT</a></sup>

<details>
<summary>Expand License</summary>

```
The MIT License (MIT)
Copyright (c) Alicia Sykes <alicia@omg.com> 

Permission is hereby granted, free of charge, to any person obtaining a copy 
of this software and associated documentation files (the "Software"), to deal 
in the Software without restriction, including without limitation the rights 
to use, copy, modify, merge, publish, distribute, sub-license, and/or sell 
copies of the Software, and to permit persons to whom the Software is furnished 
to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included install 
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED,
INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANT ABILITY, FITNESS FOR A
PARTICULAR PURPOSE AND NON INFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

</details>

<!-- License + Copyright -->
<p  align="center">
  <i>© <a href="https://aliciasykes.com">Alicia Sykes</a> 2026</i><br>
  <i>Licensed under <a href="https://gist.github.com/Lissy93/143d2ee01ccc5c052a17">MIT</a></i><br>
  <a href="https://github.com/lissy93"><img src="https://pixelflare.cc/alicia/images/octoface.png?w=56" /></a><br>
  <sup>Thanks for visiting :)</sup>
</p>

<!-- Dinosaurs are Awesome -->
<!-- 
                        . - ~ ~ ~ - .
      ..     _      .-~               ~-.
     //|     \ `..~                      `.
    || |      }  }              /       \  \
(\   \\ \~^..'                 |         }  \
 \`.-~  o      /       }       |        /    \
 (__          |       /        |       /      `.
  `- - ~ ~ -._|      /_ - ~ ~ ^|      /- _      `.
              |     /          |     /     ~-.     ~- _
              |_____|          |_____|         ~ - . _ _~_-_
-->
