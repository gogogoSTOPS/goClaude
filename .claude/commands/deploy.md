---
description: Push local changes to git, pull on server, and restart the server if needed
---

# Deploy to Server

You are tasked with deploying the latest local changes to the production server.

## Process:

1. **Check local state and commit if needed:**
   - Run `git status` to check for uncommitted changes
   - If there are uncommitted changes, invoke the `commit` sub-agent (Task tool, subagent_type=commit) to commit them before proceeding
   - Wait for the commit to complete, then continue
   - Run `git log origin/main..HEAD --oneline` to see what commits will be pushed

2. **Push to git:**
   - Run `git push origin main`
   - If push fails, report the error and stop

3. **SSH into server and deploy:**
   - Connect: `ssh -i ~/.ssh/slate_server ubuntu@140.245.216.42`
   - On the server, run the following sequence:
     ```bash
     cd ~/slate && git pull origin main
     ```
   - Check if any of these files changed in the pull (they require a Docker rebuild):
     - `Dockerfile`
     - `requirements_server.txt`
     - `server.py`
     - `engine/`
     - `templates/`
     - `static/`
   - **If a rebuild is needed** (any of the above changed):
     ```bash
     docker compose build && docker compose up -d
     ```
   - **If only config/non-code files changed** (e.g. `.env`, `data/`, docs):
     - No restart needed; confirm server is still running with `docker compose ps`
   - Wait a few seconds and verify the container is healthy:
     ```bash
     docker compose ps
     ```

4. **Report back:**
   - Confirm what commits were deployed
   - State whether a rebuild/restart was done or skipped
   - Show the final container status

## Additional Server Commands

If the user provides additional context or instructions (e.g., "run the backfill migration", "run migrations"), execute those commands on the server after the deploy is complete and the container is healthy. Run them inside the container using `docker compose exec slate <command>`.

## Notes:
- SSH key: `~/.ssh/slate_server`, user: `ubuntu`, host: `140.245.216.42`
- App directory on server: `~/slate`
- Docker service name: `slate`
- Server runs via `docker compose` (not `docker-compose`)
- The `access.md` file in the repo root has connection details
