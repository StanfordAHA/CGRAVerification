# The Formal Verification Status for the Global Controller
We are using the global controller with the JTAG stripped out, from the CGRAGenerator branch [gc\_verif](https://github.com/StanfordAHA/CGRAGenerator/tree/gc\_verif)

## Legend
Entries will be in the following format:

* *Property name*: \<Intuitive Description\>
   * SystemVerilog Assertion
   * **Status** | Previous Status -> Current Status
      * [Optional] The commit which fixed the bug
   * [Optional] Additional information

Status can be:

* **PROVEN** -- Found a full proof that the property holds
* **BOUNDED** <k> -- No trace can violate the property in <k> cycles or less.
* **FAILED** -- Found a counterexample

Properties marked (LIVE) are liveness properties, i.e. from any reachable state "something good eventually happens". The solver looks for a loop in the system state space where the property is constantly violated. In other words, it finds an infinite execution where the "good" thing never occurs. These properties can often be difficult, so if this doesn't reach a full proof, we'll include the bound it checked until.

## Global Controller Status

### [Completed] In Isolation

We started by checking properties of the global controller in isolation.

#### Environmental assumptions

These are assumptions used to aid the proof goals.

* *delay\_small*: The global controller allows the user to set the delay associated with certain ops. It helps some of the proofs go through if this delay is not too large. This assumption restricts the user-defined delays to be less than 50 cycles.
   * `assume property ((((op == wr\_rd\_delay\_reg) || (op == global\_reset) || (op == advance\_clk)) |-> (config\_data\_jtag\_out < 50)))`

#### Properties

* *finally\_ready (LIVE)*: From any state, the global controller will eventually be ready again
   * `assert property ((state != ready) |-> ##[1:$] (state == ready))`
   * **BOUNDED** 800

* *no\_underflow*: The counter which is used to return to the ready state never underflows. The counter is 32 bits wide so this would tie up the global controller for a long time.
   * `assert property ((counter == 0) |=> (counter != -1))`
   * **FAILED** -> **PROVEN**
      * Fixed in [this commit](https://github.com/StanfordAHA/CGRAGenerator/commit/56aa3f118be0c5b99467f0236ccdc6a349da7508)

* *counter\_works*: If `state` is not `ready`, the counter should always be counting down (making progress towards returning to the `ready` state).
   * `assert property (@(posedge clk) (state != ready) |=> ((counter == $past(counter) - 1) || ($past(counter) == 0)))`
   * **PROVEN**

* *rd\_stop*: If the global\_controller is not in the `reading` `state`, then `read` should not be high.
   * `assert property ((state != reading) |-> !read)`
   * **PROVEN**

* *wr\_stop\_strong*: The `write` signal can't be stuck high when the global controller is not writing.
   * `assert property (@(posedge clk_in) (($past(op) == write\_config) && (op != write\_config)) |=> !write)`
   * **FAILED**
   * This is not a game-stopper. After discussing with @alexcarsello, this situation is highly unlikely and probably would not even cause a problem because it relies on switching the clock domain. This temporarily stops the clock used when processing ops, so the `write` signal can get stuck high. This also stops the clock to the rest of the CGRA, and once the clock starts again, `write` is immediately de-asserted. Even though this is probably not a big deal, @alexcarsello said he will fix it in a future commit.

* *wr\_stop\_weak*: This is similar to wr\_stop\_strong, except the extra possibility that the clk is stopped is incorporated.
   * `assert property (@(posedge clk_in) (($past(op) == write\_config) && (op != write\_config)) |=> (!clk\_out || !write))`
   * **PROVEN**

* *correct\_config*: This property simply checks that there's no way to corrupt the configuration address or data when writing the configuration.
   * `assert property (@(posedge clk) ((state == ready) && (op == write\_config)) |=> ((config\_addr\_out == $past(config\_addr\_jtag\_out)) && (config\_data\_out == $past(config\_data\_jtag\_out))))`
   * **PROVEN**

* *clk\_live (LIVE)*: Asserts that the global\_controller `clk` cannot be stopped for too long
   * `assert property ((clk == 1)[->1:110])`
   * **PROVEN**
   * The number of cycles bounding the amount of time it can be stopped is dependent on the delay\_small assumption. Because the user can set the delay to (almost) any 32-bit value.

### [In Progress] Connected to Fabric

We are currently working on getting the global controller from the [gc\_verif](https://github.com/StanfordAHA/CGRAGenerator/tree/gc\_verif) branch along with the rest of the CGRA into [CoSA](https://github.com/cristian-mattarei/CoSA) properly.

For scaling reasons, we are starting with a 4x4 fabric instead of a 16x16.

#### Properties

* *config_check_pe*: Check that writing a configuration to any PE tile (in the 4x4), then reading that same configuration address matches up.
   * `assert property ((((global_controller.op == global_controller.write_config) && (global_controller.state == global_controller.ready)) ##2 (((global_controller.op == global_controller.read_config) && (global_controller.state == global_controller.ready)) && (config_addr_in == $past(config_addr_in, 2)))) |-> ##6 config_read |-> (config_data_jtag_in == $past(config_data_in, 8)));`
   * **PROVEN**
   * This proof relies on the following assumptions:
      * rd_delay_2: The read delay register is always equal to 2 (i.e. not letting user set a different read delay value).
      * clk_active: Not doing the operations when the CGRA is stalled or switching clock domains