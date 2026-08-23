#### Verilog and SystemVerilog Review Principles
> Favor precision over recall: report only defects likely to change synthesized hardware behavior, cause simulation/synthesis mismatch, or introduce timing hazards in the changed RTL. Account for whether the file is synthesizable design code or a simulation-only model before raising findings, and do not report style that a linter or formatter already handles. This rule covers both Verilog (`.v`) and SystemVerilog (`.sv`).

#### Blocking and Non-Blocking Assignments
- Non-blocking assignments (`<=`) used for combinational logic, or blocking assignments (`=`) used for sequential logic inside a clocked `always`/`always_ff` block, where the mixed style changes simulated ordering or infers unintended hardware
- Blocking and non-blocking assignments to the same signal within one `always` block, producing order-dependent, non-deterministic behavior
- A signal assigned from more than one `always` block, creating multiple drivers on synthesis and a race in simulation
- Prefer `always_ff`, `always_comb`, and `always_latch` (SystemVerilog) so the tool checks the intended inference instead of a bare `always @(*)`

#### Inferred Latches
- A combinational `always`/`always_comb` block where a signal is not assigned on every path (missing `else`, an incomplete `case`, or a `case` without `default`), inferring an unintended transparent latch
- Missing default assignments at the top of a combinational block, so a newly added branch silently reintroduces a latch
- Outputs left unassigned for some input combinations of a decoder, mux, or FSM next-state logic

#### Signal Width and Signedness
- Assignments or comparisons between operands of different bit widths that truncate or zero-extend silently, dropping significant bits or changing a comparison result
- Arithmetic that overflows the declared width of the result, or an intermediate expression narrowed before it is widened
- Mixed signed/unsigned operands where Verilog's context-determined signedness makes a comparison or shift behave unexpectedly; be explicit with `$signed`/`$unsigned`
- Part-selects, concatenations, or replication counts whose width does not match the target, and reliance on implicit `reg`/`wire` width from an undeclared net

#### Clock and Reset Handling
- Reset that is not correctly synchronous or asynchronous as intended, a reset polarity mismatch, or a reset released asynchronously without a synchronizing deassertion (reset-recovery hazard)
- Sequential logic missing the reset in the sensitivity list for an asynchronous reset, or a value that must survive reset being placed on the reset branch
- Gated, derived, or combinationally generated clocks used where a clock enable is intended, and multiple clocks driving the same register
- Registers with no reset where the design assumes a known power-on state

#### Clock-Domain Crossings and Races
- A signal sampled in one clock domain that is driven from another without a synchronizer (two-flop for single-bit control, handshake or asynchronous FIFO for buses), risking metastability
- Multi-bit buses synchronized bit-by-bit, so bits arrive skewed and produce transient invalid values; use gray coding or a handshake
- Combinational feedback loops, or read-during-write races on inferred memory without a defined collision policy

#### Simulation vs. Synthesis and Unsafe Constructs
- `initial` blocks, `#delay` timing, `fork`/`join`, or `force`/`release` used in code intended for synthesis, where the tool ignores them and simulation disagrees with hardware
- Incomplete or overlapping sensitivity lists in a bare `always @(...)` that make simulation differ from the synthesized combinational function; prefer `@(*)` or `always_comb`
- `casex`/`casez` whose don't-care matching hides priority bugs, or a `case` relying on `x`/`z` matching; prefer `case` with `unique`/`priority` where the intent is exclusive or prioritized
- `$display`/`$finish`/assertions guarding real behavior, and non-synthesizable system tasks left in the design path
- Full-case/parallel-case pragmas that assert properties the logic does not actually guarantee
