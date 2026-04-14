# how-to-program-workflow

## Observations
### 2026-04-13
- Problems split into well-defined problems and ill-defined problems.
- Three types of well-defined problems: arrangement, inducing structure, and transformation. (Ref: Posner, *Cognition: An Introduction*, 1973 — unverified)
- External environments in a workflow split into two kinds: shared workspaces (Notion, Jira, Google Docs) and compute (on-premise, deployment).

## Questions
### 2026-04-13
- If a workflow is a program, what are its input, output, invariant, and effect?
- A workflow is an implementation of a problem-solving procedure. What is a task/problem in the first place?
- Does a well-defined problem also require measurement to be defined, or is that implied by the goal state?
- Is the generation-verification gap related to the inherent properties of the problem type? (Ref: [generation-verification gap hypothesis](https://www.emergentmind.com/topics/generation-verification-gap-hypothesis), [asymmetry of verification and verifier's law](https://www.jasonwei.net/blog/asymmetry-of-verification-and-verifiers-law))

## Hypotheses
### 2026-04-13
- Ill-defined problems are inherently intractable. We approximate them as well-defined problems — this approximation process is called problem framing.
- Solving the framed well-defined problem does not necessarily solve the original ill-defined problem. Recognizing that gap and re-framing is essential.
- A well-defined problem is a state space with: (1) initial state = boundary condition, (2) goal state = utility function, (3) operations = state transitions.
- Workflows are now a programming target.
- The criteria for splitting a workflow into steps matter a lot.
