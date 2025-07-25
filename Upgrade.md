# ENVOY - Lightweight User Environment Manager

> [!IMPORTANT]
> If you are upgrading from version 1.1.* to 1.2.*, please be aware that this release introduces breaking changes if not followed correctly!

From version 1.2.* Envoy uses **Delegated Permissions** instead of Application permissions. It does not make use of a custom App Registration anymore but uses the public **Microsoft Graph Command Line Tools** Enterprise App. Therefor the app id and client secret are not stored locally on the device anymore! This is a huge step forward!


## Add required permissions to Microsoft Graph Command Line Tools
1. Go to Enterprise Applications
	- Azure Portal → Entra ID → Enterprise applications
	- Search for: Microsoft Graph Command Line Tools
2. Open the Enterprise App
	- Click into the "Microsoft Graph Command Line Tools"
	- Go to Security → Permissions
3. Add delegated permissions for all users in your tenant using this app
    - User.Read.All
    - Group.Read.All
    - GroupMember.Read.All
4. Grant Admin Consent


## Implementation/testing
1. Set required permissions on Microsoft Graph Command Line Tools (step above)
2. Install the newest Envoy.MSI installer (only on your test device, first)
3. Test authentication and execution. If authentication fails something might be blocked by permissions on Enterprise App **Microsoft Graph Command Line Tools** or Conditional Access policies. The sign-in logs can be found under User sign-ins (non-interactive).


## After migration:
1. Remove the previously created App Registration (shown in config.xml EntraID part). This is not required anymore for version 1.2.*
2. Remove the entire EntraID part from the config.xml (on all devices, or just the one used for central distribution)
