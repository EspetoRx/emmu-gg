# emmu-gg

Core de **Sega Game Gear** usado pelo Emmu no navegador. No desktop, o mesmo
console é executado pelo MesenCE através de `emmu-mesen-sys`; este crate não é
carregado pela aplicação Tauri.

## O que está incluído

- Wrapper Rust/WASM com a API esperada pelo worker da interface:
  `new(rom)`, `gameTitle()`, `runFrame(renderGraphics)`, `audioSamples()`,
  `audioSampleRate()`, `audioChannels()`, `setButton()`, `setVolume()`,
  `restart()`, `saveState()` e `loadState()`.
- Frame RGBA8888 nativo de **160×144**, igual ao LCD do Game Gear.
- Áudio estereofônico a **48 kHz**; o volume é aplicado antes de devolver as
  amostras ao `AudioWorklet`.
- Save states com magic `EMMUGG01` seguido do estado serializado pelo
  `smsgg-core`. Estados com outro magic são rejeitados.
- SRAM de bateria mantida em memória e preservada no `restart()`.

## Dependência de emulação

Todo o código de emulação fica dentro desta pasta, em `vendor/jgenesis`:

- `backend/smsgg-core` — Z80, VDP, mapeadores e núcleo SMS/GG;
- `backend/ti-sn76489` — PSG;
- `cpu/z80-emu`, `common/*`, `config/smsgg-config` e dependências auxiliares.

O upstream é [`jsgroth/jgenesis`](https://github.com/jsgroth/jgenesis), commit
`8d1bdfba460b3a49b493aa00368926ea375d6074`, GPL-3.0-only. Detalhes e
modificações locais estão em `UPSTREAM.txt`; a licença está em `LICENSE` e em
`vendor/jgenesis/LICENSE`.

Os nomes dos pacotes vendorizados foram sufixados com `-gg` (`dsp-gg`,
`jgenesis-common-gg`, `jgenesis-proc-macros-gg`) para não colidirem com o subset
JGenesis usado pelo `emmu-pce`.

## Configuração

`default_config()` força:

- `SmsGgHardware::GameGear` (sem BIOS);
- resolução LCD de 160×144 (`gg_use_sms_resolution = false`);
- aspecto `GgLcd` e blending de frame desligado;
- PSG SN76489 ativo e sem FM (o Game Gear não tem o slot de expansão do SMS);
- região detectada automaticamente pelo cartucho.

## Controles

A UI mapeia os botões do teclado/toque para os controles do hardware:

| Ação do Emmu | Botão Game Gear |
| --- | --- |
| Direções | D-pad |
| `A` / `B` | Botão `1` / `2` |
| `Start` | `Start` |
| `Select`, `X`, `Y`, `L`, `R` | ignorados (não existem no hardware) |

P2 também é escondido na interface para GG.

## Build

O WASM é gerado pelo script raiz da interface:

```sh
cd platforms/web/ui
npm run build:wasm
```

Saída: `platforms/web/ui/src/wasm/gg/emmu_gg.js`.

## Testes

```sh
cargo test -p emmu-gg
```

Com uma ROM de jogo válida (não BIOS):

```sh
EMMU_GG_TEST_ROM="/caminho/jogo.gg" cargo test -p emmu-gg -- --nocapture
```

O teste com ROM é ignorado quando a variável de ambiente não está definida.
Os testes sem ROM usam uma imagem mínima apenas para validar frame, áudio,
save state, botões e restart.
