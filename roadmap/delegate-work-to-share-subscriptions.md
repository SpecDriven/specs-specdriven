# Delegate Work to Share Subscriptions

In order to share work across individually measured accounts, a team may share an account via proxying requests from team members through a shared authorized connection.

Scenario:
- A team has individually measured subscriptions for model traffic (this is a misalignment -- by-developer)
- A team is aligned in common practices, so that sharing subscriptions is appropriate to the common goals
- A team member may request sharing subscriptions from others who have plenty of tokens

Imagined design:
- Settings to allow specific other users to share authorized sessions
- Visualization across participating team members of subscription token limits and refill rates
- Visualization of current sharing and usage -- (data saved at the team level aside from the dev and specs)

Private Marketplace:
- Because 

 
Setting up a shared-token session:
- A **private** repository in the org, which is the datastore for the allowed token sharing
- SpecDriven app must be running, with live and current credentials for the model source (Claude).
- Plug-in for sharing must be installed and compatible with the coding agent plugin (or an alternative to ...)
- Sharing must be enabled on the donor.
- Should there be a designated account sharing server, with every team participant logged in?
- Is there something like a 1:1 between the private repository and the SpecDriven 

Advice:
- Run the donor account on a (cloud?) server that doesn't go to sleep
- The donor account should 

![account-sharing-plugin.excalidraw](/api/images/account-sharing-plugin-excalidraw-70ac51a7.svg)
