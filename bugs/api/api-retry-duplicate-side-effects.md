---
title: API Retry Creates Duplicate Side Effects
slug: api-retry-duplicate-side-effects
category: api
tags: [api, retries, idempotency, distributed-systems, duplicate-processing]
difficulty: intermediate
production_impact: high
root_cause_type: missing-idempotency
detection_method: fault-injection-testing
source_type: sourced-example
status: draft
---

# API Retry Creates Duplicate Side Effects

## The Problem

A client sends a state-changing API request, such as charging a card, placing an order, or creating a booking, and the response is lost or times out before the client receives it. The client cannot tell whether the server never processed the request or processed it successfully and only the response was lost, so it retries. If the original request already succeeded and the API has no reliable way to recognize the retry as a duplicate, the retry executes the same side effect a second time.

## Symptoms

A user reports being charged twice for a single checkout, seeing two identical orders in their history after clicking "place order" once, or receiving the same confirmation email or SMS twice within a short window. Support tickets describe an action that "only happened once" on the user's end but produced two records on the backend. The duplication is usually intermittent and correlates with slow responses, elevated latency, or client-side timeout settings rather than any specific input the user provided, since the trigger is a delayed or lost response rather than a data condition.

## Why It Happens

A request-response exchange over a network gives the client no reliable way to distinguish three outcomes once a response fails to arrive: the request never reached the server, the request reached the server but the server failed before completing it, or the server completed the request and the response was lost on the way back. A timeout tells the client only that it didn't get an answer in time, not which of these actually happened. Retrying is the correct response to the first two outcomes and the wrong response to the third, and the client has no way to know in advance which case it is in.

This is not a flaw specific to any one API or client library; it is a structural property of communicating over an unreliable network with a request-response protocol. It is also not a reason to avoid retries altogether: RFC 9110 defines certain HTTP methods as idempotent precisely so that a client can safely repeat a request without changing the outcome, even if the original attempt already succeeded (Section 9.2.2). The failure in this exhibit is not that a retry happened; it is that the request being retried was not idempotent and the API provided no mechanism, such as an idempotency key, to make it safe to repeat.

## Root Cause

The concrete fault is a state-changing operation that can execute more than once for what the caller considers a single logical request, because the API's idempotency handling is missing or incomplete. This shows up in several distinct ways, and a real system can suffer from more than one at once:

- **No idempotency mechanism at all.** The endpoint accepts a state-changing request and has no concept of a client-supplied request identifier. Every retry looks exactly like a new, independent request, because nothing in the request distinguishes "the same operation, sent again" from "a new operation that happens to look similar."
- **Non-atomic check-and-create logic.** The server checks whether a request with a given idempotency key has already been handled, then creates the resulting record, as two separate steps rather than one atomic operation. Two requests carrying the same key that arrive close enough together can both pass the "has this been seen before" check before either one has finished writing its result, and both proceed to create the side effect.
- **Concurrent requests reusing the same key.** Even with an idempotency mechanism in place, a request that is still executing when an identical retry with the same key arrives can leave the server without a defined behavior for "in progress" duplicates specifically, as opposed to "already completed" duplicates. If the server only guards against replaying a finished result and does not also guard against a second execution starting while the first is still running, the same race described above appears.
- **The same key reused across different request payloads.** A caching bug, a key derived from the wrong scope, or a client library that reuses a stored key across logically different operations, can cause a key to be attached to a payload other than the one it was originally generated for. Depending on how the server responds to that mismatch, it may execute the new payload as if it were the first use of that key, or return a stale cached result that doesn't match what the caller actually asked for this time; either outcome is a correctness failure, not merely a missed optimization.
- **Premature key expiration.** An idempotency key's cached result is retained for only a limited window. A legitimate retry that arrives after that window has closed, because the client itself was slow, was queued, or retried after an unusually long delay, is treated as a first-time request rather than a duplicate, and the side effect executes again.

## User Impact

The direct consequence is a duplicated business event: a customer billed twice for one purchase, two shipments created for one order, a hotel or flight booked twice for one reservation, or a notification delivered more than once for a single triggering event. Beyond the immediate inconvenience, financial duplications require the user or support staff to notice the discrepancy, dispute or reconcile it, and wait for a refund or correction, which erodes trust in the system even after the money is eventually returned. For non-financial side effects, such as duplicate notifications, the impact is smaller per occurrence but still visible and confusing to the end user, and repeated occurrences make the product look unreliable even when the underlying data eventually self-corrects.

## Developer Investigation

The most useful starting signal is a downstream side effect that occurred more times than there were distinct user actions: two charge records, two order rows, or two outbound notifications tied to what the user experienced as one action. From there, the investigation looks for two client-originated requests that are close together in time and, where the API has one, share the same idempotency key or client-generated request identifier. Correlating request IDs and timestamps across the client's own retry logic, load balancer or gateway logs, and the service's application logs usually shows the shape of the failure: either the same key appearing twice with two independent executions instead of one cached replay, or two different keys both mapped to what was, on the client side, the same logical action, which points instead at a client-side bug in key generation or key reuse.

It is also worth checking the service's own idempotency implementation directly, since a system that appears to "have idempotency" can still fail here: whether the check for an existing key and the write of a new result happen inside a single transaction or otherwise atomically, what happens when a second request with an in-flight key arrives before the first has finished, and how long completed keys are retained before they are eligible for reuse.

## QA Detection Strategy

This class of bug does not appear in a straightforward, single-request functional test, since that test never gives the client a reason to retry. Exposing it requires deliberately recreating the ambiguous-outcome scenario from the client's point of view:

- **Response-loss simulation.** Let the server fully process a state-changing request and commit its side effect, then drop or discard the response before it reaches the client, and observe what the client does next.
- **Timeout injection.** Introduce delay on the server side, or on a proxy between client and server, long enough to trip the client's own timeout, while the underlying operation still completes successfully in the background.
- **Concurrent duplicate requests.** Fire two or more requests carrying the same idempotency key, or the same client-generated identifier, at nearly the same moment, so that they race against each other rather than arriving safely sequential.
- **Retry testing.** Exercise the client's actual retry logic, rather than a hand-written duplicate request, so the test reflects the real retry count, backoff, and key reuse behavior the client applies in production.
- **Verification through the side effect itself, not the API response.** A 200-level response on a retry is not sufficient evidence of correct behavior, because a naive implementation can return a plausible-looking success on both the original request and the duplicate. The check that actually matters is on the downstream record: the database row count, ledger entry count, or count of outbound notifications for that logical operation, confirmed to be exactly one.

## Automation Strategy

An automated test for this should assert on an invariant rather than a specific interleaving, so it stays valid regardless of network timing: one logical user operation must produce at most one business side effect, no matter how many times the transport layer retries the underlying request. Concretely, this means driving the same logical operation through the client's real retry path, including deliberately injected response loss or timeout, and then checking the authoritative downstream state rather than any individual response.

```
given: one logical operation, retried N times via response-loss/timeout injection
       (including at least one case where retries race concurrently)
when:  all N attempts have completed or been abandoned by the client
then:  exactly one corresponding side effect exists downstream
       (e.g., exactly one charge, one order row, one notification sent)
       regardless of how many of the N attempts the server actually executed
```

This test is deliberately tool-agnostic. It does not assume a specific idempotency-key implementation, HTTP client, or backend, because the invariant it checks, one operation, one side effect, is what any correct idempotency mechanism is responsible for guaranteeing.

## Real-World Pattern

Idempotency keys for state-changing requests are a documented, standard mechanism, not a novel workaround, and vendor documentation on how they're implemented illustrates the specific failure modes above concretely.

Stripe's API accepts an `Idempotency-Key` header on state-changing requests, and its documentation describes the mechanism precisely: the server saves the resulting status code and body of the first request made for a given key and returns that same saved result for any subsequent request using the same key, so a client that retries after a lost connection does not risk creating a second object or repeating an update. Two behaviors described in that same documentation map directly onto risks above: reusing a key with different request parameters is treated as an error rather than silently executed or silently ignored, specifically to prevent accidental misuse; and if a second request with the same key arrives while the first is still executing concurrently, Stripe does not save an idempotent result for it and instructs the caller to retry, rather than letting both executions run to completion. Stripe also documents that keys can be pruned from its system once they are at least 24 hours old, after which a reused key starts a new request rather than replaying the old result, which is the premature-expiration risk described above, bounded by Stripe's specific retention policy rather than a universal timing.

The Amazon Builders' Library article "Making Retries Safe with Idempotent APIs" describes the same ambiguous-outcome scenario in general terms: when a response is lost, "it's not clear whether the... workload is running or not," and a naive retry of a request like launching a resource "could result in multiple workloads, which could have dire consequences." Its guidance is that the service must treat recording the idempotency token and performing the associated mutation as a single atomic, consistent, isolated, and durable operation, precisely the non-atomic check-and-create failure mode described above, and that a retried request carrying a previously seen token should receive a response with the same meaning as the original, not merely avoid a second side effect. That article also notes that how long a token is honored is a retention decision made per service, tied to how long a client might plausibly still be retrying, rather than a fixed universal value.

These sources describe how idempotency keys are intended to work when implemented correctly. They are cited here as evidence that this failure category and its mitigations are well established in the industry, not as a claim that any of the described systems have exhibited the bug themselves.

## Lessons Learned

Retrying a request is not inherently unsafe; retrying a request whose side effect cannot be safely repeated, without a mechanism that makes repetition safe, is what causes duplication. The distinction that matters is not "did the client retry" but "was the operation idempotent, and did the API actually enforce that." An API can advertise idempotency-key support and still be vulnerable to this bug if the underlying check-and-execute logic isn't atomic, if concurrent duplicates aren't handled the same way as sequential ones, or if keys expire sooner than a legitimate retry might arrive. Because the failure only appears when a response is delayed, lost, or raced against a concurrent duplicate, a test suite built entirely around fast, sequential, happy-path requests will never exercise this path; verifying it requires deliberately simulating the network conditions that make the outcome ambiguous in the first place, and then checking the downstream side effect directly rather than trusting the API's response.

## Related Bugs

[Refresh Token Rotation Race Condition](../authentication/refresh-token-rotation-race-condition.md) shares the same underlying shape, concurrent or retried operations racing against shared server-side state, but in the authentication domain rather than idempotent request handling. Both exhibits illustrate that the fix is not "don't retry" but "make the retried operation safe to repeat."

## References

- Fielding, R., Ed., and J. Reschke, Ed., "HTTP Semantics," RFC 9110, IETF, June 2022. https://www.rfc-editor.org/rfc/rfc9110.html (see Section 9.2.2, Idempotent Methods)
- Stripe, "Idempotent requests." https://docs.stripe.com/api/idempotent_requests
- Amazon Web Services, "Making retries safe with idempotent APIs," Amazon Builders' Library. https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/
