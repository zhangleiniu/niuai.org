---
title: "Integrate RL with LLM Agents"
date: 2026-02-20
draft: false
tags: ["Research", "LLM"]
author: "Ibrahim Al Azher"
description: "Background and Proposed Framework"
featured_image: "title.jpeg"
cover:
    image: "title.jpeg"
    relative: false
---

[Download the presentation slides](slides.pdf)

### Overview and Motivation

- Presentation on integrating reinforcement learning concepts with LLM engines to improve performance
- Multiple approaches exist to enhance LLM performance: in-context learning, post-training (reinforcement learning), and fine-tuning
- Focus on efficiently integrating RL concepts with LLMs, particularly for multi-agent systems
- Covers differences between reinforcement learning on LLMs versus reinforcement learning on LLM agents
- Addresses RL implementation in HPC systems due to memory-intensive requirements

### Multi-Agent System Fundamentals

- Multi-agent systems consist of multiple LLMs, each with specific roles and functions
- State acts as history, compiling all previous agent turns, context, and evidence in multi-turn systems

**Communication Topologies**  

- **Sequential (Chain):** Agents work sequentially where Agent B depends on Agent A's output before the master agent makes final decision
- **Hierarchical (Leader-Worker):** Parallel approach where workers operate simultaneously and communicate with leader agent
- Leader agent provides feedback to worker agents, enabling performance updates
- Hierarchical approach is faster than sequential

**Self-Consistency and Multi-Path Generation**

- Workers can generate multiple paths using self-consistency by forcing individual LLMs to explore multiple directions
- Can use multiple temperatures or command LLMs to choose multiple paths
- Example: Four worker agents with four paths each creates 16 total paths for leader to evaluate
- Inspired by principle: "agents sample candidates and only think when candidates conflict"

**Shared Memory**

- Shared workspace for agents that persists across turns, preventing information silos and giving leader full visibility
- Implemented as structured JSON or database where agents post information
- Worker agents have append-only access to their sections; master/leader has full read-write access
- **Benefits:**
    - Consistency: Avoids contradictions between agents
    - Context management: Reduces context window requirements
    - Conflict resolution: All workers access same shared memory
- Ray tool recommended for implementing shared memory and data parallelism in HPC

### Bottlenecks in Agentic Systems

**Master-Worker Mismatch**  

- Master/leader agent may hallucinate, omitting critical information during final synthesis
- Problem occurs when leader agent lacks sufficient knowledge about worker agent contexts
- Wrong feedback creates cascading errors, potentially leading to never-ending loops
- Leader agent relies on parametric knowledge which may be insufficient

**Other Challenges**

- **Over-commitment trap:** Agents commit to first generated solution without deep deliberation or structural critique
- **Feedback loops:** Agents may repeat points without progress
- **Poor feedback quality:** Feedback from leader to workers may not be helpful
- **Credit assignment problem:** Difficulty determining which agent failed when final output is poor
- **Groundedness gap:** Similar to master-worker mismatch, challenge in determining what constitutes "good" or "perfect" answers

**Solutions to Bottlenecks**   

- Add verifier agent between worker and leader agents to score groundedness (e.g., 80%, 70%, 30%)
- Leader uses groundedness scores to direct workers appropriately
- Provide external tools (web search, databases) for verification to reduce hallucinations
- Implement stopping criteria based on iteration improvements: stop when score improvement is less than 10% between iterations
- Threshold can be adjusted based on task sensitivity

### Reinforcement Learning Approaches

**RLHF (Reinforcement Learning from Human Feedback)** 

- Uses human preference data (A vs B comparisons)
- Trains separate reward model using human preferences
- Employs PPO (Proximal Policy Optimization) algorithm

**RLAIF (Reinforcement Learning from AI Feedback)** 

- Replaces human feedback with teacher LLM (e.g., GPT-4)
- Teacher LLM judges which output is better between pairs
- Trains reward model based on AI preferences

**PPO (Proximal Policy Optimization)**  

- Actor-critic RL algorithm requiring four models: actor, reference, critic, and reward model
- Uses clipped objective to limit model changes per update
- Very memory-intensive due to need for critic model

**DPO (Direct Preference Optimization)** 

- Eliminates separate reward model required in PPO
- Uses chosen/rejected pairs to act as own reward model
- Less memory-intensive than PPO

**GRPO (Group Relative Policy Optimization)**        

- Model generates 4-8 responses for each input
- Calculates average advantage across all responses
- Identifies responses better than average (e.g., R2 and R3)
- Trains model to follow better-than-average responses
- More powerful than DPO by using average-based comparison

**KTO (Kahneman-Tversky Optimization)** 

- Only requires binary good/bad labels, not ranked pairs
- Simpler labeling requirement compared to DPO

### RL on LLM vs RL on LLM Agents

**Key Differences**   

- **RL on LLM:** Straightforward optimization of single-turn response quality
    - Goal: Improve output response based on input
    - Linear input-output relationship
- **RL on LLM Agents:** Optimize agent collaboration and specialized roles
    - Worker agents: How to synthesize information from input
    - Leader agents: How to be better judge and provide good feedback
    - Master agents: How to synthesize and represent all information
    - Must ensure efficiency and performance of each agent type, not just final output

**Implementation Differences**  

- **Input/State:**
    - LLM: Current user prompt
    - LLM Agents: Prompt plus history, processes, and conversations between agents
- **Action Space:**
    - LLM: Text generation in single block
    - LLM Agents: API calls, search queries, revisions, tool usage
- **Rewards:**
    - LLM: Subjective (human-rated or LLM-as-judge)
    - LLM Agents: Objective (verification scores, evidence matching)
- **Risks:**
    - LLM: Boring or repetitive text
    - LLM Agents: Information loss during synthesis

### Supervised Fine-Tuning for Agents

**Why SFT is Necessary**  

- RL is sparse reward problem - model must know how to call tools and format feedback messages to find reward
- Need high-quality training data to teach leader agents tool access and feedback provision
- Data can be synthetic or human-preferred

**SFT Dataset Construction**   

- Structured with thought, action, and communication styles
- Based on high-quality teacher trajectories
- Uses imitation learning
- Data collection involves labeling (human or RLAIF)

**Beyond Imitation: Trial and Error Learning**    

- Use RL to punish lazy or generic feedback
- Requires collecting: trajectories, rewards, and goal-based validation
- When leader provides poor feedback, reward is low (e.g., 10%)
- Model learns to avoid low-reward instructions and follow high-reward patterns

**Trajectory Optimization**  

- RL teaches agents to pivot when worker agents hallucinate
- Reward based on number of communication rounds needed
- Fewer rounds = higher reward (e.g., 2 rounds better than 5 rounds)
- More back-and-forth conversations indicate inefficiency

### RL Workflow for LLM Agents

**Complete Training Loop**         

1. Start with task/environment and expert/human (can be LLM or human)   
2. Expert prefers data (e.g., YW better than YL) 
3. Calculate DPO loss and update policy 
4. Train LLM-based agents with updated policy 
5. LLM agents interact with environment and generate trajectories 
6. Trajectories go to reward function (environment model) 
7. Reward loss updates policy again 
8. Iterative process requiring 1-10 iterations for final response 

**Model Architecture for Multiple Agent Types**       

- Most implementations use same base model for all agent types
- Save separate LoRA adapters for each agent category (worker, leader, master)
- Cannot work with single unified model - need three different adapters
- Datasets differ for each agent type based on their specific roles
- At inference, load appropriate adapter for each agent type

### Implementation: Scientific Paper Limitation Generation

**Project Overview**  

- Task: Generate scientific paper limitations using LLM agents and RL
- Input: Research paper with limitations section removed
- Ground truth: Paper's mentioned limitations and peer reviews
- Uses one-to-one matching between model responses and ground truth to measure performance

**System Architecture - Left Branch**    

- Input paper goes to multiple worker agents
- LLM generates knowledge checklist as additional information for workers
- Table database stores cited paper information for evidence extraction
- Worker agents communicate with verifier and reward agents based on criteria
- Leader agent receives scores from verifier and inputs from workers
- Leader provides feedback to workers: approve, improve, or modify
- Stops when performance improvement is less than 10% across two iterations
- Worker agent categories: innovation and practicality (communicate with leader in parallel)
- Master agent synthesizes all information, de-duplicates, and generates final limitations

**System Architecture - Right Branch (Novelty Agents)** 

- Takes input paper plus relevant papers from RAG (Retrieval-Augmented Generation)
- Cross-checks novelty differences between input paper and RAG papers
- Measures: novelty, technicality, experimental validation, literature aspects
- Novelty agents also communicate with leader agents for feedback
- Merger agent combines outputs from both left and right branches

**Data Collection (Rollout Process)**    

- Entire process called "rollout" for collecting training dataset
- For SFT: Must ensure data quality is very good (human or AI validation)
- For GRPO: Need at least 4-8 responses to calculate average

**GRPO Implementation with Multiple Modes**          

- Four different modes tested (task-specific):
    1. Strict grounding: Leader requires citations for all worker claims
    2. Critic heavy: Leader acts as harsh critic
    3. Retrieval heavy: Emphasizes communicative RAG
    4. Additional specialized approaches
- Each mode generates four responses (16 total paths)
- **Note:** For standard GRPO, multiple modes not required - can use single mode with four responses via self-consistency or temperature variation
- Most common approach: Chain of thought with self-consistency

**Training Requirements**   

- Three essential components:
    1. Log probabilities (token probabilities)
    2. Trajectories (agent communication patterns)
    3. Rewards (from reward model based on criteria)
- Need three LoRA adapters: worker, leader, master

**Inference Stage** 

- Load three fine-tuned LoRA models
- Repeat entire process for testing
- Generate final limitations
- Evaluate against ground truth (collected from papers' limitation sections and peer reviews)

**Reward Model**  

- Based on criteria: groundedness, specificity, adversarial penalty
- Uses LLM's parametric knowledge (no external information currently)
- Presenter acknowledges potential for hallucination in reward model
- Considering adding external information sources (RAG, databases) for verification

**DPO Experiments**       

- Tested DPO only on master agent (for merging capability)
- Used LLM-as-judge to determine chosen vs rejected pairs
- Generated two responses with different temperatures for comparison
- DPO showed considerable hallucinations
- GRPO performed better than DPO
- LLM agents alone (without RL) currently show superior performance
- Presenter acknowledges many improvements needed in implementation

**Ground Truth Collection**      

- Extracted from explicit limitation sections or subsections
- If no explicit section, searched in discussion, conclusions, or future work sections
- Used LLM-as-judge for better extraction from noisy data (when limitations not clearly bounded in conclusion sections)

### Training Approaches: Online vs Offline Policy

**Offline Policy**    

- Less memory-hungry compared to online policy
- **Stage 1:** Apply rollout to collect data
    - Collect trajectories (agent conversations)
    - Collect log probabilities (token probabilities)
    - Save LoRA adapters for each agent type
    - Save all information to JSON
- **Stage 2:** Update policy using same base model
- Two stages operate separately

**Online Policy**  

- Both stages work together simultaneously
- Does not save information from first stage separately
- Requires very large VRAM
- More memory-intensive but potentially more efficient

### Verification and Reward Models

**Influence-Based Verification** 

- Agent's reward includes both current score and future influence
- Formula: Immediate score plus gamma times impact on final master output

**Revision Trajectories** 

- Identify error steps and correct bad prefixes
- Use deletion signal and add good prefix
- Iterative correction approach

**Verifier Rubric** 

- Groundedness: How well-supported claims are
- Specificity: Level of detail and precision
- Adversarial penalty: Reverse scoring (lower is better)

**Stopping Criteria** 

- Based on 10% improvement threshold between iterations
- Task-dependent adjustment possible

### Memory Efficiency and HPC Considerations

**Model Size and GPU Requirements**      

- LLaMA 3 7B model needs at least 80GB GPU for inference
- Recommended: Use Q-LoRA with BFloat16 for optimization
- Some performance drop but acceptable for memory constraints
- 80GB GPU sufficient for LLaMA 3 8B with online policy
- Larger models require more GPU resources
- Recommended for online policy implementations

**Specific Memory Requirements**      

- LLaMA 3 7B inference: Listed as 35GB but actually needs over 40GB
- Does not fit in single GPU
- LLaMA 3 70B inference: Over 40GB
- For environments without 80GB GPUs (only 40GB available): Must use two GPUs working in parallel

**Fine-Tuning Requirements**    

- LLaMA 3 70B fine-tuning: More than 80GB, possibly around 100GB

**Implementation Recommendations**  

- Log probability collection challenging with offline policy
- PyTorch has code for collecting log probabilities but implementation is difficult
- Strongly recommend online policy over offline policy for easier implementation
- Collect version information and proceed with online approach

**LoRA Adapter Management** 

- Must save three adapters: worker, leader, master
- Load appropriate adapters during inference/testing

### Tools and Parallelization

**Ray for Parallel Processing**     

- Excellent tool for multi-agent parallel processing
- Industry-recommended
- Enables parallel worker-leader communication
- Example: Two parallel branches where worker agents communicate with leaders simultaneously
- For environments with two 40GB GPU nodes: Ray enables both nodes to work simultaneously
- Essential for parallel processing when worker agents are independent

**Sequential vs Parallel Approaches**  

- Sequential approach: Very time-consuming
- Sequential means: Worker 1 completes, then Worker 2 starts, etc.
- Parallel approach (with Ray): Much faster, workers operate simultaneously

**Other Recommended Tools** 

- LLaMA Engine for optimization

### GRPO Mathematical Details

**Training Stage (Old Model)** 

- Log probability calculation: log P_old(y|x)
- Where y = response, x = input

**Update Stage (New Model)**  

- Log probability calculation: log P_theta(y|x)
- Uses same model for policy update
- GRPO equation applies to calculate loss

### Q&A Discussion Highlights

**Question on Agent Specialization**           

- Hypothesis: Multiple agents focusing on specific tasks (programmer, tester, UI designer) may be more effective for complex tasks
- Question raised: Why train all agents together rather than individually?
- Presenter's response: Two approaches possible
    1. Train each agent individually to maximize performance
    2. Improve collaborative system (presenter's focus)  
- Assumes individual agents (like GPT) are already powerful
- Problem may come from collaboration, not individual capability
- Performance may drop across iterations (rounds 2, 3) indicating communication issues
- Training uses trajectories showing agent interactions and feedback patterns

**Verifier Reliability**      

- Concern: Verifier agent itself may hallucinate
- Not working with fixed milestones currently
- Depends on worker task
- Self-consistency may produce varying scores (e.g., 80% then 70%)
- Solution: Integrate external information sources (tools, databases) for verification

**Threshold Sensitivity**      

- 10% threshold varies by task
- For text-based tasks, 10% not huge jump
- Fixed at 10% for presenter's specific task
- Acknowledged as open parameter requiring tuning
- Initial experiments tried 40-50 rounds but took too much time
- May need to decrease threshold in future work

**Training Details**      

- Batch size and gradient accumulation: 16-32 (presenter needs to verify exact configuration)
- Training takes considerable time
- Gradient accumulation cycles need verification