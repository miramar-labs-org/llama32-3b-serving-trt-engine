# llama32-3b-serving-trt-engine

[![Build and Push](https://github.com/miramar-labs-org/llama32-3b-serving-trt-engine/actions/workflows/build-push.yaml/badge.svg)](https://github.com/miramar-labs-org/llama32-3b-serving-trt-engine/actions/workflows/build-push.yaml)
[![Deploy](https://github.com/miramar-labs-org/llama32-3b-serving-trt-engine/actions/workflows/deploy.yaml/badge.svg)](https://github.com/miramar-labs-org/llama32-3b-serving-trt-engine/actions/workflows/deploy.yaml)
[![Undeploy](https://github.com/miramar-labs-org/llama32-3b-serving-trt-engine/actions/workflows/undeploy.yaml/badge.svg)](https://github.com/miramar-labs-org/llama32-3b-serving-trt-engine/actions/workflows/undeploy.yaml)

Serves **Llama-3.2-3B-Instruct** via `trtllm-serve` on the Miramar platform. Engine compiled for DGX Spark GB10 (bfloat16, TP=1) from the `llama32-3b-trt-compile` compression project.

## Workflows

| Workflow          | Trigger | Description                                                           |
| ----------------- | ------- | --------------------------------------------------------------------- |
| Build and Push    | Manual  | Bake engine_l4/ into TRT-LLM image, push to GAR (GKE only)           |
| Deploy            | Manual  | Deploy TRT-LLM to DGX (engine_gb10) / AGX (engine_sm87) / GKE (L4)  |
| Undeploy          | Manual  | Remove deployment; GKE also tears down GPU node pool                  |

## Quick start

### DGX (no build needed)
1. Run **Deploy** (`host=dgx`) — `serving-config.yaml` is pre-configured
2. Port-forward and test:
   ```bash
   kubectl port-forward svc/trtllm 8000:8000 -n llama32-3b-serving-trt-engine
   curl http://localhost:8000/v1/models          # returns id: "engine"
   curl -X POST http://localhost:8000/v1/chat/completions \
     -H "Content-Type: application/json" \
     -d '{"model":"engine","messages":[{"role":"user","content":"Hello"}],"max_tokens":100}'
   ```
3. Run **Undeploy** when done

### AGX (no build needed)
Same as DGX but `host=agx` and engine subdir is `engine_sm87/` (must be compiled separately).

### GKE
1. Run **Build and Push** to bake `engine_l4/` into a GAR image (engine must be compiled for L4 sm_89)
2. Run **Deploy** (`host=gke`, `image_tag=latest`)
3. Run **Undeploy** when done

See [CLAUDE.md](CLAUDE.md) for runtime notes, field reference, and troubleshooting.
