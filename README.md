# Verilog / SystemVerilog Cheatsheet

A dense, **one-stop quick-reference** for Verilog and SystemVerilog — the syntax, idioms, and
gotchas you look up mid-design, in one page. Written for people who already know the language and
just need a fast reminder (and a list of the traps that cause real bugs).

## Data types
| Type | Notes |
|------|-------|
| `wire` | net; needs a continuous driver; multiple drivers allowed (buses) |
| `reg` | holds a value in a procedural block; **not** necessarily a hardware register |
| `logic` (SV) | 4-state; single driver OR procedural; prefer this over `reg`/`wire` |
| `bit` (SV) | 2-state (0/1); faster sim, no X/Z |
| `integer` / `int` | 32-bit signed |
| `[N-1:0]` | packed vector (bus) |

## Procedural blocks
```verilog
always @(posedge clk)           // sequential — use non-blocking <=
always @(*)                     // combinational — use blocking =, assign every path
always_ff @(posedge clk)        // SV: tool checks it's really sequential
always_comb                     // SV: tool checks it's really combinational
```

## Blocking vs non-blocking — the rule
- **Sequential** logic → non-blocking `<=`
- **Combinational** logic → blocking `=`
Mixing them causes sim/synth mismatch. (Deep dive in
[sta-timing-closure-notes](https://github.com/chipxpertofficial/sta-timing-closure-notes) and
[vlsi-interview-prep](https://github.com/chipxpertofficial/vlsi-interview-prep).)

## Numbers & literals
```verilog
4'b1010    // 4-bit binary
8'hFF      // 8-bit hex
16'd255    // 16-bit decimal
4'sd-2     // signed decimal
'0 / '1 / 'x   // SV: fill all bits
```

## Common idioms
```verilog
// synchronous reset
always_ff @(posedge clk) if (!rst_n) q <= '0; else q <= d;
// async reset
always_ff @(posedge clk or negedge rst_n) if (!rst_n) q <= '0; else q <= d;
// register with enable
always_ff @(posedge clk) if (en) q <= d;
// width-safe counter
logic [$clog2(DEPTH)-1:0] cnt;
// one-hot case
unique case (1'b1) sel[0]: ...; sel[1]: ...; endcase
```

## Parameterisation
```verilog
module fifo #(parameter W = 8, DEPTH = 16) (...);
localparam AW = $clog2(DEPTH);
```

## The gotchas that cause real bugs
- **Inferred latch** — a combinational `always` that doesn't assign every output on every path.
  Set defaults at the top.
- **Blocking in sequential logic** — sim/synth mismatch, races.
- **Width mismatch** — silent truncation or zero/sign-extension; size your literals.
- **Incomplete sensitivity list** (old Verilog) — use `always @(*)` / `always_comb`.
- **`==` with X/Z** — use `===`/`!==` in testbenches to compare against X/Z.
- **Multiple drivers on a `logic`** — SV flags it; in plain Verilog a `reg` driven twice is a bug.
- **Non-constant part-select** — use indexed `+:` / `-:` (`data[i*8 +: 8]`).

## SystemVerilog niceties worth using
- `enum` for FSM states (readable waveforms).
- `struct packed` for grouping bus fields.
- `interface` + `modport` to bundle a bus and its direction.
- `always_comb` / `always_ff` / `always_latch` — intent-checked by the tool.

## License
MIT — see [LICENSE](LICENSE). Maintained by the ChipXpert team.
