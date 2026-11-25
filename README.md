# ztrx

**ztrx** runs the Xbox Series build of the game natively on Windows PCs with NVIDIA graphics cards, from game files you supply yourself.

**This repository contains no game files or assets.** You need your own copy of the game. Not affiliated with or endorsed by its developer or publisher.

## Status (2026-11-25)

Tested on Windows 11 with an **NVIDIA GeForce RTX 3050** (8 GB):

| | |
|---|---|
| Boot, first-boot screens, intro, opening credits | works |
| Prologue | playable from the cell to the boat chase: cell, cafeteria, robbery, police chase (wanted level, minimap), boat, binoculars |
| Stability | 1 h 30 min+ in one session without a crash |
| Frame rate | 1-3 fps in the world (the first run is slower while shaders translate; they are cached after that) |
| Known problems | anything in shadow is black and bright scenes overexpose (the game's ray-traced lighting does not run yet); some blocky artifacts; boats sit low in the water |

## Requirements

- Windows 10/11 x64, a CPU with AVX2
- A Direct3D 12 GPU with shader model 6.6 and mesh shaders (NVIDIA RTX 20 series or newer)
- Rust (the version in `rust-toolchain.toml`)
- Several GB of free Windows commit memory (raise the paging file if `crashes.log` reports error 1455)

## Build

```powershell
cargo build --release -p victory
```

## Play

```powershell
powershell -ExecutionPolicy Bypass -File scripts\play.ps1 -Game "<your game folder>\<the game's exe>"
```

`scripts\play.ps1` keeps the shader cache and saves in folders you choose (`-Cache`, `-Data`) and presses no buttons for you. Controller: XInput pads (a DualShock/DualSense through DS4Windows works). With no pad connected the keyboard acts as one while the game window has focus.

## Faster settings (optional, off by default)

`VICTORY_RELAY=2`, or `VICTORY_LATE_WAIT=3 VICTORY_WIDE_BACK=1` cut frame time on the test machines. Set them in PowerShell before running the script.

## Debugging switches

| Switch | Effect |
|---|---|
| `VICTORY_CAPTURE=<dir>` + `VICTORY_CAPTURE_FRAMES=<list>` | save those frames as PNG (diagnostics build) |
| `VICTORY_KEYS=30:Enter,35:X` | press keys at those seconds (diagnostics build) |
| `VICTORY_D3D12_DRED=1` | on a GPU failure, report the commands in flight |
| `VICTORY_PIPELINE_WAIT_MS=24` | shorter per-frame wait for pipelines still being built |

Diagnostics build: `cargo build --profile diag -p victory --features diagnostics`

## License

GPL-3.0-or-later (see `LICENSE`). Based on VIctory (Marvisak) and victory-fork (fxceless).

```powershell
powershell -ExecutionPolicy Bypass -File scripts\play.ps1 -Game "<your game folder>\<the game's exe>"
```

<<<<<<< HEAD
`scripts\play.ps1` keeps the shader cache and saves in folders you choose
(`-Cache`, `-Data`) and presses no buttons for you. Controller: XInput pads
(a DualShock/DualSense through DS4Windows works). With no pad connected the
keyboard acts as one while the game window has focus.

## Faster settings (optional, off by default)

`VICTORY_RELAY=2`, or `VICTORY_LATE_WAIT=3 VICTORY_WIDE_BACK=1` cut frame
time on the test machines. Set them in PowerShell before running the script.

## Debugging switches

| Switch | Effect |
|---|---|
| `VICTORY_CAPTURE=<dir>` + `VICTORY_CAPTURE_FRAMES=<list>` | save those frames as PNG (diagnostics build) |
| `VICTORY_KEYS=30:Enter,35:X` | press keys at those seconds (diagnostics build) |
| `VICTORY_D3D12_DRED=1` | on a GPU failure, report the commands in flight |
| `VICTORY_PIPELINE_WAIT_MS=24` | shorter per-frame wait for pipelines still being built |

Diagnostics build: `cargo build --profile diag -p victory --features diagnostics`
=======
| Variável | Efeito |
|----------|--------|
| `VICTORY_LOG=<target>=debug,...` | saída de debug e trace |
| `VICTORY_STACKS=20,60` | pilha de cada thread nesses segundos |
| `VICTORY_WATCH=<rva>,...` | registra um u32 do título quando muda |
| `VICTORY_KEYS=30:Enter,35:X` | pressiona teclas nesses segundos, com ou sem foco |
| `VICTORY_CRASH_SAVE=<rva>:<len>,...` | salva memória do título para a pasta de cache em um crash |
| `VICTORY_CAPTURE=<dir>` | salva frames apresentados (potências de dois, ou `VICTORY_CAPTURE_FRAMES=<list>`) |
| `VICTORY_DUMP_TARGETS=1` | com os dois acima, salva todos os alvos de renderização daqueles frames |
| `VICTORY_D3D12_DEBUG=1` | camada de depuração do host |
| `VICTORY_D3D12_GBV=1` | com o acima, validação baseada em GPU (uso ruim de descritor reportado, não travado; lento) |
| `VICTORY_D3D12_DRED=1` | em uma falha de GPU, a alocação atingida e os comandos em voo |
| `VICTORY_PIPELINE_WAIT_MS=24` | reduz a espera por frame para a criação de pipeline em segundo plano do padrão de 500 ms. Um desenho pode ser omitido até que seu pipeline esteja pronto |
| `VICTORY_SKIP_PIPELINES=<hash>,...` | omite draws e dispatches, incluindo indiretos, cujo pipeline contém um hash de shader listado de 16 dígitos |
| `VICTORY_GPU_TIMING=1` | com o acima, cada lista de uma execução de submissão é cronometrada por conta própria; listas lentas registradas com seus comandos mais lentos |
| `VICTORY_D3D12_WARP=1` | dispositivo do host no adaptador de software (lento; não pode redefinir o driver) |
| `VICTORY_SHADER_LOOP_LIMIT=<n>` | loops de shader traduzidos param após `n` iterações por via em vez de 2^20 (entradas próprias de cache de shader); se um `n` mais baixo tornar os frames do mundo muito mais rápidos, loops que atingem o limite são o que os tornam lentos |
| `VICTORY_AUDIO_CAPTURE=<file.wav>` | grava o que o título toca (nosso processo apenas) |
| `VICTORY_XMA_DUMP=<dir>` | salva o contexto e entrada das primeiras 32 vozes XMA |

Cada compilação anexa falhas e falhas de GPU do host a `crashes.log` na pasta de dados do título.
>>>>>>> fork/main

## Licença

<<<<<<< HEAD
GPL-3.0-or-later (see `LICENSE`). Based on VIctory (Marvisak) and victory-fork (fxceless).
=======
GPL-3.0-or-later. Veja [LICENSE](LICENSE).
>>>>>>> fork/main
