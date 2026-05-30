# How I Found My First IDOR During Bug Bounty Testing

*By Cyber | Day 36 of my 100 Day Bug Bounty Challenge*

---

## Background

By Day 36 of my bug bounty journey, I had spent weeks studying web vulnerabilities through labs and practice environments. I understood concepts like XSS, SQL injection, and IDOR, but I wanted to see whether I could identify a real issue on a live target.

My goal was simple: move beyond solving labs and start applying the same thinking to real applications.

---

## The Approach

I selected a bug bounty program that provided a safe environment for testing and began exploring the application as a normal user.

Rather than looking for a specific vulnerability, I focused on understanding how the application worked:

- Intercepting requests with Burp Suite
- Mapping user workflows
- Identifying objects and identifiers passed between the client and server
- Asking a simple question: does the server verify ownership of this object?

That question became the foundation of my testing methodology.

---

## The Discovery

While testing a communication feature, I observed how identifiers were passed between the client and server during normal application usage.

The application correctly authenticated users, but further testing revealed that authorization checks were not consistently enforced for certain actions.

I built a controlled test case to validate this behavior and observed that it was possible to modify a request in a way that affected a resource outside of the intended user context.

At that point, I stopped further exploitation and focused on documenting the behavior.

---

## Understanding the Issue

The root cause was an authorization issue rather than an authentication issue.

- Authentication confirms who the user is
- Authorization determines what the user is allowed to access

In this case, the application correctly identified the user but failed to properly enforce ownership restrictions on specific resources.

This is a common pattern in Insecure Direct Object Reference (IDOR) vulnerabilities.

---

## Responsible Reporting

After validating the behavior, I documented:

- Steps to reproduce
- Evidence of the issue
- Impact analysis
- Recommended remediation steps

The report was submitted through the official disclosure channel.

Specific technical details have been omitted from this writeup in accordance with responsible disclosure practices.

---

## What I Learned

### 1. Recon is more important than payloads
The issue was not found through automation or complex payloads, but through careful observation of how the application handled requests.

---

### 2. Authorization flaws are subtle
Even when authentication is properly implemented, missing authorization checks can still lead to serious security issues.

---

### 3. Understanding data flow is key
Following how data moves from client to server helped identify where trust assumptions were being made incorrectly.

---

### 4. Labs translate to real-world testing
Concepts learned in controlled environments directly applied to real-world testing scenarios.

---

### 5. Documentation is part of the skill
Being able to clearly describe a vulnerability is just as important as finding it.

---

## Final Thoughts

This experience reinforced that bug bounty hunting is less about tools and more about thinking.

The key question that guided this finding was:

> What is the server trusting that it should not be trusting?

That mindset continues to guide my approach to security testing.

---

## Follow the Journey

This finding was documented as part of my 100 Day Bug Bounty Challenge (Day 36).

I’m sharing my journey as I learn — including wins, mistakes and lessons from real-world security testing.

You can follow my progress on X: **@cybermansec**

The hunt continues.

---

*Day 36/100 — First reported authorization vulnerability.*
