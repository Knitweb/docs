# VirtualPC + Alexander coordination

## Runtime

The two services are supervised by the user systemd manager:

- `virtualpc.service` — VirtualPC API on `127.0.0.1:3100`;
- `alexander.service` — Alexander GUI on `127.0.0.1:7878` and LiteLLM on
  `127.0.0.1:4002`;
- `knitweb-agents-governor.service` — host-level resource guard.
- `knitweb-backlog-feeder.timer` — refreshes selected `febuz` and `Knitweb`
  development issues every 10 minutes when fewer than three unattempted items
  remain.

All three units are enabled for automatic restart after user-session startup.
Alexander's autonomous queue is `/media/knight2/EDS2/projects/alexander/runtime/AUTONOMOUS-BACKLOG.md`.
When organization queues are empty, the feeder uses the local
`VIRTUALPC-FALLBACK-BACKLOG.md` maintenance queue.

## Resource policy

The services share `knitweb-agents.slice` with:

- CPU quota: 7680% on the measured 96-CPU host, equal to 80% aggregate;
- memory limit: 100 GiB, below 80% of the measured 125 GiB host memory;
- per-service memory ceilings: VirtualPC 36 GiB, Alexander 64 GiB;
- GPU utilization guard: pause at 80% utilization;
- host hysteresis: pause at 80% CPU/memory/GPU, resume below 70%.

The governor sends `SIGSTOP` to both service cgroups during a high-load phase,
then resumes them with `SIGCONT`. This preserves service state and prevents a
restart from bypassing the cap.

## Current state

At the time of setup, unrelated host workloads were using approximately 99% CPU.
The agent services are therefore intentionally paused. They will resume
automatically when the host falls below the 70% recovery threshold.

The first autonomous security task was picked and failed without a code change
because the old Ollama backend was unavailable. Alexander is now configured to
use the available local LM Studio models through its own LiteLLM proxy on port
4002; the next retry remains resource-gated.
