# Documentation Conventions

This file records the evidence format used for the OverRide repository. It is
separate from the project README so that the README remains the owner's project
summary.

## Per-level artifacts

Each `levelXX` directory contains:

- `source`: a readable reconstruction of the original binary's relevant behavior.
- `walkthrough.md`: the commands, observations, and reasoning needed to reproduce the solution.
- `flag`: the password recovered for the next account.
- `Ressources`: text-only analysis records, command output excerpts, and supporting notes.

No challenge binary or downloaded ISO file belongs in the repository.

## Flag policy

Record the recovered next-level password in the current level's `flag` whenever
the challenge produces one. Leave `flag` empty only when no password can be
recovered or the result is otherwise inapplicable. In that case, the
`walkthrough.md` must state the reason.

For example, solving `level00` reveals `/home/users/level01/.pass`, so that
password is recorded in `level00/flag`.



# Evaluation Checklist

This repository follows the mandatory artifact layout required by the subject.
Each completed level contains only the following text artifacts:

| Level | Status | Result recorded in `flag` |
| --- | --- | --- |
| level00 | Mandatory | level01 password |
| level01 | Mandatory | level02 password |
| level02 | Mandatory | level03 password |
| level03 | Mandatory | level04 password |
| level04 | Mandatory | level05 password |
| level05 | Mandatory | level06 password |
| level06 | Mandatory | level07 password |
| level07 | Mandatory | level08 password |
| level08 | Mandatory | level09 password |
| level09 | Bonus | end password |

For every level, the evaluator can inspect:

- `source` for a readable reconstruction of the binary behavior relevant to the exploit.
- `walkthrough.md` for the observed reverse engineering, derived values, and verified demonstration.
- `flag` for the recovered next-account password, following the policy in `documentation_conventions.md`.
- `Ressources/analysis.md` for selected disassembly and runtime evidence.

No challenge binary, ISO image, core dump, or generated payload file is stored
in any level directory.