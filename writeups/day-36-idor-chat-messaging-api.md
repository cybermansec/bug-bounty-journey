# IDOR in Chat Messaging API Allows Cross-Conversation Message Injection

## Summary
During security testing of a live e-commerce application, I identified an IDOR vulnerability in the chat messaging feature that allows messages to be injected into another user’s conversation by manipulating conversation identifiers.

## Vulnerable Endpoint
POST /api/.../conversations/{conversationId}/messages

## Steps to Reproduce
1. Create two separate user sessions (User A and User B)
2. Capture chat request using Burp Suite
3. Extract conversation identifier and related parameters
4. Replace User A values with User B values
5. Forward modified request
6. Observe HTTP 201 Created response
7. Message appears in target conversation

## Impact
- Cross-conversation message injection
- User impersonation risk
- Potential social engineering attacks
- Chat system abuse

## Notes
This issue was discovered during live security testing and reported through responsible disclosure channels. Resolution status was confirmed to be a duplicate of a bounty awarded report.

## Tools Used
- Burp Suite
- Browser Developer Tools
