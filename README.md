# RunPod Manager

An agent charter for managing a [RunPod](https://runpod.io) account through
[`runpodctl`](https://github.com/runpod/runpodctl). Give it an API key; it
manages pods, serverless endpoints, templates, volumes and billing, and keeps
a written memory of what exists and why in `memories/`.

- `SYSTEM_PROMPT.md` — the operating rules the agent follows
- `bin/rp` — wrapper that loads the key from `.env` and runs `runpodctl`
- `memories/` — persistent state the agent reads and writes each session
- `CLAUDE.md` — tells Claude Code to load the above automatically

## Setup

### 1. Install runpodctl

```bash
# macOS
brew install runpod/runpodctl/runpodctl

# Linux / WSL
wget -qO- cli.runpod.net | sudo bash

# conda / mamba / pixi
conda install conda-forge::runpodctl

runpodctl version
```

### 2. Create a RunPod API key

Per the [RunPod docs](https://docs.runpod.io/get-started/credentials#create-an-api-key):

1. In the RunPod console, open the [Credentials page](https://console.runpod.io/user/credentials).
2. Select the **API Keys** tab and click **Create API Key**.
3. Give the key a name (e.g. `runpod-manager-agent`) and set its permissions:
   - **All** — full access. Pick this so the manager can create, stop and delete resources.
   - **Restricted** — choose access per API endpoint (None / Read Only / Read/Write).
   - **Read Only** — the manager can inspect but not change anything.
4. Click **Create**, then click the new key to copy it.

RunPod does not store the key, so save it somewhere secure (e.g. a password
manager) as well as in `.env`. To edit permissions, disable, or revoke it later:
**Credentials → API Keys** → pencil icon, toggle, or trash icon → **Revoke Key**.

### 3. Add the key to `.env`

```bash
git clone https://github.com/vsheshjain/runpod-manager.git
cd runpod-manager
cp .env.example .env
# edit .env and paste the key after RUNPOD_API_KEY=
```

`.env` is gitignored. Don't paste the key into chat or commit it.

### 4. SSH key for pods (optional)

`.env` holds the **path** to your private key, not the key itself:

```bash
RUNPOD_SSH_KEY=$HOME/.ssh/runpod
```

No key yet? `ssh-keygen -t ed25519 -f ~/.ssh/runpod -C runpod`. Register the
public half with your account once (needs the API key from step 3):

```bash
set -a; source .env; set +a
bin/rp ssh add-key --key-file "$RUNPOD_SSH_KEY.pub"
bin/rp ssh list-keys
```

Then connect to a pod:

```bash
bin/rp ssh info <pod-id>
ssh -i "$RUNPOD_SSH_KEY" <user>@<host> -p <port>
```

The key only reaches pods created after it was added.

### 5. Verify

```bash
bin/rp doctor
bin/rp pod list
```

A JSON list (possibly `[]`) means it works. `"code":"no_credentials"` means
`.env` wasn't picked up; `"code":"unauthorized"` means the key is wrong or
revoked.

### 6. Start the manager

```bash
claude        # from inside the repo
```

Claude Code reads `CLAUDE.md`, which points it at `SYSTEM_PROMPT.md` and
`memories/`. Then just ask: "what pods do I have running?", "spin up a 4090
with pytorch", "stop everything idle", "invoke endpoint X with this payload".

With another agent, load `SYSTEM_PROMPT.md` as its system prompt and run it
from the repo root.

## Usage

Always go through `bin/rp` so the key comes from `.env`:

```bash
bin/rp gpu list
bin/rp pod create --image runpod/pytorch:2.8.0-py3.11-cuda12.8.1-cudnn-devel-ubuntu22.04 \
  --gpu-id "NVIDIA GeForce RTX 4090" --wait
bin/rp pod get <id>
bin/rp pod logs <id> --tail 50
bin/rp pod stop <id>
bin/rp serverless list
bin/rp serverless run <id> --input '{"prompt":"hello"}'
```

Output is JSON on stdout, errors are JSON on stderr — pipe into `jq`.

## Safety

The agent confirms before anything that costs money or destroys data:
creating compute (with the $/hr stated), `--workers-min ≥ 1`, deleting pods,
endpoints or volumes. Details in `SYSTEM_PROMPT.md` §9.

## Memory

After each session that changes something, the agent records in `memories/`
what the API can't tell it: why a pod exists, what's on each volume, which
endpoint is production, and what broke and how it was fixed. No secrets are
ever written there. Conventions in `SYSTEM_PROMPT.md` §11.
