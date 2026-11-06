# Postmortem: Black October (PHP2Shell) lab

Blameless review of the full loop: Identify, Protect, Detect, Respond, Recover.

## What went well
- Detection 5, written in September, caught a web server opening a shell without being written for PHP2Shell. It watches behaviour, not one specific bug.
- During the response, php-old was disconnected from the network before it was deleted. That cut off any attacker while keeping the evidence for docker diff, docker top and the logs.

## What went wrong
- Detection 5 missed a web server downloading tools with curl (false negative). It only knew about shells and miners.
- Detection 5 labelled every alert as "Reverse shell (T1059)", including a curl download and a plain command like id. A wrong label sends the next analyst looking for the wrong thing.

## Changes
| Problem | Fix | Status |
|---|---|---|
| curl false negative | Added curl and wget to the detector, plus a regression test | Done, 3 Nov |
| No habit of testing detections against new attacks | When a major new attack appears, test existing detections against a simulated log of its post-exploitation activity before trusting them | Adopted |
| Wrong labels | Label downloads as T1105 (Ingress Tool Transfer), reverse shells as T1059, other commands as command execution (T1059) | Done, 4 Nov |

## Still open
- Intel AMT: check ME firmware version and whether AMT is enabled, in the BIOS.
