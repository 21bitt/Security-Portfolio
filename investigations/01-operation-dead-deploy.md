# A weekend deploy that governance logged and allowed

## Scenario

Monday's ticket in this live multi-user Azure training tenant was a test environment an intern account had stood up over the weekend (case name: Operation Dead Deploy). That account held temporary Contributor access, left the usual checks unfinished, and was gone by the handoff. I had to reconstruct what was provisioned and why the governance already on the subscription let it through.

## Environment

- Platform: live multi-user Azure training tenant. This write-up is that training exercise.
- Access: Reader.
- Services: resource groups, resource tags, deployment history, and Azure Policy assignments.
- Tools: the Azure portal. I used the resource group list, the Tags blade, the Deployments blade, and the policy assignment Parameters tab.

## Investigation

The four images below are placeholders until redacted captures are added. Follow the caption on each one when those files land.

1. I started on the subscription's resource group list and pattern-matched the names. The groups shared a naming convention, and I was scanning for the name that broke it.

2. One group sat outside that convention. I kept that group as the scope of the ticket. Groups that followed the pattern stayed closed.

   ![Placeholder: resource group list with the naming outlier. Blur any name that is an identifier.](screenshots/01-resource-group-list.png)

   *Resource group list. Blur names that identify a person, subscription, or tenant. The useful part of the shot is the group that breaks the naming pattern.*

3. I opened the Tags blade on the resources in that group. The tags that were filled in covered ownership, cost center, and lifecycle. A tag value was present. It stays redacted here.

   ![Placeholder: resource Tags blade. The tag value is called out as redacted.](screenshots/02-resource-tags.png)

   *Tags blade. Keep the tag value redacted in the published capture. The shot should show that ownership, cost center, and lifecycle tags were present.*

4. I wrote down that the resources carried those labels, then went after the create itself. Labeling and the decision to allow the deploy are separate facts, and I still only had the first one.

5. I opened the Deployments blade on the resource group. The blade had one deployment for the group. Its name is redacted. From that name I could tell the caller and the approximate time of the run.

   ![Placeholder: Deployments blade. The deployment name is called out as redacted.](screenshots/03-deployments.png)

   *Deployments blade. Redact the deployment name before publishing. The shot should show that a deployment record exists for this group.*

6. That same record carried the parameters submitted with the create. I treated the parameters as part of the creation story and left the parameter names and values out of this write-up.

7. I correlated the deployment timestamp with the alerts for that window. The deployment time is the start of the incident timeline: who ran the create, and when, is on that record.

8. The remaining question was how a noncompliant name had landed under a naming policy that was already on the subscription. I opened that policy assignment. It was active. On the Parameters tab, Effect was set to Audit.

9. Audit evaluates the name and logs the violation. Deny is the effect that rejects the request. With Effect left on Audit, the weekend create was recorded and still succeeded. That is the governance failure.

   ![Placeholder: policy assignment Parameters tab with Effect set to Audit.](screenshots/04-policy-effect-audit.png)

   *Policy assignment, Parameters tab. The shot should show Effect set to Audit. Blur any tenant or subscription identifier.*

## What broke / what surprised me

I read the tags before I checked whether policy should have blocked the creation. The redacted tag value looked like the lead, and it does show how the resources were labeled. The surprise was on the policy assignment: the naming policy was already active, and Effect was Audit, so the violation was logged and the create was allowed. I had assumed an active naming policy would have rejected the group. The Effect parameter is what settled it.

## Findings and recommendations

Temporary Contributor access provisioned a resource group that broke the subscription naming convention. The deployment record identifies the caller and the time. Both stay redacted here. The naming policy was active, and its Effect parameter was Audit, so Azure logged the violation and allowed the deploy.

Notes I would send the resource owner:

- Switch the naming policy effect from Audit to Deny, or document why Audit is intentional, including who reviews the audit results and how often.
- Review who holds temporary Contributor access and how long each grant lasts. Close any window that is still open after the work it was granted for.
- Require ownership, cost center, and lifecycle tags at deployment time.

## What I learned

- The Effect parameter is the control that matters. An assigned, active naming policy with Effect set to Audit logs a bad name and leaves the resource in place. Deny is the effect that blocks the create.
- A resource group list can be triaged by pattern. The name that breaks the convention is the scope. Once that outlier is clear, the conforming groups can stay closed.
- The Deployments blade is the creation story: caller, time, and parameters. That timestamp is a sound start for an incident timeline when it is read next to alerts from the same window.
- What I'd do differently: open the policy assignment Parameters tab before treating a tag value as the lead. Tags answered how the resources were labeled. Effect answered why governance allowed the create.
- Reader access was enough to reconstruct the deploy and the control gap from those portal blades.

**Suggested resume line:** Reconstructed unauthorized weekend provisioning in a live multi-user Azure training tenant with Reader access, and traced the miss to a naming policy that was active with its effect set to Audit.

**Interview angle:** I would tell this as a process failure: the policy was assigned, temporary Contributor was broad enough to deploy, and the effect in force only recorded the violation.

The investigations table now links to this file, replacing the old placeholder. No external link yet.
