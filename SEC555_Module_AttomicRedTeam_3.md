# SEC555 — Atomic Red Team (module notes)

## Objective

Document how **Atomic Red Team** (or equivalent simulation libraries) was used in the lab to generate **telemetry** on endpoints, then correlate that activity in **SIEM** or EDR consoles.

## Investigation workflow

1. **Select atomics** that map to the technique under study (e.g. credential access, lateral movement).
2. **Execute** on the designated lab host only, with explicit approval and rollback plan.
3. **Collect evidence:** Windows **Event Log**, **Sysmon**, EDR alerts, and any **network** captures.
4. **Tune detection:** translate noisy rules into high-fidelity logic (parent/child process, command-line, user context).

## QC / placeholders

- Record **exact atomic IDs**, hostnames, and time range for each run so analysts can pivot logs without ambiguity.
- If this file was meant to hold lab-specific output, paste **sanitized** command transcripts and alert screenshots references here.
