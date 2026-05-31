# How I Found an IDOR in a Major E-Commerce Platform's Chat API

*By Cyber | Day 36 of my 100 Day Bug Bounty Challenge*

---

## Background

I was 36 days into a 100 day bug bounty challenge. I document everything publicly on X (@cybermansec) — the wins, the losses, the signal walls, all of it.

This is the story of how I found my first real vulnerability on a live bug bounty target. Not a lab. Not a simulation. A real production application used by hundreds of thousands of merchants worldwide.

---

## Setting the Scene

I had been working through PortSwigger labs for weeks — XSS, IDOR, SQL injection, access control. I understood the concepts. But there's a gap between understanding a vulnerability in a controlled lab environment and finding one independently on a real target.

The goal was to close that gap.

I chose a major e-commerce platform with a well-documented bug bounty program and a clear scope. They explicitly invite researchers to test against development stores — meaning I could create my own test environment with full permission.

I set up three development stores and got to work.

---

## Recon Phase

My methodology was simple:

1. Walk the application as a normal user
2. Intercept every request in Burp Suite
3. Map every input field and parameter
4. Ask one question for each: *does the server verify ownership of this object?*

I tested the obvious surfaces first — search bar, contact form, profile name editor. The platform handled all of them well. Input was sanitized, encoded, or rejected at the server level.

Then I installed a first-party chat application built by the platform's own team. Chat applications are interesting targets because they handle real-time user input, store messages, and expose them to multiple parties — the customer and the merchant admin.

---

## Finding the Vulnerability

I opened the chat widget on my test store as a customer and sent a message. In Burp Suite's HTTP history I captured the request:

```
POST /[chat-api]/storefront/conversations/{conversationId}/messages HTTP/2
Host: [messaging-api-host]
Content-Type: application/json
X-[Platform]-Chat-Shop-Identifier: [store-specific-identifier]

{
  "message": {
    "type": "text",
    "content": {"text": "Hello"},
    "group": "customer",
    "user_token": "[customer-token]"
  }
}
```

Two parameters immediately stood out:

**1. The Conversation ID in the URL**
A unique identifier for the conversation between a customer and a specific store. It appeared in plain text in the URL path.

**2. The Shop Identifier in the header**
A store-specific value sent with every chat request — also visible in plain text in any storefront's network traffic. No authentication required to see it.

Both values are publicly visible to anyone who opens the chat widget on any store and inspects their network requests.

The question I asked: *what happens if I swap these values with those from a different store?*

---

## The Test

I set up a second development store, created a customer account, and initiated a chat session. I now had:

- **Store A** — Conversation ID and Shop Identifier
- **Store B** — Conversation ID and Shop Identifier

In Burp Suite's Repeater I took Store A's chat request and made two modifications:

1. Replaced the Conversation ID in the URL with Store B's Conversation ID
2. Replaced the Shop Identifier header with Store B's Shop Identifier

I kept everything else identical — Store A's session, user token, and origin headers.

I sent the modified request.

**The server returned HTTP 201 Created.**

```json
{
  "message": {
    "id": "...",
    "type": "text",
    "content": {"text": "Hello I'm Store A's customer"},
    "origin": "storefront"
  }
}
```

---

## Confirming the Impact

I logged into Store B's merchant admin inbox.

The message was there. Sitting in Store B's conversation — sent from Store A's customer session, using Store A's credentials, with no authorization from Store B's merchant.

The server had accepted the request without verifying that the customer's session belonged to Store B. It only checked that the Conversation ID and Shop Identifier matched — both of which were publicly obtainable.

---

## Technical Breakdown

The vulnerability exists because the API performs authentication but not authorization.

**Authentication** — the server verifies that you have a valid customer session. ✓

**Authorization** — the server verifies that your session belongs to the store you're sending messages to. ✗

The missing check is: *does this customer's session belong to the store associated with this Conversation ID?*

Without that check, any customer from any store can inject messages into any other store's conversation by:

1. Visiting the target store's storefront — no account needed
2. Opening the chat widget — the Shop Identifier is immediately visible in network traffic
3. Starting a chat on any store to obtain a valid Conversation ID
4. Modifying the request in Burp Suite to swap both values
5. Sending the modified request

The attack requires zero authentication beyond being a normal website visitor.

---

## Impact

**Who is affected:**
Any merchant using the chat feature on this platform.

**What an attacker can do:**
- Impersonate customers to manipulate merchant support conversations
- Flood merchant inboxes with spam or fake complaints
- Social engineer merchants into issuing refunds or revealing order details
- At scale — automate cross-store message injection targeting thousands of merchants simultaneously

**Severity assessment:** Medium
- No sensitive data exfiltrated
- No account takeover
- No authentication bypass
- But real potential for business disruption, social engineering, and merchant manipulation at scale

---

## Responsible Disclosure

This vulnerability has been reported through the platform's official bug bounty program. In accordance with responsible disclosure practices, I am not naming the platform until the vulnerability has been remediated and I have received permission to disclose.

The report includes:
- Full steps to reproduce
- Proof of concept screenshots
- Impact assessment
- Suggested remediation: verify that the requesting customer's session is associated with the store linked to the Conversation ID before processing the request

---

## What I Learned

**1. Large platforms harden the obvious surfaces.**
Search bars, form fields, profile inputs — these get tested constantly and are usually well protected. The interesting vulnerabilities live in less obvious places: APIs, third-party integrations, newer features.

**2. Authentication ≠ Authorization.**
This is the root cause of IDOR vulnerabilities. The server knew *who* I was. It just didn't check *what I was allowed to do*. One missing ownership verification = unauthorized access.

**3. Publicly visible parameters are always worth testing.**
Both the Conversation ID and Shop Identifier were visible to anyone in plain network traffic. The assumption that "it's just an identifier, not a secret" doesn't hold when there's no server-side ownership check backing it up.

**4. Recon is everything.**
I found this by walking the application methodically, capturing every request, and asking the same question for every parameter: *does the server verify ownership?* That question is the foundation of IDOR hunting.

**5. Development stores are real attack surfaces.**
The platform gave me permission to test against development stores — but the API endpoints I tested are the same ones used in production. What works on a development store works everywhere.

---

*Follow my 100 day bug bounty journey on X: @cybermansec*

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

This finding was documented as part of my 100 Day Bug Bounty Challenge (Currently in day 71).

I’m sharing my journey as I learn — including wins, mistakes and lessons from real-world security testing.

You can follow my progress on X: **@cybermansec**

The hunt continues.

---

*Day 36/100 — First reported authorization vulnerability.*
