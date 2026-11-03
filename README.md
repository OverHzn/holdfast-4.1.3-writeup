# HoldFast Wallet 4.1.3: unproven confirmation heights are no longer trusted at all

**By oxhulk**

**Disclosure status:** cleared for publication from 2026-11-03 by Around The Block
security. This write-up describes only what the wallet now guarantees. It
intentionally contains no reproduction steps, no proof-of-concept code, and no
detail that could be used to exploit any version.

## Summary

HoldFast Wallet 4.1.3 closes the remaining case of a locked-coins issue in the
wallet's handling of server-reported confirmation data. The fix is a removal of
trust, not a new check layered on top of the old one: a confirmation height
reported by a server is no longer taken at all unless the wallet can prove it
from headers it already holds.

## What the wallet now guarantees (4.1.3)

1. **No unproven height is used.** If a reported confirmation height cannot be
   proven against headers the wallet itself holds, the wallet simply does not
   use it — in any path, not just the common one.
2. **An unproven coin is never treated as confirmed.** Coins whose confirmation
   rests only on an unproven server claim do not gain confirmed status.
3. **An unproven coin is never selected beside real coins.** A spend cannot be
   built that mixes a real, confirmed coin with a coin whose confirmation was
   never proven.
4. **Real coins cannot be frozen this way.** Because of (2) and (3), a user's
   genuine coins can no longer end up marked as spent by a transaction that
   rests on an unproven coin and can never confirm.

## How this differs from 4.1.2

Version 4.1.2 narrowed the issue substantially: in most cases a server-reported
height was treated only as a hint, so an unproven coin stayed unconfirmed and
unselectable. But a narrower case remained in which the old behavior could
still surface. Version 4.1.3 removes the underlying trust entirely, so there is
no remaining case to narrow — the wallet no longer accepts an unprovable
server height in the first place.

## Historical impact, in general terms

Before this fix, in that narrow remaining case, a user's real coins could be
left marked as spent until the wallet was rebuilt from the chain. This was a
funds-locking issue, not a funds-moving one: it did not move anyone's coins to
an attacker. The triager assessment for the issue was Low to Medium, reflecting
that it required a reconcile server to report false data.

## Timeline

- Report submitted to Around The Block security, with a proof of concept
  reproduced against the wallet's real core.
- Triager reproduced the locked-coins consequence in 4.1.2 and confirmed the
  narrower remaining case.
- Fix decided: refuse any server-reported height no header held by the wallet
  can prove. The reporter's test became the regression test for the fix.
- **HoldFast Wallet 4.1.3 released** (Google Play and signed APK on GitHub,
  aroundtheblock-wallet-releases, tag v4.1.3), closing the remaining case.
- Write-up cleared for publication from 2026-11-03.

## Credit

Reported by **oxhulk**. Thanks to Around The Block security (Mark McNiel) for
the reproduction, the fix, and the disclosure clearance.

---
*Note: no exploit detail is included by design, per the disclosure terms
agreed with the vendor. Nothing in this write-up is sufficient to reproduce
the issue against any version.*
