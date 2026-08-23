#### VHDL Review Principles
> Favor precision over recall: report only defects likely to change synthesized hardware behavior, cause simulation/synthesis mismatch, or introduce timing hazards in the changed RTL. Account for whether the file is synthesizable design code or a simulation-only model before raising findings, and do not report style that a linter or formatter already handles.

#### numeric_std and Type Usage
- Arithmetic performed on `std_logic_vector` via the deprecated `std_logic_arith`/`std_logic_unsigned` packages instead of `numeric_std` with `signed`/`unsigned`, giving nonportable or ambiguous results
- Conversions between `integer`, `signed`, `unsigned`, and `std_logic_vector` done implicitly or with the wrong function (`to_integer`, `to_unsigned`, `to_signed`, `resize`), truncating or mis-signing values
- Mixing signed and unsigned operands, or comparing values of different widths, where `numeric_std` silently resizes or the result is metavalue-dependent
- Reliance on `'U'`, `'X'`, or `'Z'` propagation in arithmetic, so simulation produces metavalues that synthesis cannot reproduce

#### Ranges, Indexing, and Resize
- Vector indexed or sliced outside its declared range, or with the wrong direction (`to` vs. `downto`), causing a runtime error in simulation and wrong bits in hardware
- `resize` that unintentionally truncates the high bits of a `signed`/`unsigned`, or zero/sign extension that changes the represented value
- Off-by-one range arithmetic in `(N-1 downto 0)` declarations and loop bounds, and `others => '0'` aggregates applied to a target of unexpected width

#### Inferred Latches and Process Sensitivity
- A combinational process whose sensitivity list omits signals it reads, so simulation and synthesis disagree; prefer `process(all)` (VHDL-2008) where available
- A combinational process that does not assign every output on every path (missing `else`, incomplete `case`/`when`, no default assignment), inferring an unintended latch
- A `case` without `when others`, or a selected assignment that does not cover all input values, leaving outputs undefined for some cases

#### Clock and Reset Handling
- A clocked process using a condition other than `rising_edge(clk)`/`falling_edge(clk)` (e.g. `clk'event and clk = '1'` combined with data in the sensitivity list), blurring the intended synchronous behavior
- Reset that is not synchronous or asynchronous as intended, a reset polarity mismatch, or an asynchronous reset released without synchronized deassertion (recovery/removal hazard)
- Signals that must retain state across reset placed on the reset branch, or registers with no defined reset where a known startup state is assumed
- Gated or derived clocks where a clock enable is intended, and more than one clock driving the same register

#### Clock-Domain Crossings and Resolved Signals
- A signal generated in one clock domain and sampled in another without a synchronizer (two-flop for single-bit, handshake or asynchronous FIFO for buses), risking metastability
- Multi-bit buses crossing domains without gray coding or a handshake, so bits arrive skewed
- Multiple drivers on a resolved signal (`std_logic`) relied on for wired logic, where the resolution function hides an unintended multi-driver conflict; unintended shared drivers on what should be a point-to-point signal

#### Simulation vs. Synthesis and Unsafe Constructs
- `wait for`, `after` delays, initial signal values, or `assert`/`report` used to drive behavior in code intended for synthesis, where the tool ignores them and hardware disagrees with simulation
- Variables (`:=`) inside a clocked process used where their immediate-update semantics differ from signal (`<=`) semantics and change the inferred register or its ordering
- Reads of a signal in the same process cycle expecting the updated value, ignoring VHDL's postponed signal update (delta-cycle) semantics
- Nonsynthesizable constructs (`access` types, files, unbounded loops, dynamic memory) left in the design path, and metavalue-dependent branches that behave differently in gate-level simulation
