# Qwen Code — Docker Lab

> Run [Qwen Code](https://github.com/QwenLM/qwen-code) inside a Linux Docker container.
> Nothing gets installed on your Windows machine except Docker Desktop.
> The container can only see the folder you choose — your host stays clean and safe.

---

## Part 1 — Install Docker Desktop

1. Download Docker Desktop from:
   https://www.docker.com/products/docker-desktop/

2. Run the installer. Accept the defaults.

3. When it finishes, **restart your computer**.

4. Open Docker Desktop and wait until the bottom-left icon turns **green** (Engine running).

5. Open **PowerShell** and verify:

```powershell
docker --version
```

You should see something like `Docker version 28.x.x`.

---

## Part 2 — Create a Container from Scratch

We will create a Linux container that:

- Has Node.js 20 pre-installed
- Can **only** access your current project folder
- Has no access to the rest of your computer

### 2.1 — Navigate to your project folder

Open PowerShell and `cd` into the folder you want Qwen to work in:

```powershell
cd $HOME\Documents\myproject
```

> Replace `myproject` with your actual project folder.

### 2.2 — Run the container

```powershell
docker run -it `
  --name qwen-dev `
  -v "${PWD}:/workspace" `
  -w /workspace `
  node:20-bookworm `
  bash
```

**What each flag does:**

| Flag | Meaning |
|------|---------|
| `-it` | Interactive mode with a terminal |
| `--name qwen-dev` | Give the container a name so you can find it later |
| `-v "${PWD}:/workspace"` | Mount your **current folder only** into `/workspace` inside the container |
| `-w /workspace` | Start inside `/workspace` |
| `node:20-bookworm` | Use the official Node.js 20 Linux image |
| `bash` | Open a bash shell |

> The first time you run this, Docker will download the Node image (~350 MB). This only happens once.

Your prompt now looks like this — you are **inside Linux**:

```
root@abc123:/workspace#
```

---

## Part 3 — Install Qwen Code Inside the Container

Run this inside the container:

```bash
npm install -g @qwen-code/qwen-code@latest
```

Wait for the install to finish, then verify:

```bash
qwen --version
```

Now start Qwen:

```bash
qwen
```

Qwen is now running and can **only** see files in `/workspace` (your mounted project folder).

### Exiting

- Press `Ctrl + C` to stop Qwen
- Type `exit` to leave the container

---

## Part 4 — Coming Back Next Time (Re-enter the Container)

When you close the terminal, the container **stops** but is **not deleted**. Everything you installed is still there.

### 4.1 — Start the container again

```powershell
docker start qwen-dev
```

### 4.2 — Attach to it

```powershell
docker exec -it qwen-dev bash
```

You are back inside the container. Run `qwen` and continue working.

### 4.3 — Quick one-liner

You can combine both steps:

```powershell
docker start qwen-dev; docker exec -it qwen-dev bash
```

---

## Part 5 — Using a Dockerfile (Repeatable Setup)

Instead of installing Qwen manually every time, create a **Dockerfile** that does it automatically.

### 5.1 — Create the Dockerfile

In your project folder, create a file called `Dockerfile` (no extension) with this content:

```dockerfile
FROM node:20-bookworm

RUN npm install -g @qwen-code/qwen-code@latest

WORKDIR /workspace
```

### 5.2 — Build the image

From PowerShell, in the same folder as the Dockerfile:

```powershell
docker build -t qwen-image .
```

### 5.3 — Run a container from your image

```powershell
docker run -it `
  --name qwen-auto `
  -v "${PWD}:/workspace" `
  -w /workspace `
  qwen-image `
  bash
```

Qwen is already installed — just type `qwen` and you're ready.

### 5.4 — Coming back next time

Same as Part 4, but use the container name `qwen-auto`:

```powershell
docker start qwen-auto; docker exec -it qwen-auto bash
```

---

## Part 6 — Authenticate with Qwen OAuth

Qwen Code gives you **free requests per day** if you sign in with a Qwen account. No API key needed.


```sh
QWEN_OAUTH=1 qwen -p "Hello"
# Should show:
# === Qwen OAuth Device Authorization ===
# Please visit: https://chat.qwen.ai/authorize?user_code=XXXX
```

Qwen will display a URL. **Copy that URL** and open it in a browser **on your Windows machine**. Sign in or create an account at `qwen.ai`, and authorize access.

Once you complete the browser flow, Qwen will detect the login and you're authenticated.

> Your credentials are cached inside the container at `~/.qwen/`, so you usually won't need to log in again — as long as you reuse the same container.

**Alternative: API Key (if OAuth doesn't work in the container)**

> In some Docker setups the OAuth browser flow may not work. In that case, use an API key instead.

Create the settings file inside the container:

```bash
mkdir -p ~/.qwen
cat > ~/.qwen/settings.json << 'EOF'
{
  "modelProviders": {
    "openai": [
      {
        "id": "qwen3-coder-plus",
        "name": "qwen3-coder-plus",
        "baseUrl": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "description": "Qwen3-Coder via Dashscope",
        "envKey": "DASHSCOPE_API_KEY"
      }
    ]
  },
  "env": {
    "DASHSCOPE_API_KEY": "sk-YOUR-KEY-HERE"
  },
  "security": {
    "auth": {
      "selectedType": "openai"
    }
  },
  "model": {
    "name": "qwen3-coder-plus"
  }
}
EOF
```

Replace `sk-YOUR-KEY-HERE` with your actual API key from [Alibaba Cloud ModelStudio](https://modelstudio.console.alibabacloud.com/).

Then start `qwen` — it will use your key automatically.

---

## Part 7 — Re-entering the Container (Summary)

Every time you want to use Qwen Code again:

**Step 1** — Open PowerShell

**Step 2** — Navigate to your project folder:

```powershell
cd $HOME\Documents\myproject
```

**Step 3** — Start and enter the container:

```powershell
docker start qwen-dev; docker exec -it qwen-dev bash
```

**Step 4** — Run Qwen:

```bash
qwen
```

That's it. Your auth credentials and installed packages are preserved between sessions.

> If you used the Dockerfile approach, replace `qwen-dev` with `qwen-auto` in the commands above.

---

## Cheat Sheet

| Task | Command |
|------|---------|
| **First time** — create container | `docker run -it --name qwen-dev -v "${PWD}:/workspace" -w /workspace node:20-bookworm bash` |
| **Install Qwen** (inside container) | `npm install -g @qwen-code/qwen-code@latest` |
| **Start Qwen** (inside container) | `qwen` |
| **Authenticate** (inside Qwen) | `/auth` |
| **Exit Qwen** | `Ctrl + C` |
| **Exit container** | `exit` |
| **Re-enter container** | `docker start qwen-dev; docker exec -it qwen-dev bash` |
| **Build from Dockerfile** | `docker build -t qwen-image .` |
| **Run from Dockerfile image** | `docker run -it --name qwen-auto -v "${PWD}:/workspace" -w /workspace qwen-image bash` |
| **Delete a container** | `docker rm qwen-dev` |
| **List all containers** | `docker ps -a` |
