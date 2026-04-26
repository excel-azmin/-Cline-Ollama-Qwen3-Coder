## The realistic setup: Cline + Ollama + Qwen3-Coder
This gets you VS Code with an autonomous agent that edits files, runs commands, and iterates — all using a local model.

## Step 1 — Install Ollama

```
curl -fsSL https://ollama.com/install.sh | sh
```

That's it. It auto-detects your NVIDIA GPU and sets up CUDA. Verify:

```
ollama --version
nvidia-smi   # confirm GPU visible
```

## Step 2 — Pull a model that fits your VRAM
Pick one based on your actual VRAM:
24 GB (RTX 3090/4090/5090) — best option:

```
ollama pull qwen3-coder:30b
```

~18 GB on disk, fits with room for context. This is the sweet spot.
16 GB (RTX 4070 Ti Super / 4080):

```
ollama pull qwen2.5-coder:14b
```

12 GB (RTX 3060 12GB / 4070):

```
ollama pull qwen2.5-coder:7b
```

8 GB (RTX 3060 8GB / 4060):

```
ollama pull qwen2.5-coder:7b-instruct-q4_K_M
```

Tight, expect to close other GPU apps.
Test it works:

```
ollama run qwen3-coder:30b "write a python function that reverses a string"
```

## Step 3 — Increase the context window

Ollama defaults to 2048 tokens of context, which is uselessly small for agentic coding. You need at least 32k. Create a Modelfile:

```
cat > Modelfile <<EOF
FROM qwen3-coder:30b
PARAMETER num_ctx 32768
EOF
```

ollama create qwen3-coder-32k -f Modelfile

(Adjust num_ctx down to 16384 or 8192 if you run out of VRAM. Bigger context = more VRAM.)

## Step 4 — Install Cline in VS Code

In VS Code: Extensions → search "Cline" → install (publisher is "Cline" / saoudrizwan.claude-dev).
Step 5 — Wire Cline to Ollama
Open Cline (left sidebar robot icon) → Settings gear → API Provider:

* API Provider: Ollama
* Base URL: http://localhost:11434
* Model ID: qwen3-coder-32k (or whatever you named it)

* Configure IP accsible 

```
sudo systemctl edit ollama.service

# Pas it 
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_ORIGINS=*"

# Restart Daemon
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

Save. Done.

## Step 6 — Vibe code

Open Cline's chat in VS Code, type something like:

> Build me a small FastAPI app with a /todos endpoint backed by SQLite. Add tests.

It'll plan, ask permission to create files, write them, run commands, iterate. Approve each step the first few times so you see what it's doing.
