# Behavioral — Behavior Patterns

| Pattern | Structure | HFT | Description + Example |
|---------|-----------|-----|-----------------------|
| Observer | Subject → notify → listeners | not on hot path | Notify all subscribers when an event occurs. Callback list. Virtual call + dynamic dispatch = latency. Use on slow paths like config changes. |
| Strategy | Interface + different algorithms | with templates | Switch algorithms at runtime. Virtual version is slow. Template + policy-based design version is used in HFT. E.g. different order routing strategies. |
| Command | Encapsulate operation as object | sometimes | Undo/redo, put in queue, execute later. An Order can be thought of as a command object. |
| State | State objects + transitions | common | Object's behavior changes based on state. E.g. Order state machine — New→Acked→Filled→Cancelled. Implemented with enum + switch or state objects. |
| Template Method | Skeleton in base, subclass fills in | with CRTP | Define algorithm skeleton in base, let subclass implement steps. Done zero-cost with CRTP instead of virtual. |
| Chain of Responsibility | Handler chain | sometimes | Passes request to next handler if current can't process. Risk check pipeline is similar — chain of pre-trade checks. |
| Iterator | begin/end + operator++ | everywhere | Foundation of STL. Write iterators for your own containers — range-for works. |
| Visitor | Double dispatch | with std::visit | `std::variant` + `std::visit` combination. Your `MessageTypes_t` variant and dispatch is exactly this pattern. Works without virtual. |
