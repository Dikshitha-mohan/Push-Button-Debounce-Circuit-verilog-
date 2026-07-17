# Push-Button Debounce Circuit (Verilog)

A synthesizable RTL design that removes mechanical contact bounce from a push-button input, with a self-checking testbench simulating realistic bouncy presses.

## Overview

Mechanical push buttons don't produce a clean 0→1 or 1→0 transition when pressed — the contacts physically bounce for a few milliseconds, producing multiple rapid glitches. If fed directly into digital logic, this can be misread as several presses instead of one. This project implements a standard **2-flip-flop synchronizer + counter-based debounce filter** in Verilog, along with a **single-cycle pulse generator** for clean edge-triggered actions.

## Features

- **Metastability protection** — 2-FF synchronizer brings the asynchronous button input into the clock domain safely.
- **Configurable debounce window** — set via the `DEBOUNCE_LIMIT` parameter (number of stable clock cycles required before accepting a new level).
- **Clean level output** (`btn_clean`) — stable, glitch-free representation of button state.
- **Single-cycle pulse output** (`btn_pulse`) — fires exactly once per press, useful for triggering counters, toggles, or state machine transitions.
- **Self-checking testbench** — uses `$random` to simulate bouncy presses/releases and automatically verifies:
  - Clean presses produce the correct output level
  - Bouncy presses still produce exactly one pulse
  - Short glitches (shorter than the debounce window) are correctly rejected

## Files

| File | Description |
|---|---|
| `spi_master.v` *(rename as `debounce.v`)* | RTL design — synchronizer, debounce counter, edge detector |
| `tb_debounce.v` | Self-checking testbench with bouncy press/release simulation |

## Module Ports

**`debounce`**

| Port | Direction | Width | Description |
|---|---|---|---|
| `clk` | input | 1 | System clock |
| `rst_n` | input | 1 | Active-low async reset |
| `btn_in` | input | 1 | Raw, noisy button input |
| `btn_clean` | output | 1 | Debounced, stable level |
| `btn_pulse` | output | 1 | Single-cycle pulse on clean press |

**Parameter:** `DEBOUNCE_LIMIT` — number of consecutive stable clock cycles required before accepting a new input level. In simulation this is set low (10) for fast test runs; for real hardware at ~50 MHz, a typical value is ~500,000 (≈10 ms debounce time).

## How It Works

1. **Synchronization** — the raw `btn_in` signal passes through a 2-stage flip-flop chain to eliminate metastability risk from the asynchronous input.
2. **Debounce filtering** — a counter increments only while the synchronized input differs from the current `btn_clean` value. If the input holds steady for `DEBOUNCE_LIMIT` cycles, `btn_clean` updates to match it. Any glitch back to the old value resets the counter.
3. **Edge detection** — `btn_clean` is compared against its previous cycle's value; a rising edge (0→1) produces a single-cycle `btn_pulse`.

## Simulation

Tested on **EDA Playground** using **Icarus Verilog 12.0**.

1. Place the design file in the **Design** pane and the testbench in the **Testbench** pane.
2. Select **Icarus Verilog 12.0** as the simulator.
3. Run the simulation — check the console for `PASS`/`FAIL` results across 4 test scenarios:
   - Clean single press
   - Bouncy press (15 random glitches)
   - Bouncy release + second bouncy press
   - Short glitch rejection
4. Open **EPWave** → click **Get Signals** → select `clk`, `rst_n`, `btn_in` (top-level scope) and `btn_clean`, `btn_pulse` (`.dut` scope) to visualize the debouncing in action.

## Example Console Output

```
---- Test 1: Clean press ----
PASS: btn_clean went high after clean press
---- Test 2: Bouncy press ----
PASS: exactly 1 pulse generated despite bounce (count=1)
---- Test 3: Bouncy release + second press ----
PASS: btn_clean settled low after release
PASS: exactly 1 pulse on second bouncy press (count=1)
---- Test 4: Short glitch rejection ----
PASS: short glitch correctly ignored
ALL TESTS PASSED
```

## Possible Extensions

- Parameterize for active-low buttons
- Add a falling-edge pulse (`btn_release_pulse`)
- Instantiate multiple debounce channels for a keypad
- Port to an FPGA dev board (Basys3/Nexys) with a real push button for hardware validation

## Author

Dikshitha M
