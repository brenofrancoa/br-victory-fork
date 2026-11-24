# VIctory fork (em andamento)

Um fork em andamento do [VIctory](https://github.com/Marvisak/VIctory), uma camada de compatibilidade que executa um título de Xbox Series nativamente no Windows a partir de arquivos que o próprio usuário fornece. Este fork segue o `main` upstream e acrescenta correções com um único objetivo: fazer o mundo do jogo renderizar corretamente e de forma jogável em um PC específico.

**Este repositório não contém arquivos, código ou assets do jogo.** Você precisa de sua própria cópia do jogo. Não é afiliado nem endossado pelo desenvolvedor ou editora do jogo.

## Créditos

- **[VIctory](https://github.com/Marvisak/VIctory)** por Marvisak e seus colaboradores é a base de tudo aqui: o carregador, os serviços de runtime, o back-end Direct3D 12 e o recompilador de shaders. Licenciado sob GPL-3.0-or-later; este fork também o é. O histórico do upstream é mantido intacto abaixo do único commit que carrega as mudanças deste fork.
- **[XCL](https://github.com/ivall/xcl)** por ivall e seus colaboradores (GPL-2.0-only) foi usado como referência para como o mesmo título se comporta sob outra camada. Correções inspiradas nele foram reescritas do zero no código do VIctory.
- As mudanças deste fork foram desenvolvidas com o [Claude Code](https://claude.com/claude-code).

## Status deste fork (2026-11-25)

**Ainda não é jogável de forma estável: texturas carregam no mundo, mas a GPU remove o dispositivo (crash) após alguns segundos de execução na fase de jogabilidade.**
Testado em uma máquina: Windows 11, Intel i9-12900KF, NVIDIA RTX 3080, 16 GB RAM DDR5. O perfil Series X é usado em todo o jogo.

| Estágio | Estado neste fork |
|-------|--------------------|
| Boot, menus, tela inicial | funciona, 30 fps (como upstream) |
| Cutscene de abertura | reproduz do início ao fim, em ordem, iluminada: cartões de título, cartões de crédito, cenas noturnas e do entardecer |
| Salvamentos | o jogo salva automaticamente um ponto de controle e o carrega novamente |
| Primeira cena de jogabilidade (o prólogo) | renderiza: personagens, texturas, céu, exterior iluminado pelo sol; HUD, menu de pausa e input do controle funcionam; texturas aparecem, mas a GPU é removida (crash) após alguns segundos |
| Luz na sombra | ausente: a luz refletida com rastreamento de raios do jogo e as sombras com rastreamento de raios de lâmpada e sol não executam ainda, então tudo que o sol não atinge diretamente fica preto |
| Taxa de quadros no mundo | cerca de 0,9 fps (cerca de 1,1 s por frame) com as otimizações ativas (`VICTORY_SHADER_LOOP_LIMIT=1000`, `VICTORY_LATE_WAIT=1`, `VICTORY_RELAY=1`); começou em 3-6 segundos por frame |
| Estabilidade | a GPU é removida (device removed) após alguns segundos na fase de jogo quando `VICTORY_RELAY` ou `VICTORY_LATE_WAIT` estão acima do estágio 1; investigar o crash na thread de releio/leitura da memória da placa |

### O que este fork mudou (resumo)

Renderização:
- A culling de meshlets lia seus buffers de cena como o tipo de visualização errado, então o mundo quase não desenhava; corrigido.
- Tabelas de descritores por desenho poderiam se esgotar, após o que os desenhos usavam slots nunca feitos para eles e a placa travava; corrigido.
- A luz empacotada pelos compute shaders do jogo era lida de volta através de cópias vazias (céu, probes, neblina); corrigido para cada nível de mip e camada.
- A reconstrução de neblina, shadow-mask, cabelo, skinning de blend-shape, fogo e vários programas de material não traduziam; corrigidos.
- Gathers em um nível de mip nomeado (com e sem offset) liam nível 0; corrigido.
- Amostras de nível implícito fora de shaders de pixel produziam DXIL inválido em 730 programas de compute; corrigido.
- Listas de retângulo, gêmeos de profundidade e estêncil para texturas de profundidade escritas por compute, leituras de volta de textura.
- Rastreamento de raios, em andamento e desligado por padrão: os próprios programas construtores de estrutura do jogo traduzem, a memória do título pode ser lida por endereço bruto a partir de shaders, o teste de raio/caixa e raio/triângulo de hardware é emulado em DXIL (verificado contra uma referência na GPU), e 59 dos 64 programas de rastreio traduzem. Nada é iluminado por isso ainda. Veja [docs/fork-ray-tracing.md](docs/fork-ray-tracing.md).

Velocidade e memória:
- Esperas por valores de memória, consultas de write-watch e consultas de página não andam mais pelas tabelas de página por chamada.
- Grandes buffers de título recebem uma cópia na memória da placa, com escritas de shader encontradas por hashing na placa.
- Submissões podem ser enviadas antes da espera pela placa (`VICTORY_LATE_WAIT`), com uma thread de releio que pousa os resultados assim que a placa termina (`VICTORY_RELAY`): em uma primeira execução de cinco minutos o tempo ocioso da placa atrás da thread de submissão caiu de cerca de 136 para 30 ms por frame.
- A memória do título pode ser aberta como um heap de gráficos para que buffers sejam usados no lugar em vez de copiados (`VICTORY_INPLACE`, primeiro estágio apenas; um probe confirmou a abordagem na GPU de teste).
- A fila de texturas bindless não faz mais upload de visualizações obsoletas; a tabela de visualizações não cresce mais sem limites; a biblioteca de pipeline é mapeada em vez de lida para a memória.
- Uma perda intermitente de dispositivo foi rastreada até o arquivo da biblioteca de pipeline sendo substituído enquanto mapeado; corrigido.

- Shaders traduzidos recebem passagens de limpeza sobre o módulo concluído (código morto, dobra de constantes, reuso de valores, fatos de ramificação); a contagem de instruções de host do pacote cai cerca de 43 %, valendo cerca de 20 ms de tempo de GPU por frame na máquina de teste (`VICTORY_SHADER_OPT=0` as desliga).

A maioria das mudanças de velocidade mais recentes fica atrás de interruptores de ambiente que estão desligados por padrão (`VICTORY_LATE_WAIT`, `VICTORY_RELAY`, `VICTORY_WIDE_BACK`, `VICTORY_CARD_BUFFERS_MIN_MB`, `VICTORY_INPLACE`, `VICTORY_BVH_BUILDER`, ...). Cada um é descrito em um comentário onde é lido.

### Problemas conhecidos

- Tudo na sombra está preto (veja acima).
- O interior da primeira sala de jogabilidade às vezes não é desenhado depois que a cutscene entrega para a jogabilidade.
- Cerca de 24 GB de memória estão em uso durante a jogabilidade.
- **Crash da GPU (device removed) na fase de jogo**: após alguns segundos de execução, o dispositivo GPU é removido, encerrando o título. Ocorre especialmente com `VICTORY_RELAY` acima de 1 e com `VICTORY_LATE_WAIT` no estágio 2 ou 3; a causa está na thread de releio/leitura da memória da placa que lê enquanto a GPU ainda escreve. Investigação em andamento.
- Alguns linters de estilo são desligados para este fork no `Cargo.toml` do workspace até que o código mais novo seja arrumado.
- Muitas mudanças aqui são específicas para este título e esta GPU, e algumas são marcadas `shortcut:` no código onde aproximam o que o console faz.

Tudo abaixo desta linha é o próprio README do VIctory upstream, inalterado; sua seção de status descreve o upstream, não este fork.

---

# VIctory

Executa o título (Xbox Series X|S) nativamente no Windows, a partir de arquivos que o próprio usuário fornece.

O código x86-64 do título executa inalterado dentro do nosso processo; o VIctory fornece a plataforma por baixo dele e converte os dados do título, com antecedência onde pode e em primeiro uso onde deve (shaders). Veja [docs/architecture.md](docs/architecture.md).

## Status (2026-11-23)

**Ainda não é jogável.** O título inicia, executa seus menus a seus próprios 30 fps e carrega o mundo, onde a renderização falha.

| Estágio | Estado |
|-------|-------|
| Boot, telas de primeiro boot (acessibilidade, idioma, configurações, legal) | funciona, 30 fps |
| Vídeo de introdução com som | funciona |
| Tela inicial, perfil e salvamentos | funciona |
| Carregando para o mundo (cerca de 50 s) | funciona |
| O mundo (introdução da primeira missão) | alcançado e executa com draws indiretos; seu cartão de título renderiza, a cena depois dele ainda está escura; 0,5-1,8 s por frame |

Testado em uma máquina: Windows 11, AMD RX 6700 XT.

> **Aviso.** Entrar no mundo em uma GPU real pode atualmente travá-la, e o Windows então redefine o driver de exibição (todos os monitores ficam pretos por um momento). Para olhar mais a fundo, use uma compilação de diagnóstico com `VICTORY_D3D12_WARP=1`, que renderiza no adaptador de software.

O que cada parte faz hoje e o que ainda precisa:

| Subsistema | Hoje | Precisa |
|-----------|-------|-------|
| Carregador | Executa o executável do título e suas DLLs no nosso processo; relatórios de crash nomeiam o código culpado e são mantidos em `crashes.log` | – |
| Serviços de runtime | Um usuário offline conectado, licenças, pacotes, salvamentos que sobrevivem a um encerramento em qualquer etapa, caixa de diálogo de entrada de texto | Fala está em silêncio; o diálogo e o relançamento não foram testados ao vivo |
| Rede | Offline por design: o título não pode alcançar a rede | – |
| Arquivos e streaming | Arquivos e dados de streaming lidos, zlib e BCPack decodificados em paralelo | Carregamento é limitado por escritas na memória do título |
| Entrada | Controles XInput, teclado, teclado como substituto de controle | Vibração do gatilho |
| Áudio | Som de menu, vídeo e carregamento; vozes XMA e fluxos Opus decodificam | Reverb está em silêncio; resultados Opus não verificados contra o título |
| GPU: menus | Desenhos, compute e apresentação no host D3D12; apenas memória do título alterada é copiada | Aquecimento de quadro deriva em uma tela em compilações de lançamento |
| GPU: mundo | Pipelines de malha, buffers escritos por shader, tempos de vida de descritores, execução indireta (argumentos limitados), pipelines feitos por threads de trabalho e mantidos em disco | A maior parte da cena ainda está faltando (shaders não traduzidos deixam desenhos de lado); frames levam segundos; consultas reais, limites de profundidade, HDR e escalonamento estão faltando |
| Shaders | Código de GPU do console recompilado para DXIL no primeiro uso, cacheado no disco. Menus: todos. Mundo: malha 8.191 de 8.506, vértice/pixel 10.617 de 12.346, compute 1.643 de 2.539 | Atomicos de imagem de 64 bits, estágios de geometria mesclados, shaders do construtor de rastreamento de raios; alguns programas de compute precisam de uma GPU com ondas de 64 vias |
| Texturas | Layouts entalhados do console e desentalhamento para os modos que o título usa | Planos de metadados de alvo de renderização |
| Modo Series S | Executa pelos menus até a tela de carregamento | O mundo, como acima |

## Começando

Requisitos: Windows 10/11 x64, uma CPU com AVX2, uma GPU Direct3D 12 com shader model 6.6 (e shaders de malha para o mundo), Rust (fixado em `rust-toolchain.toml`).

O título também precisa de vários GiB de commit do Windows disponível na inicialização. Verifique **Commitado** em **Gerenciador de Tarefas → Desempenho → Memória**. Se `crashes.log` relatar o erro do Windows 1455, feche outros aplicativos ou aumente o tamanho do arquivo de paginação do Windows, então tente novamente.

```powershell
cargo xtask install-hooks                # verificação de política de ativo a cada commit
copy victory.example.toml victory.toml   # então defina seus caminhos (fora do repositório)
cargo test --workspace                   # não precisa de dados do jogo
cargo run -p victory -- config           # verifique seus caminhos
```

Execute o título:

```powershell
cargo build --release -p victory
target\release\victory.exe run <dir do jogo>\<exe do título> --execute
```

A legenda da janela mostra a taxa de quadros. Salvamentos, configurações e `crashes.log` são mantidos sob a pasta de dados (padrão `%LOCALAPPDATA%\victory\data`), dados derivados sob a pasta de cache.

Entrada: Controles XInput. Com nenhum conectado, o teclado é um controle enquanto a janela do título tem foco:

| Teclas | Controle |
|------|-----|
| W A S D / I J K L | controle esquerdo / direito |
| setas | direcional |
| Enter, Backspace, X, Y | A, B, X, Y |
| Q / E, Z / C | ombro, gatilhos |
| F / H | cliques do controle |
| Esc, Tab | Menu, Visão |

## Comandos

```
victory inspect <exe|dll> [--imports] [--exports]
victory census [paths...] [--missing] [--strict]
victory lib-exports <import.lib> [--list]
victory shader-info [paths...] [--verbose] [--disasm]
victory translate [paths...] [--validate] [--pipelines] [--list] [--jobs N]
victory run <exe> [--execute] [--unimplemented trap|zero] [--strict]
            [--console series-x|series-s] [-- <argumentos do título>]
victory config
```

Log: `VICTORY_LOG=victory_loader=trace,info` (uma compilação de lançamento registra info e acima).

## Depuração

Uma compilação de lançamento não tem ajudas de depuração. Construa-as com

```powershell
cargo build --profile diag -p victory --features diagnostics   # target\diag\victory.exe
```

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

## Licença

GPL-3.0-or-later. Veja [LICENSE](LICENSE).
