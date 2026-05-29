# Day 70 - Bug Bounty Journey

Today I continued testing live targets with a focus on XSS discovery.

Activities:

* Moved to a new bug bounty program.
* Tested search functionality and captured requests using Burp Suite.
* Modified search request parameters to observe application behavior.
* Received HTTP 403 responses from a Web Application Firewall (WAF).
* Confirmed that request modifications were being blocked before reaching the application.

Key Learning:

Understanding security controls is part of vulnerability research. Not every interesting response leads to a vulnerability, but observing how applications and WAFs react to user input helps build a better understanding of attack surface and filtering mechanisms.

Status:

Continuing reconnaissance and XSS testing while focusing on input flow, filtering behavior, and potential reflection points.
