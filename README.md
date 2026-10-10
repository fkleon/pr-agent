# PR-Agent local runner

This repository runs [PR-Agent](https://github.com/The-PR-Agent/pr-agent) in Docker Compose against a local Git checkout. The Compose configuration selects PR-Agent's `local` Git provider and GitHub Copilot models. Local runs write results into the mounted checkout rather than posting comments to a hosted pull request.

## Requirements

- Docker Engine with the Docker Compose plugin
- A local Git checkout to analyze
- Authentication for the language-model provider configured in `docker-compose.yml`

The Docker build installs PR-Agent from source in `docker/pr-agent/`. If that directory is missing (for example, in a fresh clone of this wrapper), check out the source there:

```bash
git clone https://github.com/The-PR-Agent/pr-agent.git docker/pr-agent
```

## Configure

1. **Mount your projects directory.** In `docker-compose.yml`, the `./projects` host directory is mounted at `/projects` in the container. Change the host-side path to the directory containing the checkout you want to analyze.
2. **Configure model authentication.** The Compose file selects `github_copilot` models. Follow PR-Agent's [GitHub Copilot model setup](https://docs.pr-agent.ai/usage-guide/changing_a_model/#github-copilot). A named Docker volume (`litellm-data`) is mounted at `/root/.config/litellm` to retain LiteLLM state between runs. The optional access-token file mount in the Compose file is commented out by default.
3. **Review credentials before adding them.** `secrets.toml` is bind-mounted into the container for PR-Agent provider credentials. Do not put real credentials in a tracked file. The current `.gitignore` excludes `*.secret`, but does not exclude `secrets.toml` or `agent.env`; add appropriate ignore rules or use an ignored local file and update the Compose mount before storing private values. The local Git provider does not require a hosted Git-provider token.

The Compose settings, including model names and logging options, are in `docker-compose.yml`. `.env` supplies the `VERSION` value used for the local image tag. `agent.env` is passed to the container as an environment file.

## Build and run

Build the image from the checked-out PR-Agent source:

```bash
docker compose build pr-agent
```

The target repository must be available under the mounted `/projects` directory. Run PR-Agent from that repository's container path, with `HEAD` on the branch you want to analyze and a local target branch present (for example, `main`):

```bash
docker compose run --rm -it \
  -w /projects/my-project \
  pr-agent --pr_url=main review
```

Replace `/projects/my-project` with the checkout's path relative to the mounted projects directory, and replace `main` with the target branch. The local provider compares the current `HEAD` with that target branch. By default, `review` writes `review.md` and `describe` writes `description.md` in the checkout.

For example, to generate a description instead:

```bash
docker compose run --rm -it \
  -w /projects/my-project \
  pr-agent --pr_url=main describe
```

## References

- [PR-Agent local Git provider](https://docs.pr-agent.ai/usage-guide/local_git_provider/)
- [PR-Agent model configuration](https://docs.pr-agent.ai/usage-guide/changing_a_model/)
