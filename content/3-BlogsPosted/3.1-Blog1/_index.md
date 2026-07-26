---
title: "Blog 1"
date: 2026-07-26
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# CENTRALIZED AMAZON BEDROCK ACCESS MANAGEMENT AND CONTROL WITH AWS AI GATEWAY PATTERN

Hello everyone!

When deploying Generative AI in an enterprise, governance is always a major challenge: how do you handle authorization, control costs, enforce rate limiting, and ensure data isolation between tenants?

Today, I want to share insights about the AI Gateway pattern developed by Dynatrace and standardized by AWS into a reference pattern. This architecture helps centrally manage all access to Amazon Bedrock by combining familiar serverless services.

## Context & Problem

If a client application calls Amazon Bedrock directly via the AWS SDK, the system will encounter several limitations:

- Difficult to integrate with existing identity systems (like OAuth2/JWT).
- Hard to manage quotas and costs per team/tenant in a multi-tenant architecture.
- Requires updating intermediate code every time Bedrock releases a new feature or API.

**Solution:** Build an AI Gateway layer in front of Bedrock. This layer remains transparent to the client—developers continue using Boto3/AWS SDK as usual—while the entire auth, quota, and streaming flow is handled centrally in the backend.

## Architecture Highlights

- **Future-Proof Design with Dynamic Forwarder:** The Lambda integration signs the AWS SigV4 request and forwards it. Regardless of whether Bedrock adds new models or APIs, the AI Gateway handles it smoothly without requiring code changes.
- **Flexible Identity Integration:** Uses a Lambda Authorizer to validate JWT tokens from Auth0, Keycloak, or Okta before the request reaches Bedrock.
- **Real-Time Response Streaming Support:** Leverages API Gateway Response Streaming to stream data chunks directly from the LLM back to the client with minimal latency.
- **Anti-Noisy Neighbor & Cost Control:** Easily applies Usage Plans, API Keys, and rate limiting on API Gateway to control traffic for each team.

## Real-World Scenario: Request Processing Flow

When a client calls the ConverseStream API for the Claude 3.5 Haiku model via the AI Gateway, the process flows as follows:

```text
Client (Boto3 SDK)
  │ (API call unsigned + JWT header)
  ▼
Amazon API Gateway
  │ (Calls Lambda Authorizer to validate JWT)
  ▼
AWS Lambda Integration
  │ (Preserves request -> signs AWS SigV4)
  ▼
Amazon Bedrock
  │ (Streams response in real time)
  ▼
Client receives streamed data
````

Client initialization: the client points `endpoint_url` to the API Gateway, disables client-side signing (`signature_version = UNSIGNED`), and attaches the JWT token in the header.

Authentication at API Gateway: the API Gateway calls the Lambda Authorizer to verify the validity of the JWT token.

Digital signing & forwarding: the Lambda integration receives the request, keeps the payload/parameters intact, signs it with AWS SigV4 using the Lambda's IAM role, and forwards it to Amazon Bedrock.

Streamed response: the API Gateway streams data chunks (`contentBlockDelta`) directly from Bedrock back to the client.

## Extensibility

Because it sits at the API Gateway layer, you can easily add advanced features:

* Prompt & Response Caching: cache common queries to reduce API costs and latency.
* AWS WAF Integration: add security rules to prevent SQLi, XSS, or enforce IP restrictions.
* Custom Content Filtering: add logic to filter sensitive data (PII) within Lambda before sending the prompt to Bedrock.

## Conclusion

The AI Gateway pattern strikes a great balance between developer experience and enterprise governance. Client-side developers can continue using official SDKs smoothly, while the Security & Cloud Ops teams maintain complete control over system access, traffic, and costs.

## References

* AWS Compute Blog – Building an AI gateway to Amazon Bedrock with Amazon API Gateway
* GitHub Repository: amazon-api-gateway-ai-gateway-pattern

<img src="/AWS_PhuongAnh/images/Blogs/blog1.png" alt="Blog 1" width="1000" />

