# Controller FSM Requirements

Owner: **Member 04 - Cache Controller FSM and Main-Memory Model**

This hand-off records the behavior required by the frozen interface. `rtl/cache_controller.v` satisfies it. The delivered states are `IDLE`, `LOOKUP`, `MEM_WAIT`, and `RESPOND`. Refill is the `mem_ready` cycle inside `MEM_WAIT`, not a separate state. The diagram, transition table, and timing are in `docs/fsm_design.md`.

## Required behavior from the shared interface

The controller must support these externally visible phases:

1. Wait until `req_valid && req_ready` accepts one request.
2. Capture tag, index, and offset as stable `active_*` fields.
3. Observe `lookup_hit` after the selected cache line is available.
4. On a hit, return `cache_byte_data` without asserting `mem_req`.
5. On a miss, assert `mem_req` and wait for `mem_ready` while keeping the active address stable.
6. When memory responds, pulse `refill_write_en` and return `refill_byte_data`.
7. Pulse `resp_valid` for one cycle and become ready for the next request.

The delivered state names are `IDLE`, `LOOKUP`, `MEM_WAIT`, and `RESPOND`. The implementation follows `docs/interface_contract.md`.

## Acceptance points

- Reset returns the controller to its idle state.
- One request is accepted only when `req_ready` is high.
- A hit never starts a memory request.
- A miss never writes the cache before `mem_ready`.
- Refill enable is a one-cycle pulse.
- Exactly one response pulse is produced per accepted request.
- Active request fields remain stable until completion.
