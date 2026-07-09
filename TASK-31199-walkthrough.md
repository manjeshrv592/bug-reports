# TASK-31199 — Step-by-Step Walkthrough (Study Notes)

> Companion notes to `TASK-31199.html`. The report is the formal record; this file explains
> the mechanism in plain steps, with the queries to see each piece yourself.

**Bug in one line:** Singapore cheque requests were saved with the fake Purchase Entry No
`ParameterErr` because the branch's number series was never configured — and since every
request shared that same fake value, any single rejection erased the whole branch's
pending queue.

---

## Who is who

| Role | What they do | In this ticket |
|---|---|---|
| **Ops (operations)** | Handle job files, enter vendor costs, raise **Cheque Requests** to get vendors paid | Users 2496 (job 2602711), 3946 (07-Jul re-send) — the complainants |
| **Finance** | See raised requests on **Purchase Posting Approval**, approve or **reject** | The rejections that (unknowingly) wiped the queue |
| **QA** | Reproduced and verified | User 3943 (the `Test123` rows, the 05:41 verification send) |

The flow is a handoff: **ops requests payment → finance approves/rejects**. The bug sat
exactly on that handoff.

---

## The broken flow, step by step (before the fix)

**Step 1 — Ops clicks Send.**
The browser posts the selected invoices to the API with the login branch: `Branch = 'S'`.

**Step 2 — The save procedure starts.**
`ff.Update_2ChequeRequest` receives the data and, for each vendor invoice, asks the number
generator for a purchase entry number:

```sql
exec appConfig.Get_Number 'S', '', 'VINV', <today>, '', @N output
```

**Step 3 — The generator translates the branch code.**
Inside `appConfig.Get_Number`, before anything else:

```sql
if @Branch='S' set @Branch='SG'
```

From this point it is searching for **SG**. (Same pattern for other branches:
W→ANT, R→RTM, E→UKM, Y→NY.)

**Step 4 — The lookup fails.**
It searches the series table:

```sql
select ... from appConfig.Numbers where BranchId='SG' and NType='VINV' and isActive=1
```

Zero rows (until the fix). Instead of raising an error, the procedure quietly returns the
fallback text `'Parameter Err'` → spaces stripped → **`'ParameterErr'`**.

**Step 5 — The save procedure doesn't check what came back.**
It takes `@N` — whatever it contains — and writes it everywhere a real number would go:

- `ff.ICost.PurEntryNo = 'ParameterErr'` (stamped on the cost rows, with `ChqReqSent = now`)
- `Bss_Report.ff.VendorPayment` — audit row with `PurEntryNo = 'ParameterErr'`
- `Bss_Report.ff.Cheque_Request_List` — audit row with `PurEntryNo = 'ParameterErr'`
- The notification mail body — the entry-number column shows `ParameterErr`

No error, no warning. The screen says "Updated Successfully". This is why the bug ran for
**three years** unnoticed — nothing ever failed visibly.

**Step 6 — The collision.**
A real number (`VINVSG260001`, `VINVSG260002`…) is unique per invoice. `ParameterErr` is
not — every Singapore request gets the identical string. The branch's entire queue ends up
sharing one key. (Side effect: on the approval page, all pending Singapore invoices appear
lumped under the single entry "ParameterErr".)

**Step 7 — The damage.**
When finance rejects an entry, `ff.Update_ChqReq_Reject` clears by that key:

```sql
Update ff.ICost set ChqReqBy=null, ChqReqSent=null, PurEntryNo=null, PurEntryDate=null
where PurEntryNo = 'ParameterErr'   -- matches EVERYTHING, not one invoice
```

One rejection → every pending Singapore request un-sent. That is the vanish-and-resend
loop from the ticket: ops sends → someone rejects *anything* → ops's request silently
vanishes → ops re-sends → repeat. Proven in the reject log: single rejections clearing
13, 40, even **150** cost rows at once.

**After the fix, only Step 4 changes** — the lookup finds the new row (NumId 31331),
increments its counter, and returns a real formatted number (`VINV`+`SG`+`26`+`0002` →
`VINVSG260002`). Steps 5–7 then behave correctly by themselves: unique number saved,
unique number in the mail, rejection matches exactly one invoice.

The fix does **not** change Step 5's blindness — the procedure still wouldn't notice a
failure if a series expires or goes missing again. Permanent-fix recommendation (in the
report): make `Update_2ChequeRequest` abort when the generator returns `ParameterErr`
or `toinsert`.

---

## Why "the email was not sent" (what users experienced)

The mail machinery itself was never broken. Three things created the perception:

1. **Requests kept being erased** (Step 7), so finance's queue showed nothing and no
   follow-up mail was expected/seen — it looked like the send failed.
2. **The mail goes to the West-Africa distribution, not Singapore finance** — recipients
   come from `ff.Email` (`EType='CHQREQUEST'`), and these bookings route by their
   controlling office = GHA: TO `t.sasikumar@`, CC `wafops@`, `ap.intl@`,
   BCC `alert@` (hardcoded in the frontend). Anyone watching a Singapore mailbox saw
   nothing, ever — by design.
3. **Delivery was never provable** — the API's mail logging function is a no-op (body
   commented out), so only the recipients' mailboxes / the `alert@` BCC can confirm receipt.

Two separate lookups — don't confuse them:

| Lookup | Table | Key | Purpose |
|---|---|---|---|
| Number | `appConfig.Numbers` | `BranchId + NType` | Issue the Purchase Entry No (this bug) |
| Mail | `ff.Email` | `EType + BranchFrom` | Decide who receives the notification |

---

## Where the entry number is saved (and how to look it up)

```
appConfig.Numbers    → the counter       (Number = how many issued; not the number itself)
ff.ICost             → the live state    (erasable; what the pages read)   [BB_Cost/MJ_Cost/CC_Cost for Exp/Misc/CHA]
VendorPayment        → send register     (permanent; one row per send; payment recorded here later)
Cheque_Request_List  → send history      (permanent; one row per invoice per send, full detail)
```

Example lookups with the first real number:

```sql
-- the counter
select BranchId, NType, Number from appConfig.Numbers where NumId = 31331;

-- live state (3 cost lines of invoice 7552555066)
select iCostID, VendorInvNo, ChqReqSent, PurEntryNo from ff.ICost where PurEntryNo = 'VINVSG260001';

-- send register
select * from Bss_Report.ff.VendorPayment where PurEntryNo = 'VINVSG260001';

-- send history
select * from Bss_Report.ff.Cheque_Request_List where PurEntryNo = 'VINVSG260001';
```

**Signature of this bug:** live state NULL but history rows exist = "sent, then wiped".

---

## How to see ParameterErr in the database

```sql
-- 1. Live rows currently stuck with it (health check — should stay empty for S now)
select iCostID, VendorInvNo, ChqReqSent, PurEntryNo from ff.ICost   where PurEntryNo='ParameterErr';
select CostID,  VendorInvNo, ChqReqSent, PurEntryNo from ff.BB_Cost where PurEntryNo='ParameterErr';

-- 2. Full history of every failed send (~17,800 rows across S / Y / D since 2023)
select Typ, JobNumber, VendorInvNo, PurEntryNo, Branch, Created
from Bss_Report.ff.Cheque_Request_List
where PurEntryNo='ParameterErr' order by Created desc;

-- 3. The mass-wipes (look at CostIDList — one reject clearing up to 150 rows)
select Segment, PurEntryNo, CostIDList, RejectedBy, Rejected
from Bss_Log.ffL.ChqReq_Reject_Log
where PurEntryNo='ParameterErr' order by Rejected desc;
```

---

## Proving the root cause (3 queries)

```sql
-- 1. Which branches HAVE a VINV series (GHA, RTM, ANT, UKM, T, M ... — no SG/NY/D before the fix)
select BranchId, YearCode, DateFrom, DateTo, Number, isActive
from appConfig.Numbers where NType='VINV' order by BranchId, DateFrom desc;

-- 2. Which branch codes ASK for VINV numbers (generator logs each call pre+post remap → pairs)
select Branch, count(*) calls from ffLog.Lg_AppConfig_Number
where NType='VINV' group by Branch order by calls desc;
-- pairs: S=SG(1,219)  Y=NY(7,013)  W=ANT  R=RTM  E=UKM ; D alone (not remapped)

-- 3. The damage per branch (only the series-less branches appear)
select Branch, count(*) failed_sends, min(Created) first_seen, max(Created) last_seen
from Bss_Report.ff.Cheque_Request_List
where PurEntryNo='ParameterErr' group by Branch order by failed_sends desc;
-- Y 11,661 (still active) | D 3,823 (still active) | S 2,065 (stopped at the fix)
-- GHA 63 / LFW 8 — stopped Mar 2025 when THEIR series rows were created (the precedent)
```

Argument in one sentence: every branch whose (remapped) code has a series row produces
real numbers; every branch whose code lacks one produces `ParameterErr`; Singapore's
code `SG` lacked one.

---

## The fix (applied 09 Jul 2026, NumId 31331)

```sql
-- Preview: expect 0 rows
select * from appConfig.Numbers where BranchId='SG' and NType='VINV';

-- FIX (1 row): creates Singapore's purchase-entry number series
insert into appConfig.Numbers (BranchId, Segment, NType, Number, YearCode, DateFrom, DateTo, isActive, NFormat, POD)
values ('SG', '', 'VINV', 0, '26', '2026-01-01', '2026-12-31', 1, 'TypeBranchFY9999', '');

-- Verify: expect 1 row, Number = 0
select * from appConfig.Numbers where BranchId='SG' and NType='VINV';

-- Revert (ONLY while Number is still 0):
-- delete from appConfig.Numbers where BranchId='SG' and NType='VINV' and Number=0;
```

Scope: **branch-wide** (all Singapore cheque requests, all jobs, permanently) — the ticket's
jobs are just where it was reported and verified. Contains no job/invoice data; it is only
the number counter.

### Verification result (09 Jul 2026)

| Time | Invoice | Result |
|---|---|---|
| 04:58 | 7552555066 (job 2608785) | `ParameterErr` — sent BEFORE the insert |
| 05:41 | 7552555066 (job 2608785) | **`VINVSG260001`** — sent AFTER the insert; all 3 cost rows stamped |

Perfect before/after on the same invoice, same user, same day.

---

## Remaining checklist

- [ ] Confirm the 05:41 mail arrived (t.sasikumar / wafops / ap.intl / alert@ BCC)
- [ ] Re-send invoice **8506988862** (job **2607794** — the ticket's real job; ticket said 2608785) → expect `VINVSG260002`
- [ ] Re-send job **2602711**'s import invoices (2553204170 / 2553206870 / 2553206871)
- [ ] Reject-precision test: reject one `VINVSG…` entry → only its own rows clear
- [ ] Clean QA's test rows: `update ff.ICost set ChqReqBy=null, ChqReqSent=null, PurEntryNo=null, PurEntryDate=null where VendorInvNo='Test123' and BookingID=85159;`
- [ ] If this environment was staging: run the same fix on **production**
- [ ] Branches **Y** (insert `NY` row) and **D** (insert `D` row) — same defect, still active; drafts in the report, need finance sign-off (Y's whole queue shares one key)
- [ ] Until Y/D are fixed: finance must **not reject** any `ParameterErr` entry

## Permanent fixes recommended (dev team)

1. `ff.Update_2ChequeRequest` — abort with an error when `Get_Number` returns `ParameterErr`/`toinsert`
2. `ff.Update_ChqReq_Reject` — clear by CostID list (already computed for its log), not by `PurEntryNo`
3. Yearly series rollover check (every row has a `DateTo`; expiry produces `toinsert`, e.g. DXB ended 2023-12-31)
4. Add logging to `ff.Cancel_Purchase_Posting` (it deletes its own audit rows on cancel)
