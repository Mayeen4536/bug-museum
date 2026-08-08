---
title: Refresh Token Rotation Race Condition
slug: refresh-token-rotation-race-condition
category: authentication
tags: [oauth, refresh-tokens, concurrency, race-condition]
difficulty: intermediate
production_impact: high
root_cause_type: race-condition
detection_method: concurrency-testing
source_type: sourced-example
status: draft
---

# Refresh Token Rotation Race Condition

## The Problem

Authentication systems that use rotating refresh tokens issue a new refresh token every time the old one is exchanged for a new access token, and invalidate the one that was just used. This works well when exactly one request performs that exchange at a time. It breaks down when a client application allows more than one part of itself, such as several in-flight API calls, multiple tabs, or retry logic, to attempt the exchange independently. With strict single-use rotation and no overlap or leeway mechanism, only the first exchange to reach the authorization server succeeds, and every other concurrent attempt is left holding a refresh token that has already been retired; providers that configure an overlap or leeway window can tolerate more of this concurrency before the failure appears.

## Symptoms

Depending on the authorization server's configuration, a client may see any of the following: a user is signed out without having taken any action that would explain it; an in-progress session degrades into repeated `invalid_grant` errors on subsequent requests; some API calls succeed while others fail within the same short window; or the application prompts for reauthentication that appears unrelated to anything the user did. None of these symptoms individually points at concurrency, since each on its own is indistinguishable from a token that simply expired or an ordinary network failure. This is a general failure mode of rotating refresh tokens, not a description of any single vendor's behavior; the exact error surface depends on the implementation and its rotation and reuse policy.

## Why It Happens

Refresh token rotation turns the refresh token into mutable state shared by every part of a client capable of triggering a refresh, instead of a stable, reusable credential. Each successful exchange advances that state to a new generation and retires the previous one. If two callers read the same generation of that state and act on it independently, at most one of them can successfully advance it; the other has fallen behind, and its now-superseded refresh token cannot be exchanged in a system that treats reuse as a signal of possible compromise. This is a variation on a familiar concurrency problem, the same class as two writers racing to update a database row or a cache key, applied to authentication state instead of application data. Nothing about OAuth introduces this risk on its own; rotation only surfaces it, because rotation is the mechanism that makes each refresh token single-use.

## Root Cause

The concrete fault is that multiple refresh operations were allowed to race against the same rotating credential without coordination on the client side. This typically happens when a client does not serialize its own token refresh path: for example, an HTTP client that refreshes independently per outgoing request rather than sharing a single in-flight refresh operation across concurrent callers, or application code that responds to a failed request by triggering a fresh refresh call without checking whether a refresh is already underway. Because the authorization server enforces one-time use of each rotated refresh token, the first exchange to arrive succeeds, and every other concurrent attempt using the same, now-stale, token fails.

## User Impact

The visible effect is disruption during otherwise ordinary use rather than a data-correctness problem: a session ends and the user has to sign in again, sometimes mid-action, such as submitting a form or progressing through a multi-step flow, and any unsaved state in that flow can be lost. In applications that issue several API calls in parallel, such as a single-page application loading multiple resources on navigation, the failure is more likely to surface right after a burst of activity that lines up with the access token's expiry, since that is when the most refresh attempts are competing to happen at once. Not every implementation produces a hard logout; some recover silently by re-authenticating in the background, depending on how the client is written.

## Developer Investigation

The most useful evidence is timing and sequence, not the tokens themselves. Correlating request or trace IDs across a client's concurrent calls, alongside timestamps of every refresh attempt, usually shows more than one refresh request in flight for the same session within a small window. Authentication SDK or authorization server logs typically record the outcome of each attempt, success or a rejection such as `invalid_grant`, and in many implementations an indication that reuse of a previously issued refresh token was detected; that is a strong signal to look for. It is enough to know that a refresh happened and which generation of the token it produced, not what the token's value was. Raw access or refresh token values should never appear in logs.

Retrying a failed request as a first response to this kind of failure can make the investigation harder rather than easier. A naive retry often triggers another independent refresh attempt, which can either race against the one that just failed or arrive after the authorization server has already revoked the whole token family in response to the first reuse, so the retry fails for a different reason than the original request did. Viewed later, without correlated timestamps and request IDs across every attempt, this looks like unrelated, repeated flakiness instead of one race with a single, identifiable cause.

## QA Detection Strategy

This failure does not show up in sequential, happy-path authentication testing, one login, one refresh, one logout, because that path never gives two refresh attempts a reason to compete. Exposing it deliberately means constructing concurrency around the token expiry boundary: firing several protected requests at nearly the same moment an access token is expected to expire, using a shortened token lifetime in a test environment so the boundary can be reached without waiting out a real expiry window, and adding controlled network delay or jitter around the refresh call to widen the window in which two attempts can overlap. Because the outcome depends on timing, a single passing run is not sufficient evidence that the path is safe; the same scenario should be run repeatedly to see whether the race reproduces at all, and how often.

## Automation Strategy

An automated test for this needs to orchestrate the race rather than avoid it, then assert on outcomes and invariants instead of a specific interleaving. Concretely: drive several concurrent operations that each require a valid access token right at or after expiry, let the client's real refresh logic run without mocking it out, and check afterward that the session ends up in a consistent state, either maintained under a valid current token or cleanly re-authenticated, with no caller left holding a permanently broken session while others succeeded. The test should not depend on sleep calls to keep requests apart; deliberately overlapping them is the point.

```
for each of N concurrent callers:
    call protected_endpoint() using the client's normal request path
assert: every caller ends in {success, clean re-auth}, none stuck on invalid_grant
assert: at most one refresh exchange succeeded per token generation
```

This stays valid regardless of the stack underneath it; the specific client, SDK, or authorization server does not change what the test is checking for.

## Real-World Pattern

This is a documented, recurring characteristic of rotating refresh tokens, not an isolated defect. RFC 9700, the IETF's current best-practice guidance for OAuth 2.0 security, requires authorization servers to detect refresh token replay for public clients using either sender-constrained refresh tokens or refresh token rotation. Under rotation, the previous refresh token is invalidated with each exchange; if an already-invalidated refresh token is later presented, the authorization server cannot tell whether the legitimate client or an attacker submitted it, so it revokes the active refresh token, forcing the legitimate client to obtain a fresh authorization grant. RFC 9700 does not define an overlap or leeway window for concurrent legitimate requests, and does not speak to that scenario.

Auth0's documentation describes the same shape of problem from an implementation angle, and is the source for the overlap-window and family-revocation behavior discussed elsewhere in this exhibit: rotation invalidates the previous refresh token as soon as a new pair is issued, automatic reuse detection revokes the affected refresh-token family and forces re-authentication when an already-invalidated token is presented, and a configurable rotation overlap period exists specifically to tolerate refresh attempts that land within a short window of each other without treating that overlap as a breach. This material is cited here as evidence that the failure mode described in this exhibit is a known, structural consequence of rotating refresh tokens across implementations, not as a report of an incident at Auth0 or any other vendor. Exact behavior, including whether an overlap period exists, how long it is, and how broadly a detected breach is revoked, varies by authorization server, SDK, and configuration, and should be confirmed against the specific provider in use before drawing conclusions about any given system.

## Lessons Learned

Authentication state is shared mutable state once rotation is involved, and deserves the same concurrency discipline as a database row or a cache entry, not the assumption that a token is safe to read and reuse from multiple places at once. The security mechanism responsible for exposing this failure, reuse detection, exists to catch stolen tokens; it has no way to distinguish an attacker replaying a token from a client's own uncoordinated code racing itself, so an application-level concurrency bug can surface as a security event instead of a straightforward functional failure. Because the failure is intermittent and timing-dependent, diagnosing it depends on reconstructing a timeline of correlated requests rather than inspecting any single failing call in isolation, which makes timestamped, correlated logging part of this system's testability rather than an optional nicety. A test suite that only exercises the sequential login-refresh-logout path will never encounter this class of bug at all, since the failure requires deliberately induced concurrency to appear.

## Related Bugs

No other exhibits exist in this repository yet. This section will link to related concurrency and session-management exhibits once they are added.

## References

- Lodderstedt, T., Bradley, J., Labunets, A., and D. Fett, "Best Current Practice for OAuth 2.0 Security," RFC 9700, IETF, January 2025. https://www.rfc-editor.org/rfc/rfc9700.html (see Sections 2.2.2 and 4.14)
- Auth0, "Refresh Token Rotation." https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation
- Auth0, "Configure Refresh Token Rotation." https://auth0.com/docs/secure/tokens/refresh-tokens/configure-refresh-token-rotation
