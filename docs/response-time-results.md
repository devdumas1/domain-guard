# Domain Guard: Response Time Results

The first response-time data collected from the agent, over one month of monitoring a 43-domain portfolio. Each incident's **Domain Guard** time is compared with the **control**: an estimate of how long a user would take to find and fix the same error manually. Times are in minutes.

- **Cause known:** time until the cause of the issue was identified.
- **Fix verified:** time until the fix was confirmed working.

| Domain | Issue | Control: cause known | Domain Guard: cause known | Control: fix verified | Domain Guard: fix verified |
|---|---|---:|---:|---:|---:|
| prdpanama.com | Missing DMARC | 6 | 1 | 20 | 8 |
| domaintechinv.com | DNS rotation alert | 5 | 1 | 8 | 2 |
| panamasummit.com | HTTP 5xx outage | 18 | 5 | 55 | 23 |
| datingoffers.co.uk | False alarm | 9 | 2 | 15 | 5 |
| yogapanama.com | Domain expiring soon | 5 | 1 | 25 | 10 |
| invoiceagency.com | Brief, self-recovering outage | 2 | 1 | 5 | 4 |
| cbranch.com | Broken SPF | 12 | 3 | 35 | 12 |
| domaintechinv.com | Repeated DNS rotation alert | 6 | 1 | 10 | 3 |
| netwebservers.com | Missing DMARC | 7 | 2 | 25 | 9 |
| panamasummit.com | Second HTTP 5xx outage | 14 | 4 | 45 | 18 |
| tupanama.com | Wrong A record target (expected host known) | 15 | 4 | 40 | 14 |
| datingoffers.co.uk | Repeated false alarm | 7 | 2 | 12 | 4 |
| invoiceagency.com | Second self-recovering outage | 3 | 1 | 7 | 5.6 |
| harmonytrust.com | Missing DMARC | 5 | 1 | 18 | 7 |
| panamasummit.com | Third HTTP 5xx outage | 16 | 5 | 55 | 24.8 |
| domaintechinv.com | Repeated DNS rotation alert | 5 | 1 | 9 | 3 |
| **Average** | | **8.44** | **2.19** | **24.00** | **9.53** |

## Summary

Across 16 incidents:

| Measure | Control (avg) | Domain Guard (avg) | Reduction |
|---|---:|---:|---:|
| Cause known | 8.44 min | 2.19 min | 74% |
| Fix verified | 24.00 min | 9.53 min | 60% |
