# A user's app approval that survived the password reset

## Scenario

The ticket in this live multi-user Azure training tenant was a standard user in accounting who was phished, completed multi-factor authentication, and had that signed-in session stolen. I reconstructed how the session became a durable OAuth consent-phishing path across a legacy connector app and a second app the attacker registered, where a person who is already signed in is led to approve an application. The question on the ticket was how the access survived a normal containment playbook (case name: The Stolen Identity).

## Environment

- Microsoft Entra ID, the sign-in directory, in a live multi-user Azure training tenant. This write-up is that training exercise.
- App registrations.
- Reader on directory apps.

## Investigation

I followed the two apps in five stages.

1. Entry. The directory keeps a listing for each application it allows to sign in and call the organization's services, called an app registration, and an Owner of that listing can change the application's credentials. I opened the legacy connector app in the portal and read the notes on Branding & properties. The note recorded the entry. The accounting user had been phished and had completed multi-factor authentication, so the stolen session already included proof that the check had succeeded, and Conditional Access, the rules that allow a sign-in when those conditions are met, let it through. That user was still an Owner on the legacy connector app. The assignment had never been removed, which is access drift. The note stays out of this write-up.

![The legacy app's properties page, where the investigation note was recorded. The note itself is blurred.](images/01-branding.png)

*The legacy app's properties page, where the investigation note was recorded. The note itself is blurred.*

2. Escalate. I opened Certificates & secrets. An application has its own password, called a client secret, and anyone who holds it can sign in as the application. Under Client secrets, a new one was on the list. The Expires column was the signal: the date sat nearly a century out, which marks a credential planted for persistence. The text written on that secret stays out of this write-up. The application then acts under its own directory identity, called a service principal, and that sign-in receives the permissions already granted to the application. Owner on the legacy app was enough for the attacker to authenticate that way and continue without another sign-in as the accounting user.

![The app's password list. The expiry date, 12/31/2099, is the problem.](images/02-client-secret.png)

*The app's password list. The expiry date, 12/31/2099, is the problem.*

3. Pivot. Each action the application may take against directory and user data is granted on its own, called an API permission, and an attacker signed in as the application can use every action an administrator has already approved. I read that list on the legacy app. Those actions ran through Microsoft Graph, the interface for directory and user data. They belonged to the application itself, and an administrator had already approved them, recorded as admin consent, so they were live. That list was the blast radius: how far a sign-in as the application could reach. Owners also listed a service principal for a second app the attacker had registered. The default directory setting lets a standard user register an app, so that second app was created without an administrator. Ownership outlives any one secret. After the first secret is rotated, the Owner can create another.

![The only owner is another application, not a person. That is what lets a deleted password be replaced.](images/03-owners.png)

*The only owner is another application, not a person. That is what lets a deleted password be replaced.*

4. Persist. I opened Expose an API on the legacy app. A named action another application is allowed to request, called a scope, was published there, and it lets the attacker's app ask the legacy app to perform that action for a signed-in user. The scope's name stays out of this write-up. Rotating the client secret leaves this request available. The attacker asks for that delegated access, access exercised as the user, from their own app.

![A custom permission published on the legacy app, which lets another app request access. The permission's user-facing name is blurred.](images/04-expose-api.png)

*A custom permission published on the legacy app, which lets another app request access. The permission's user-facing name is blurred.*

5. Loot. I opened Authentication on the attacker-created app. The address the sign-in system sends the person to after they accept, carrying the authorization code that proves they accepted, is called a redirect URI, and the party that controls the address receives that code. One registered address was the kind used for local development. Another pointed at attacker-controlled infrastructure. The attacker-created app's client id, its public identifier, together with that redirect URI and the exposed scope, is what a consent-phishing URL is built from. A person already signed in on a company device accepts the consent prompt, and the authorization code is sent to that redirect. I confirmed the suspicious redirect by reading it. A URL-encoded value had to be decoded before I could read it. The decoded value and the address stay out of this write-up.

Phishing the password again has to get past device compliance, location, and multi-factor authentication. Consent phishing starts from a person who is already signed in on a trusted device and asks them to approve an application. The directory keeps that approval as a standing record that the application may act for the user, called an OAuth grant (OAuth2PermissionGrant), and a password reset, a session revocation, or enforcing multi-factor authentication leaves the attacker that record. A trusted service that carries out a request it should never have authorized is called a confused deputy, and that let the attacker send the legacy connector app a request the app should have refused. The token from that grant lets the attacker act as the user against that backend. The broad permissions that belong to the application itself stay on the application sign-in.

## What broke / what surprised me

A standard user can register an app under the default directory setting, with no administrator in the path. Owner on an app registration is real privilege. A review aimed at Global Administrators would miss it, because Owner is recorded on the application. A single new secret looks like the whole incident until the Owners list is opened.

## Findings and recommendations

Determined: a five-stage consent-phishing chain on the legacy connector app. A standard user in accounting was phished, completed multi-factor authentication, and was still an Owner through access drift. A client secret expiring nearly a century out had been added. Microsoft Graph application permissions had admin consent. A service principal for the attacker-created app was an Owner. A custom scope was exposed. The attacker-created app had a redirect URI on attacker-controlled infrastructure, beside one that looked like local development.

Recommendations to the resource owner:

- Revoke the client secret on the legacy connector app.
- Remove the rogue service principal from Owners.
- Delete the custom exposed API scope.
- Revoke the OAuth2PermissionGrant explicitly. A password reset, session revocation, and enforcing multi-factor authentication leave the grant in place.
- Remove the attacker redirect URI.
- Review the Microsoft Graph application permissions and reduce them to what the connector still needs.
- Disable the default that lets a user register an application.
- Audit every app's Owners list the same way directory roles are audited.
- Alert on new client secrets and on new redirect URIs.

## What I learned

- A stolen user session and a client-credentials sign-in, the application signing in with its own secret, are different. The session is what passed Conditional Access. The secret is what let the attacker sign in as the service principal and use the application's permissions without signing in as that person again.
- The OAuth grant is why the access survived. Password reset, session revocation, and enforcing multi-factor authentication end the user's sessions. The grant has to be revoked on its own.
- What I'd do differently: check Owners and Expose an API before calling the secret rotation done. The new secret was the obvious artifact. The ownership and the custom scope are what still worked after that secret was gone.
- Reader on directory apps was enough to see the notes, the secret expiry, the permissions, the owners, the scope, and the redirect URIs.

<!--
**Suggested resume line:** Reconstructed a five-stage OAuth consent-phishing chain across two app registrations in a live multi-user Azure training tenant, from a stolen session that passed Conditional Access, through a long-lived secret on the legacy connector app, an ownership pivot, a custom exposed scope, and an attacker redirect, with remediation that revokes the OAuth grant explicitly.

**Interview angle:** How an attacker still has access after the password is reset, the sessions are revoked, and multi-factor authentication is enforced.
-->
