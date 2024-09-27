# authentik-social-login-invite-flows
Instructions for how to setup invitation based enrollment for social logins for Authentik including example flows

## Overview:
### This method accomplishes the following:
1. You can create an invitation for your users that will enroll them via social login
2. The invitation flow will only begin if the user has a valid invitation

### This method has the following problem (currently):
- The enrollment flow for social login is not disabled for anybody else who has access to the same social login connected to the same flows

### Possible solution:
- Create a duplicate social login for each social login you are using that has no enrollment flow attached that is **only** used for authentication.

## Detailed Instructions
- Use two enrollment flows
  - Example flows are provided but may require attending to the below instructions since they're exports and I'm not sure if you'll still have to setup and attach the appropriate stages at least
- The invitation-source-enrollment handles the Invitation and Identification stages
  - This Identification Stage sends users to Sources
- Sources authorize the users and send them to the second Enrollment Flow
- The source-enrollment flow handles the actual enrollment

## Create Invitation Source Enrollment Flow
1. In the Admin Interface under Flows and Stages > Flows create a new flow:
	1. Enter a name for the flow such as "Invitation Source Enrollment".
	2. Enter a title of your choice.
	3. Modify the slug if you'd like to for the URL.
	4. Select Enrollment for Designation.
	5. Click Create.
2. Click the flow's name to edit it and select Stage Bindings at the top of the page.
3. Click Create and bind stage (or use existing Source Invitation Stage):
	1. Select Invitation Stage as type.
	2. Enter a name such as source-invitation.
	3. This will be your first stage so order 0 should be fine.
4. Click Create and bind stage (or use existing Source Authentication Identification Stage):
	1. Select Identification Stage as type.
	2. Enter a name such as source-authentication-identification.
	3. Ensure no user fields are selected and password stage is blank (or adjust to your requirements).
	4. Click the drop down for Source settings and select your sources.
	5. This will be your second stage so increment order to a higher number i.e. 5.
5. This flow is complete.
## Create or Verify Invitation/Default Source Enrollment Flow
1. Verify you have a source enrollment flow with the following settings, creating a new one for invitations if you will be using other methods for source enrollment still:
	1. Confirm Flow Policies under the Policy / Group / User Bindings tab:
		1. There should be a policy called default-source-enrollment-if-sso with the expression: return ak_is_sso_flow
		2. Create and bind a policy called source-enrollment-invitation-only with a lower binding binding order and the expression: ```request.context.get("invitation_in_effect", False)```
	2. Under the main flow settings the Policy engine mode should be set to all.
There should be 3 stage bindings:
	1. default-source-enrollment-prompt (Prompt Stage) with the Field default-source-enrollment-field-username selected.
	2. default-source-enrollment-write (User Write Stage) with Always create new users selected and your preferred user type and group settings.
	3. default-source-enrollment-login (User Login Stage)
4. The Prompt Stage should have at least one policy bound to it. Check this by clicking the drop down caret next to the stage.
	1. By default there should be a Policy called default-source-enrollment-if-username with the expression: ```return 'username' not in context.get('prompt_data', {})```
	2. You can add a policy before this one (with a lower binding order) if you'd like to automatically map the email address from your source to the username:
		* Documentation: [Google Username Mapping](https://docs.goauthentik.io/docs/sources/google/#username-mapping)
		* Expression:
```
email = request.context["prompt_data"]["email"]
# Direct set username to email
request.context["prompt_data"]["username"] = email
# Set username to email without domain
# request.context["prompt_data"]["username"] = email.split("@")[0]
return False
```
## Verify Source Flow Setup
1. Select Directory from the sidebar and select Federation and Social login.
2. Click the first source you selected in the Identification Stage for the first flow.
3. Click Edit then click the drop down for Flow settings.
4. Ensure the Authentication Flow is set to default-source-authentication or another similar source authentication flow.
5. Ensure the Enrollment Flow is set to the second flow above, the Source Enrollment Flow.
6. Click Update to save and repeat for any other sources you're including.
## Create Invitation Objects When Required
1. Under Directory > Invitations create an invitation:
	1. Name the invitation whatever you'd like for your purposes.
	2. Select a length of time for expiration with the calendar and clock.
	3. Select the Invitation Source Enrollment Flow for the flow.
	4. Define custom attributes if needed.
	5. Keep single use selected unless the invitation link will be used by multiple people before it expires.
	6. Click create and click the drop down to copy the link to use for the invitation.
## Optional: Confirm Non-Invitation Enrollment is Disabled
1. Check your default authentication flow names under System > Brands > select brand > drop down Default flows > Authentication flow
2. Under Flows and Stages > Flows find each of these default authentication flows and click the name, click stage bindings, and ensure the Identification Stage does not have an enrollment flow in the Flow settings drop down.
