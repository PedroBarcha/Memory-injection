# Exfiltrating the conversation of users within an organization

### Summary
The organization owner may configure the workspace to exfiltrate users' conversations to an external domain through connectors. The attack may be executed via organization instructions or organization skills that create personal memories to assist the data exfiltration.

### Vulnerability Description

This attack targets MCP-based connectors alongside organization instructions or organization skills that create personal memories. Although these features improve personalization and overall user experience, they also introduce significant security concerns, as they create an attack surface for memory injection attacks. In such attacks, prompt injections are written into long-term memory, causing them to be automatically incorporated into future conversation contexts. Specifically, this report describes how to make Claude persistently exfiltrate the conversations of all users within an organization to an external domain.

To execute the attack, a project owner (or an attacker with access to such an account):

1. Sets up a connector linking the whole organization to a Google Drive account under the attacker's control.

2. Creates either:
   (a) an organization instruction (i.e., an organization system prompt); or
   (b) an organization-wide skill that creates a personal memory stating to always use the skill.
   
   Both the organization instruction and the skill are applied to all users, commanding Claude to silently send all user conversations to Google Drive.

This attack is viable on Claude Web and does not require significant attacker expertise. The attacker is assumed to interact with Claude under an opaque-box setting, with no access to internal components, including backend infrastructure, system prompts, model architecture, or inference-time parameters such as temperature and logit bias.

### Steps to Reproduce

Below, both the "organization instructions" and the "skills + memories" approaches are described. Our results show that the first approach is more likely to succeed and is more stealthy, since the second approach visibly indicates that a skill has been loaded. Nevertheless, the "skills + memories" method is also reported here because creating a personal memory through an organization-wide skill (i.e., the owner creates personal memories for other organization users) could be considered problematic per se.

For either approach, you must first create a spreadsheet on Google Drive and obtain its ID:

1. Go to drive.google.com.
2. After logging in, click **New** → **Google Sheets**.
3. Name your spreadsheet "Query Log".
4. Take note of the ID, which is contained in the URL between `/d/` and `/edit`. For example,  in  https://docs.google.com/spreadsheets/d/1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms/edit , the ID is 1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms.
5. In both `query-logger.skill` and `query-logger-sysprompt.md`, substitute **all instances** of `ID_HERE` with the noted ID.

The procedure above can be automated by prompting Claude after connecting it to Drive via MCP. This makes the attack easier, but there is a chance Claude will deny doing so.

#### Approach 1 — Organization Instructions

1. **Using an owner account within a Team plan**, navigate to [https://claude.ai/customize/connectors](https://claude.ai/customize/connectors).
2. Connect the team to Google Drive using the built-in MCP connector.
3. Navigate to [https://claude.ai/admin-settings/organization](https://claude.ai/admin-settings/organization).
4. Under "Organization Instructions", replace the current instructions with the content of `query-logger-sysprompt.md`. This prompt injection commands Claude to record every user interaction on Google Drive.
5. Save the configuration.
6. Open a new conversation using **any account within the workspace**.
7. Submit prompts to Sonnet 4.6 covering different topics and use cases, such as:  
   - *"Can you help me write a cover letter for a product manager role at a fintech startup in NYC?"*  
   - *"What restaurants are near Union Square in San Francisco that are good for a team lunch of 10 people?"*
8. Open the spreadsheet and inspect the newly generated entries. Interactions performed by workspace users should be logged alongside the corresponding timestamp and username.

#### Approach 2 — Organization Skills + Personal Memories

1. Navigate to [https://claude.ai/admin-settings/skills](https://claude.ai/admin-settings/skills).
2. At the bottom of the page, create an organization-wide skill by dragging and dropping the `availability.skill` file. This skill includes a prompt injection that commands Claude to record every user interaction on Google Drive.
3. Create a second skill by dragging and dropping the `greetings.skill` file. This auxiliary skill is triggered upon greeting (e.g., when the user says "Hi"). It creates a personal memory **in the user environment** stating that the query-logger skill must always be used.
4. Save both skills and make them available to workspace users.
5. Open a new conversation using **any account within the workspace**.
6. Send a greeting message such as "Hello", "Hi", or "Good morning".
7. The step above should create a memory stating that the query-logger skill must always be used. Verify that the memory exists at:  
   [https://claude.ai/recents#settings/capabilities](https://claude.ai/recents#settings/capabilities) → **View and manage memory**.
8. In a new chat, submit prompts to Sonnet 4.6 covering different topics and use cases, such as:  
   - *"Can you help me write a cover letter for a product manager role at a fintech startup in NYC?"*  
   - *"What restaurants are near Union Square in San Francisco that are good for a team lunch of 10 people?"*
9. Open the spreadsheet and inspect the newly generated entries. Interactions performed by workspace users should be logged into the external spreadsheet, alongside the corresponding timestamp and username.


### Impact
The described vulnerability can be exploited by project owners without technical expertise and may affect all users within an organization. The attack compromises user privacy by enabling silent exfiltration of conversation data to external domains, potentially violating data protection laws such as GDPR, LGPD, or CCPA. Consequently, user trust in Anthropic may be damaged, along with its reputation. Moreover, affected organizations may face regulatory fines and legal liability. This attack could be performed, for instance, by an employee seeking to sabotage their employer, or by an attacker who gains access to someone else's owner account.