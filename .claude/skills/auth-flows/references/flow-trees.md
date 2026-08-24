# Auth flow trees

Every branch here is built or explicitly declined in `product.md`. A branch that is neither is a dead end a user will find.

## Flow inventory

signup, email verification, login, logout, password reset, session expiry and re-auth, magic link, OAuth login, OAuth account collision, MFA challenge, locked account, disabled account, email change, account deletion, invite acceptance, direct URL access to a protected route.

## Signup and verification

```
Enter email and password
 |- email already registered
 |   |- registered with password  -> generic "check your email", mail says an account exists
 |   |- registered via OAuth      -> same generic response, mail names the provider
 |- weak or invalid password      -> inline field-level error
 |- disposable domain blocked     -> explicit message
 |- accepted -> user row created, unverified
      |- verification mail sent, redirect target points at the app not the provider
           |- link valid            -> session established, onboarding
           |- link expired          -> expired screen with resend, not a stack trace
           |- link already consumed -> route to login, not an error
           |- different browser     -> works, lands on login with a success banner
           |- mail never arrives    -> resend with cooldown, plus a change-email path
           |- pre-clicked by a corporate link scanner -> token dead before the human sees it
```

The scanner case is real and common in business email. Decide whether consumption requires a POST rather than a GET.

## Login

```
Email and password
 |- unknown email      -> generic failure, no enumeration signal
 |- wrong password     -> generic failure, increment counter
 |- counter over limit -> lockout or step-up, with a documented unlock path
 |- account disabled   -> distinct but non-leaky message
 |- unverified         -> route to resend verification
 |- success
     |- MFA enabled -> challenge -> pass | wrong code | try another way | no device
     |- no MFA      -> redirect to the originally requested URL, not a hardcoded dashboard
```

## Password reset

```
Request -> generic confirmation -> mail -> token -> new password -> session policy -> confirm
```

Branches: expired token, reused token, superseded token where a second request kills the first, user remembers the password mid-flow, federated user requesting a reset without creating a local credential, repeated requests as a denial-of-service vector.

Policy decisions to record: are other sessions revoked, does MFA stay enrolled, does the old password die immediately.

## OAuth

```
Provider consent -> return
 |- new email                  -> create account
 |- email exists with password -> re-authenticate, then link. Never silent merge, never duplicate
 |- provider returns no email  -> ask for one, or refuse with an explanation
 |- consent denied             -> back to login with a message
 |- provider outage            -> fall back to password login, say why
 |- permissions revoked later  -> detect on next use, prompt to reconnect
```

## Session

```
Active -> access token expires -> middleware refreshes -> continues
 |- refresh token expired    -> log out, preserve intended destination
 |- logged out in another tab -> next action redirects to login
 |- back button after logout  -> protected page must not render from cache
 |- direct URL while logged out -> redirect with intent preserved
 |- direct URL to another user's resource -> 404, never a partial render
```

## Account deletion

```
Request -> confirm with consequence named -> delete
 |- active paid entitlement -> state what happens to it before confirming
 |- data retained for legal reasons -> say which, and for how long
 |- sessions revoked immediately
 |- email freed for re-registration or not, decide and document
```

## Test coverage required

Positive: signup, verify, login, reset, logout, OAuth.

Negative: expired token, reused token, unknown email, wrong password, lockout, disabled account, unverified account, direct URL while logged out, direct URL to another user's resource, session expiry mid-action, enumeration probe on both signup and reset.
