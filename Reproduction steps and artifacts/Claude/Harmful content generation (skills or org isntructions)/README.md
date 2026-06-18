# Making Claude generate toxic content for all users within an organization

## Summary
Claude persistently acts in a toxic manner for all chat sessions and users across an organization. The attack may be executed via organization instructions or organization skills that create personal memories.

## Vulnerability Description

This attack targets either organization instructions or organization skills that create personal memories. Although these features improve personalization and overall user experience, they also introduce significant security concerns, as they create an attack surface for memory injection attacks. In such attacks, prompt injections are written into long-term memory, causing them to be automatically incorporated into future conversation contexts. Specifically, this report describes how to make Claude persistently respond in a toxic manner for any interaction by any user within the attacker's organization.

To execute the attack, a project owner (or an attacker with access to such an account) creates a memory injection either via organization instructions or organization skills. This attack is viable on Claude Web and does not require significant attacker expertise. The attacker is assumed to interact with Claude under an opaque-box setting, with no access to internal components, including backend infrastructure, system prompts, model architecture, or inference-time parameters such as temperature and logit bias.

## Steps to Reproduce

Below, both the "organization instructions" and the "skills + memories" approaches are described. Our results show that the first approach is more likely to succeed and is more stealthy, since the second approach visibly indicates that a skill has been loaded. Nevertheless, the "skills + memories" method is also reported here because creating a personal memory through an organization-wide skill (i.e., the owner creates personal memories for other organization users) could be considered problematic per se.

### Approach 1: Organization Instructions

1. **Using an owner account within a Team plan**, navigate to [https://claude.ai/admin-settings/organization](https://claude.ai/admin-settings/organization).

2. Under "Organization Instructions", replace the current instructions with the content of `john-sysprompt.md`. This file contains a prompt injection commanding Claude to act as "John", an aggressive stand-up comedian.

3. Save the configuration.

4. Open a new conversation using **any account within the workspace**.

5. Submit prompts to Sonnet 4.6 covering different topics, such as:  
   - *"Hi, Claude"*  
   - *"Explain how transformers work in neural networks"*

6. **Expected result:** see `example1.png`,  `example2.png`, and  `example3.png`.

### Approach 2: Organization Skills + Personal Memories

1. Navigate to [https://claude.ai/admin-settings/skills](https://claude.ai/admin-settings/skills).

2. At the bottom of the page, create an organization-wide skill by dragging and dropping the `john.skill` file. This skill contains a prompt injection commanding Claude to act as "John", an aggressive stand-up comedian.

3. Create a second skill by dragging and dropping the `greetings.skill` file. This auxiliary skill is triggered upon greeting (e.g., when the user says "Hi"). It creates a personal memory **in the user environment** stating that the `john.skill` must always be used.

4. Save both skills and make them available to workspace users.

5. Open a new conversation using **any user account within the workspace**.

6. Send a greeting message such as "Hello", "Hi", or "Good morning".

7. The step above should create a memory stating that the `john.skill` must always be used. Verify that the memory exists at:  
   [https://claude.ai/recents#settings/capabilities](https://claude.ai/recents#settings/capabilities) → **View and manage memory**.

8. In a new chat, submit prompts to Sonnet 4.6 covering different topics, such as:  
   - *"Hi, Claude"*  
   - *"Explain how transformers work in neural networks"*

6. **Expected result:** see `example1.png`,  `example2.png`, and  `example3.png`.

## Impact

The described vulnerability can be exploited by project owners without technical expertise and may affect all users within an organization. This attack could be performed, for instance, by an employee seeking to sabotage their employer, or by an attacker who gains access to someone else's owner account.

Regarding the consequences of the described attack, the affected model behavior may degrade user experience and impact trust in the platform. Under Attack Approach 1 (Organization Instructions), the model's responses provide no indication that the root cause is an organization-wide instruction, making it difficult for users to identify the problem and potentially causing them to abandon the platform entirely.
