# Domain Guard: Response Time Results

The first response-time data collected from the agent, over one month of monitoring a 43-domain portfolio. Each incident's **Domain Guard** time is compared with the **control**: an estimate of how long a user would take to find and fix the same error manually. Times are in minutes.

- **Cause known:** time until the cause of the issue was identified.
- **Fix verified:** time until the fix was confirmed working.
- **Resolved by Domain Guard:** Domain Guard applied the fix itself through a one-click approved DNS change, with no manual troubleshooting.

| Domain | Issue | Control: cause known | Domain Guard: cause known | Control: fix verified | Domain Guard: fix verified | Resolved by Domain Guard |
|---|---|---:|---:|---:|---:|:---:|
| prdpanama.com | Missing DMARC | 6 | 1 | 20 | 8 | Yes |
| domaintechinv.com | DNS rotation alert | 5 | 1 | 8 | 2 | No |
| panamasummit.com | HTTP 5xx outage | 18 | 5 | 55 | 23 | No |
| datingoffers.co.uk | False alarm | 9 | 2 | 15 | 5 | No |
| yogapanama.com | Domain expiring soon | 5 | 1 | 25 | 10 | No |
| invoiceagency.com | Brief, self-recovering outage | 2 | 1 | 5 | 4 | No |
| cbranch.com | Broken SPF | 12 | 3 | 35 | 12 | Yes |
| domaintechinv.com | Repeated DNS rotation alert | 6 | 1 | 10 | 3 | No |
| netwebservers.com | Missing DMARC | 7 | 2 | 25 | 9 | Yes |
| panamasummit.com | Second HTTP 5xx outage | 14 | 4 | 45 | 18 | No |
| tupanama.com | Wrong A record target (expected host known) | 15 | 4 | 40 | 14 | Yes |
| datingoffers.co.uk | Repeated false alarm | 7 | 2 | 12 | 4 | No |
| invoiceagency.com | Second self-recovering outage | 3 | 1 | 7 | 5.6 | No |
| harmonytrust.com | Missing DMARC | 5 | 1 | 18 | 7 | Yes |
| panamasummit.com | Third HTTP 5xx outage | 16 | 5 | 55 | 24.8 | No |
| domaintechinv.com | Repeated DNS rotation alert | 5 | 1 | 9 | 3 | No |
| **Average** | | **8.44** | **2.19** | **24.00** | **9.53** | **5 of 16 (31.25%)** |

## Summary

Across 16 incidents:

| Measure | Control (avg) | Domain Guard (avg) | Reduction |
|---|---:|---:|---:|
| Cause known | 8.44 min | 2.19 min | 74% |
| Fix verified | 24.00 min | 9.53 min | 60% |

Domain Guard resolved 5 of the 16 incidents (31.25%) itself: three missing DMARC records, a broken SPF record and a wrong A record target.
