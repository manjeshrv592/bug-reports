# Bug Report Template & Reference

## File Naming
```
TASK-{task_number}.html
```
Example: `TASK-31113.html`

---

## Step 1 — Register in index.html

Add one entry to the `reports` array in `index.html`:

```javascript
{
  task: '99999',
  title: 'Short title of the bug',
  module: 'Module Name',           // e.g. Cheque Request, Buy Rate, Sea Import
  branch: 'Branch Name (Code)',    // e.g. Rotterdam (R)
  jobs: 'JOB123456',
  invoices: 'INV123',              // if applicable
  date: '01 Jul 2026',
  file: 'TASK-99999.html',
  summary: 'One-line summary of the issue and root cause.'
}
```

---

## Step 2 — Create TASK-{number}.html

Copy the full HTML below and replace every `<!-- TODO -->` placeholder.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>TASK-99999 &mdash; <!-- TODO: module name --></title>
  <style>
    html { scroll-behavior: smooth; }
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: 'Segoe UI', Arial, sans-serif;
      font-size: 13px;
      color: #1a1a1a;
      background: #fff;
      padding: 40px;
      max-width: 940px;
      margin: auto;
    }

    .header { background: #1a3c6e; color: #fff; padding: 24px 28px; border-radius: 6px; margin-bottom: 28px; }
    .header h1 { font-size: 20px; font-weight: 700; margin-bottom: 6px; }
    .header p { font-size: 12px; color: #b0c4de; }

    .meta-box {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 10px;
      background: #f4f7fb;
      border: 1px solid #d0dbe8;
      border-radius: 6px;
      padding: 16px 20px;
      margin-bottom: 32px;
    }
    .meta-box .item label { font-size: 11px; color: #6b7a90; text-transform: uppercase; letter-spacing: 0.5px; display: block; margin-bottom: 3px; }
    .meta-box .item span { font-weight: 600; color: #1a1a1a; }

    .page-index { background: #f4f7fb; border: 1px solid #d0dbe8; border-radius: 6px; padding: 14px 20px; margin-bottom: 28px; }
    .page-index p { font-size: 11px; font-weight: 700; color: #6b7a90; text-transform: uppercase; letter-spacing: 0.5px; margin-bottom: 8px; }
    .page-index ol { margin: 0; padding-left: 18px; }
    .page-index li { margin-bottom: 4px; }
    .page-index a { font-size: 13px; color: #1a3c6e; text-decoration: none; }
    .page-index a:hover { text-decoration: underline; }

    .issue-block { border: 1px solid #d0dbe8; border-radius: 8px; margin-bottom: 36px; overflow: hidden; }
    .issue-header { background: #1a3c6e; color: #fff; padding: 14px 20px; }
    .issue-header h2 { font-size: 15px; font-weight: 700; margin-bottom: 2px; }
    .issue-header p { font-size: 12px; color: #b0c4de; }
    .issue-body { padding: 20px; }

    .takeaway { background: #f0f4ff; border-left: 4px solid #1a3c6e; border-radius: 0 5px 5px 0; padding: 11px 16px; margin-bottom: 18px; font-size: 12.5px; color: #1a1a1a; line-height: 1.6; }
    .takeaway strong { display: block; font-size: 11px; text-transform: uppercase; letter-spacing: 0.5px; color: #6b7a90; margin-bottom: 4px; }

    .sec { margin-bottom: 22px; }
    .sec-title { font-size: 13px; font-weight: 700; color: #1a3c6e; border-left: 4px solid #1a3c6e; padding-left: 10px; margin-bottom: 12px; }
    .sec p { line-height: 1.7; color: #333; margin-bottom: 8px; }

    .step { border: 1px solid #e0e7ef; border-radius: 6px; margin-bottom: 14px; overflow: hidden; }
    .step-header { background: #e8eef7; color: #1a3c6e; padding: 8px 14px; font-weight: 600; font-size: 12.5px; }
    .step-body { padding: 14px 16px; }
    .step-body p { margin-bottom: 8px; line-height: 1.6; color: #333; }

    pre {
      background: #1e1e2e;
      color: #cdd6f4;
      padding: 14px 16px;
      border-radius: 5px;
      font-family: 'Consolas', 'Courier New', monospace;
      font-size: 12px;
      line-height: 1.6;
      overflow-x: auto;
      margin: 10px 0;
      white-space: pre-wrap;
      word-break: break-word;
    }

    .result-box {
      background: #f0f4f0;
      border: 1px solid #b2d0b2;
      border-radius: 5px;
      padding: 10px 14px;
      font-family: 'Consolas', 'Courier New', monospace;
      font-size: 12px;
      color: #1a1a1a;
      margin: 10px 0;
      white-space: pre-wrap;
    }

    table { width: 100%; border-collapse: collapse; margin: 10px 0; font-size: 12px; }
    th { background: #1a3c6e; color: #fff; padding: 8px 10px; text-align: left; font-weight: 600; }
    td { padding: 7px 10px; border-bottom: 1px solid #e0e7ef; vertical-align: top; }
    tr:nth-child(even) td { background: #f4f7fb; }
    .highlight-row td { background: #fdecea !important; }
    .pass { color: #1a7a1a; font-weight: 700; }
    .fail { color: #c0392b; font-weight: 700; }

    .alert { border-radius: 5px; padding: 12px 16px; margin: 12px 0; font-size: 12.5px; line-height: 1.6; }
    .alert-danger  { background: #fdecea; border-left: 4px solid #c0392b; color: #7b1e1e; }
    .alert-warning { background: #fff8e1; border-left: 4px solid #f39c12; color: #7a5c00; }
    .alert-info    { background: #e8f4fb; border-left: 4px solid #2980b9; color: #1a4a6e; }
    .alert strong  { display: block; margin-bottom: 4px; }

    .back-link { display: inline-block; margin-bottom: 20px; font-size: 12px; color: #1a3c6e; text-decoration: none; }
    .back-link:hover { text-decoration: underline; }

    .footer { margin-top: 36px; border-top: 1px solid #d0dbe8; padding-top: 14px; font-size: 11px; color: #888; display: flex; justify-content: space-between; }

    @media print {
      body { padding: 20px; }
      .issue-block { page-break-inside: avoid; }
    }
  </style>
</head>
<body>

  <a href="index.html" class="back-link">&larr; Back to Bug Reports Index</a>

  <div class="header">
    <h1>TASK-99999 &mdash; <!-- TODO: bug title --></h1>
    <p>System: BLISS &nbsp;|&nbsp; Module: <!-- TODO --> &nbsp;|&nbsp; Date: <!-- TODO: DD Month YYYY --></p>
  </div>

  <div class="meta-box">
    <div class="item"><label>Task</label><span>99999</span></div>
    <div class="item"><label>Module</label><span><!-- TODO --></span></div>
    <div class="item"><label>Date Reported</label><span><!-- TODO --></span></div>
    <div class="item"><label>Issues in Task</label><span><!-- TODO: number --></span></div>
  </div>

  <div class="page-index">
    <p>Issues in this report</p>
    <ol>
      <li><a href="#issue-1"><!-- TODO: one-line description of Issue 1 --></a></li>
      <!-- Add more <li> entries for additional issues -->
    </ol>
  </div>


  <!-- ===========================
       ISSUE 1
       =========================== -->
  <div class="issue-block" id="issue-1">
    <div class="issue-header">
      <h2>Issue 1 &mdash; <!-- TODO: issue title --></h2>
      <p><!-- TODO: Job / Branch / key IDs --></p>
    </div>
    <div class="issue-body">

      <div class="takeaway">
        <strong>Key Takeaway</strong>
        <!-- TODO: one or two sentences on the general pattern — what to look for next time -->
      </div>

      <div class="sec">
        <div class="sec-title">Problem</div>
        <p><!-- TODO: what was reported, what is not working, which page/SP --></p>
      </div>

      <div class="sec">
        <div class="sec-title">Investigation</div>

        <div class="step">
          <div class="step-header">Step 1 &mdash; <!-- TODO --></div>
          <div class="step-body">
            <p><!-- TODO: what we checked and why --></p>
            <pre><!-- TODO: query --></pre>
            <div class="result-box"><!-- TODO: result --></div>
            <p><!-- TODO: conclusion from this step --></p>
          </div>
        </div>

        <!-- Copy the .step block above for each additional step -->

      </div>

      <div class="sec">
        <div class="sec-title">Root Cause</div>
        <div class="alert alert-danger">
          <strong><!-- TODO: table/object — specific ID: value --></strong>
          <!-- TODO: clear explanation of exactly what is wrong and why -->
        </div>
      </div>

      <div class="sec">
        <div class="sec-title">Solution</div>
        <p><!-- TODO: context / who confirmed / date applied --></p>
        <pre>-- TODO: final query or code change</pre>
        <!-- Use .alert-warning for temp fixes that need reverting -->
        <!-- Use .alert-info for permanent fixes pending -->
      </div>

    </div>
  </div>


  <!-- Copy the entire .issue-block above for each additional issue, incrementing id="issue-2" etc. -->


  <!-- OPTIONAL: Observations — use when post-fix review reveals rows or states that look wrong but are expected/not bugs -->
  <!--
  <div style="margin-bottom: 28px;">
    <div style="font-size: 14px; font-weight: 700; color: #1a3c6e; border-left: 4px solid #1a3c6e; padding-left: 10px; margin-bottom: 12px;">Observations</div>
    <p style="color: #333; line-height: 1.7; margin-bottom: 10px;">
      TODO: describe any remaining anomalies that are expected / not bugs. E.g. empty Vendor Inv rows for
      charges where the vendor invoice has not yet been received.
    </p>
    <table>
      <thead>
        <tr><th><!-- TODO: col 1 --></th><th><!-- TODO: col 2 --></th><th>Reason</th></tr>
      </thead>
      <tbody>
        <tr><td><!-- TODO --></td><td><!-- TODO --></td><td><!-- TODO --></td></tr>
      </tbody>
    </table>
  </div>
  -->

  <!-- Tables & Objects Involved -->
  <div style="margin-bottom: 28px;">
    <div style="font-size: 14px; font-weight: 700; color: #1a3c6e; border-left: 4px solid #1a3c6e; padding-left: 10px; margin-bottom: 12px;">Tables &amp; Objects Involved</div>
    <table>
      <thead>
        <tr><th>Object</th><th>Type</th><th>Role</th></tr>
      </thead>
      <tbody>
        <tr><td><!-- TODO --></td><td><!-- Table / SP / View --></td><td><!-- TODO --></td></tr>
      </tbody>
    </table>
  </div>

  <div class="footer">
    <span>BLISS &mdash; Bug Report &nbsp;|&nbsp; TASK-99999 &nbsp;|&nbsp; <!-- TODO: Module --></span>
    <span><!-- TODO: date --></span>
  </div>

</body>
</html>
```

---

## Alert classes — when to use each

| Class | Use for |
|-------|---------|
| `alert-danger` | Root cause callout |
| `alert-warning` | Temp fix that needs a revert |
| `alert-info` | Permanent fix pending / notes |

---

## Common Root Causes (Cheque Request)

| Symptom | First thing to check |
|---------|----------------------|
| Record not showing | `TallyPosted IS NULL`, `ChqReqSent IS NULL`, `isActive = 1`, `CRevenueID IS NULL`, `BookingDate >= cutoff` |
| Import job missing | `ff.ICost` + `ff.BBooking_Import` — check `TallyPosted` and `BookingDate` |
| Export job missing | `ff.BB_Cost` + `ff.BJob` — check `bv.ETD is not null`, `ChqReqSent` |
| Change request blocking | `cr.Cost_Entries` — `ChangeRequestdBy NOT NULL` and `ResentBy NULL` |

---

## Key Objects Reference

| Object | Type | Purpose |
|--------|------|---------|
| `ff.Get_2ChequeRequest` | SP | Powers Cheque Request page |
| `ff.ICost` | Table | Import cost entries |
| `ff.BB_Cost` | Table | Export cost entries |
| `ff.MJ_Cost` | Table | Misc job cost entries |
| `ff.CC_Cost` | Table | CHA job cost entries |
| `ff.BBooking_Import` | Table | Import booking header |
| `ff.BJob` | Table | Export job header |
| `ff.MiscJob` | Table | Misc job header |
| `Cha.Job_CHA` | Table | CHA job header |
| `cr.Cost_Entries` | Table | Change request tracking |
