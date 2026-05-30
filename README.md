# weft-network-proto

gRPC contract of **weft-network**, the controller that reconciles
Routers, Load Balancers, DNS zones / records, and Scheduling Rules
from the agent's event stream into the data plane (Envoy +
WireGuard + VyOS/FRR + CoreDNS).

Sibling repo to [`weft-proto`](../weft-proto) — the two control
planes scale independently (agent = identity / compute / storage ;
network = traffic / naming / placement), so they own their own
proto surface. Same conventions (`task proto` to regenerate stubs,
backwards-compatible additions only, breaking changes need a new
major version).

## Service

| RPC family | Resources |
| ---------- | --------- |
| Routers          | List / Create / Delete (peer WG, egress VyOS / FRR) |
| Load Balancers   | List / Create / Delete / SetBackends (Envoy via xDS) |
| DNS Zones        | List / Create / Delete (CoreDNS authoritative + optional RFC-2136 push to a downstream BIND) |
| DNS Records      | List / Create / Delete (operator-managed `static`, controller-reconciled `auto`) |
| Scheduling Rules | List / Create / Delete (declarative placement constraints) |

## Deployment

weft-network runs as 3 infra microVMs (one per DC), etcd-elected
leader. The webui talks to whichever replica is reachable ; reads
are local snapshots, writes go to the leader (followers forward).
A separate socket (`--weft-network-socket`) so the agent and the
network controller stay independent at the wire boundary.
