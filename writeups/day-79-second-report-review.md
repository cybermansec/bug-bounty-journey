# Day 79 - Report Review and Lessons Learned

## Background

A day after submitting my second bug bounty report, I received the outcome from the program.
The issue involved user-controlled input causing unhandled ASP.NET exceptions and exposing application-generated error pages.
After review, the report was marked as a duplicate of a previously submitted report and was ultimately considered Informative by the program.

## What Happened

While testing a live target, I discovered that specific input could trigger server-side errors and expose framework-generated error pages.
This behavior appeared unusual enough to investigate further, so I documented the issue and submitted a report through the program's official disclosure process.
It was reviewed and linked to an earlier submission describing the same underlying behavior.

## Outcome

* Report Status: Duplicate
* Original Classification: Informative
* Reward: None
Although the report did not result in a valid security finding, it still provided valuable experience with the reporting and review process.

## Looking Forward

This is now the second report I have submitted to a real program. It was not the outcome I hoped for, but the process provided another opportunity to learn and improve.

