# model-router-launchers

Two tiny launchers that turn [llama.cpp](https://github.com/ggml-org/llama.cpp) (`llama serve` router mode) into a two-lane local model server on one machine with an 8GB unified-memory iGPU:

- **main router — :8091** hot-swaps one big + one small model (e.g. ling-3.0-tiny primary, LFM 2.5/8b/12b, gemma VL, ui-venus VL)
- **micro router — :8092** co-resides alongside it with two tiny models (~2GB combined) for fast-lane traffic

Both are OpenAI-compatible (`/v1`), bound to 127.0.0.1, hot-swap on demand.

## Why

One GPU, many models. Instead of picking a model per task, both lanes stay hot and swap models in/out as requests arrive — you get big-model quality and tiny-model latency from the same box, fully offline.

## Install

```bash
# build llama.cpp with the router/serve target, then:
cp start-model-router start-model-router-micro ~/.local/bin/
chmod +x ~/.local/bin/start-model-router*

# point at your model directory (preset .ini files live there too):
export MODEL_ROUTER_DIR=/path/to/router-models
```

Edit the header comments to list your own lineup, then:

```bash
start-model-router start|stop|status|logs
start-model-router-micro start|stop|status
```

## The trap worth knowing

`/v1/models` answers instantly even before models are loaded and usable. If you're scripting against this, verify with a real `/chat/completions` — don't trust the models listing.

## Tuning

- `MAX=2` models resident per lane — raise if you have more memory
- `-t 4 -tb 6` thread split — match to your CPU
- Both presets (`models-preset.ini`, `models-micro.ini`) are plain llama.cpp router preset files listing your GGUFs

## License

MIT
