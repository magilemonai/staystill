# LOOPS

<!-- recursive self-improvement loops — scouted by magi lemon command, 2026-07-02 -->

- Loss telemetry → pacing: log loss raceTime+phase in staystill_memory at endGame so one Claude command can retune T/AI_BREAK_TIMES/moveThreshold. first step: add lastLossT+lastLossPhase to the saveMemory patch.
- Shipped jokes → more jokes: feed NOTIFICATION_POOL + NARRATOR_LINES to Claude as voice samples to draft new in-schema entries. first step: a Claude command appending N NOTIFICATION_POOL rows in {t,type,from,body} shape.
- Play data → NEXT.md: after each session a Claude step reads exported staystill_memory + git log and rewrites the NEXT.md bullets magi reads. first step: add a dev console dump that exports the memory aggregate as JSON.
- Self-hardening harness: turn the jsdom harness that drove all endings (caught the pushNotification loop) into a check asserting all four endGame outcomes via ?ts. first step: commit it covering finale/2nd/monument/loss.
