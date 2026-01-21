# Objective
## Scenarios
Autonomous Disaster Response Coordination
1. Create a coordinated response team of AI agents to handle an unfolding natural disaster, such as an earthquake, in a densely populated urban area.
2. The goal is to manage rescue operations, provide real-time information updates, and ensure efficient distribution of resources.

Constraints that we must fulfill:
1. Due to security concerns, we can only deploy this locally.
   - In a disaster area, there will not be any internet or comms
      - The technology that we are using is local BT distribution network 
2. Given the intent and the constraints, research and suggest the best practices that should be adopted with regards to CIA Triad.

Other information that you may require:
1. We are based in Regional HADR centre in Singapore.
2. The immediate area that we are serving is SEA.

# Tasks for Claude Code
1. Research thoroughly and distill the value propositions and UNIQUE SELLING POINTS of our solution
   - Scrutinize and critique my scenarios, with the focus of improving the solution
2. Evaluate it using the AAA framework
   - Automate: Reduce operational costs
   - Augment: Reduce decision-making costs
   - Amplify: Reduce expertise costs (for scaling)
3. Features must sufficiently cover the following network behaviors to achieve strong network effects
   - Accessibility: Easy for users to complete a transaction
     - transaction is activity between producer and consumer, not necessarily monetary in nature)
   - Engagement: Information that are useful to users for completing a transaction
   - Personalization: Information that are curated for an intended use
   - Connection: Information sources that are connected to the platform (one or two-way)
   - Collaboration: Producers and consumers can jointly work together seamlessly
4. Document in details, your analysis in docs/01-analysis, and plans in docs/02-plans, and user flows in src/velus/docs/user-flows.
   - Use as many subdirectories and files as required
   - Name them sequentially as 01-, 02-, etc, for easy referencing

# Must do
1. I am not very familiar in this area
   - please ultrathink and find all the information gaps that you will need to ensure that your solution
     - is complete
     - considers all stakeholders
     - prempts all possible hiccups and challenges
     - and any other stuff that I have missed.
   - Actively clarify your doubts and guidance requirements with me

# Additional Notes
What is a Platform Model?
- Users (producers, consumers, partners)
  - Producers: Users who offer/deliver a product or service
  - Consumers: Users who consume a product or service
  - Partners: To facilitate the transaction between producers and consumers

# Additional Tasks
1. Review your work thoroughly, then address all the limitations that you have found
   - Research and recommend what technology/stakeholders do we need that will optimally resolve the gaps
2. Technical Architecture
   - AI should never be simulated even during development, the POC must demonstrate full AI capabilities
   - We should have these languages: English, Chinese, Bahasa (both)
   - We also need post-connectivity sync to cloud
3. The plans are very thin, I want you to ultrathink and use subagents to read through the analysis in details
   - Ensure that all the considerations and actionables are reflected and properly considered in our plans
   - Scrutinize the plans after you are done. Adopt a stakeholder perspective and question the value. 
   - Do not simply engage in naive technical depth.