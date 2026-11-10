# A weekend test deploy that policy audited and allowed

## Scenario

Monday's ticket in this live multi-user Azure training tenant was a test environment an intern account had stood up over the weekend and left behind (case name: Operation Dead Deploy). That account had temporary Contributor access. With Reader access, I had to reconstruct what was created and why the active naming policy let it through.

## Environment

- Platform: live multi-user Azure training tenant. This write-up is that training exercise.
- Access: Reader.
- Services: resource groups, resource tags, deployment history, and Azure Policy assignments.
- Tools: the Azure portal. I used Resource groups, then the blades on the resource and the policy assignment: Tags, Deployments, and Parameters.

## Investigation

The four images below are placeholders until redacted captures are added. Follow the caption on each one when those files land.

1. I opened Resource groups in the Azure portal and pattern-matched the list against the organization's naming standard. That standard uses an `rg-` prefix.

2. One resource group broke the `rg-` prefix. I kept that group as the scope of the ticket.

   ![Placeholder: resource group list with the naming outlier. Blur any name that is an identifier.](screenshots/01-resource-group-list.png)

   *Resource group list. Blur names that identify a person, subscription, or tenant. The useful part of the shot is the group that breaks the organization's `rg-` naming standard.*

3. I opened the Tags blade on a resource in that group. A tag was present, and it was consistent with the metadata I expected for ownership, cost center, and lifecycle. The value stays redacted here.

   ![Placeholder: resource Tags blade. The tag value is called out as redacted.](screenshots/02-resource-tags.png)

   *Tags blade. Keep the tag value redacted in the published capture. The shot should show a tag consistent with ownership, cost center, and lifecycle.*

4. That tag showed the resource carried the expected metadata. I still needed the create itself, so I went to the deployment history on the same group.

5. I opened the Deployments blade for that resource group. There was one deployment. Its name is redacted. That name broke the organization's naming standard as well.

   ![Placeholder: Deployments blade. The deployment name is called out as redacted.](screenshots/03-deployments.png)

   *Deployments blade. Redact the deployment name before publishing. The shot should show the single deployment on this resource group.*

6. I used that deployment record as the start of the timeline. Deployment history is where the create sits in time. The name and the parameter values stay out of this write-up.

7. A naming policy already existed and was active. The assignment was on the subscription, so it covered this resource group. I opened the assignment's Parameters tab. Effect was set to Audit.

8. With Effect set to Audit, the policy wrote an audit result for the naming violation, and the resource was still created. Deny is the effect that rejects the request.

   ![Placeholder: policy assignment Parameters tab with Effect set to Audit.](screenshots/04-policy-effect-audit.png)

   *Policy assignment, Parameters tab. The shot should show Effect set to Audit. Blur any tenant or subscription identifier.*

## What broke / what surprised me

The policy audit result made the naming outlier visible. From there the case was still catchable in the portal: the resource group broke the `rg-` standard, and the one deployment on that group broke the naming standard too.

The sharper surprise was that an active policy did not stop the create. I had the broken resource name in front of me, and the naming policy was on. Effect was Audit, so the control recorded the violation and the resource remained.

## Findings and recommendations

This was an unauthorized-style test deploy over the weekend, and it was shadow IT: an account with temporary Contributor access stood up a test environment outside the organization's naming standard. The resource group name broke the `rg-` prefix, and the single deployment name on that group broke the naming standard as well. A tag was present and consistent with ownership, cost center, and lifecycle. The naming policy was active and assigned on the subscription, which covered this group. Effect was Audit, so the policy produced an audit result and did not deny the create.

Recommendations to the resource owner:

- Set the naming policy Effect to Deny, or document why Audit is intentional and who reviews the audit results.
- Review temporary Contributor access: who holds it, and how long each grant lasts.
- Require ownership, cost center, and lifecycle tags at deployment time.
- Train people who receive temporary Contributor on the naming standard before they use that access.
- Review the rest of the policy baseline for other policies whose Effect is Audit, where the assumption has been that those policies block.

## What I learned

- Deployment history is a timeline. The single deployment on the resource group is the create event I can put at the start.
- Tags carry ownership, cost center, and lifecycle. On this resource a tag was present and consistent with that metadata. The value stays redacted.
- Audit and Deny are different controls. An active naming policy with Effect set to Audit writes an audit result and still allows the resource to be created. Deny is the effect that blocks the create.
- What I'd do differently: check the policy Effect earlier. I started from the resource name. Effect is what explained why that name was allowed to exist.
- Reader access in the Azure portal was enough to see the group, the tag, the deployment, and the policy parameters.

**Suggested resume line:** Reconstructed a weekend test deployment in a live multi-user Azure training tenant with Reader access, and showed that an active naming policy audited an `rg-` break because its Effect was Audit.

**Interview angle:** I would tell this as a process failure: temporary Contributor left in place, a naming standard those people still need to be trained on, and a policy effect that recorded the shadow IT.

The investigations table now links to this file, replacing the old placeholder. No external link yet.
