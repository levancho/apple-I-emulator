# Apple I Terminal

> Woz's garage board, minus the garage. A real 6502 CPU emulator running behind a period-accurate hex monitor, wearing the wood-cased unit's face.

No OS. No file system. No undo. Boots straight to a blinking `\` and waits for you to type hex like it's July 1976.

## What's actually running

This isn't a video of a terminal or a scripted animation — it's a real (if simplified) MOS 6502 instruction set implemented in JavaScript, wired to a memory-mapped 6820 PIA, driving a 40×24 uppercase-only CRT. Anything you deposit and run executes as genuine machine code.

| Spec | Value |
|---|---|
| CPU | MOS 6502 @ 1 MHz (emulated, official opcodes) |
| Memory | 8 KB RAM, `$0000`–`$1FFF` |
| I/O | 6820 PIA at `$D010`–`$D013` (KBD / KBDCR / DSP / DSPCR) |
| Display | 40 cols × 24 rows, uppercase only, green phosphor |
| Firmware | ~ replaces the 256-byte WOZ Monitor ROM — reimplemented in JS, not a ROM dump |
| Stack | `$0100`–`$01FF`, standard 6502 |
| Launch price (1976) | $666.66 |

## Using the monitor

Same syntax family as the original WOZ Monitor:

```text
0300              examine one byte at $0300
                  (press Return again to walk to the next address)

0300.030F         examine a range, 8 bytes per line

0300: A9 00 8D 12 D0 00
                  deposit bytes sequentially starting at $0300

0300R             run the program starting at $0300
```

Worked example — hand-assemble a loop that prints one character:

```text
\0300: A9 41 8D 12 D0 00
\0300R
A
```

That's `LDA #$41` (load 'A'), `STA $D012` (write it to the display register), `BRK` (hand control back to the monitor).

## Running it locally

It's a single static HTML file — no build step, no bundler, no `node_modules`.

```bash
# clone
git clone https://github.com/levancho/apple-I-emulator.git
cd apple-I-emulator

# serve it any way you like
python3 -m http.server 8080
# or
npx serve .
```

Open `http://localhost:8080` and start typing.

## Deploying (Cloudflare Pages)

Follows the same Wrangler pattern as the rest of the `l3v.ai` sites (deployed independently of this GitHub repo — Cloudflare Pages project name doesn't need to match the repo name):

```bash
npx wrangler pages deploy . --project-name=l3v-apple1-site
```

Then attach the custom domain (`apple1.l3v.ai`) to the Pages project from the Cloudflare dashboard, or:

```bash
npx wrangler pages domain add apple1.l3v.ai --project-name=l3v-apple1-site
```

## Known limitations

- Official 6502 opcodes only — no illegal/undocumented instructions.
- No cycle-accurate timing; runs in fixed-size instruction batches per tick.
- BCD (decimal mode) is tracked as a flag but doesn't change `ADC`/`SBC` behavior.
- 8 KB of RAM, not the original's 4 KB — more headroom for demo programs, at the cost of strict accuracy.

## Credits

Built as a love letter to the machine that started it all. Steve Wozniak designed the original in 1976; this is a from-scratch reimplementation, not a copy of Apple's ROM.
