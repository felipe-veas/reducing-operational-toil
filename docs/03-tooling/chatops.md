# ChatOps: Safe and Auditable Execution

## 1. The Operational Problem: Terminal Isolation

As discussed in [Operational Tooling](cli-tooling.md), internal CLIs are far safer than raw infrastructure commands. However, running a CLI locally on an engineer’s laptop during a severe incident still presents a critical operational vulnerability: isolation.

When an Incident Commander (IC) asks an operator to drain traffic from a degraded availability zone, the operator executes the command in their own terminal. The rest of the response team—the IC, the database SME, the communications lead—is completely blind to what the operator just typed and what the system returned.

If the command fails cryptically, the operator has to copy-paste the stack trace into Slack. If they typo the command and take down the wrong zone, no one else on the bridge can see it happen to stop them. This creates an asynchronous, fragmented, and terrifyingly opaque response loop.

## 2. Incorrect Handling: Bastion Hosts and SSH

The traditional way to centralize command execution is the bastion host (jump box). Engineers are required to SSH into a fortified server before running operational tools.

While this prevents commands from originating on unsecured laptops, it does nothing to solve the isolation problem. An SSH session is still a private, 1-to-1 connection between an operator and a machine. It also introduces massive toil: managing SSH keys, VPNs, and IP allowlists.

During an incident, when speed is paramount, forcing a responder to wrestle with a VPN client, generate a short-lived token, and SSH through two jump boxes just to run a diagnostic command drastically inflates the Mean Time To Recovery (MTTR).

## 3. The Safer Approach: Conversational Operations (ChatOps)

ChatOps is the evolution of internal tooling. It moves the execution interface from the private terminal to the public incident channel (Slack, Teams, etc).

Instead of running a CLI locally, the operator types a command in the incident channel:
`@opsbot cluster drain --region us-east-1a`

The bot receives the command, executes the underlying automation, and replies in the channel with the exact output:
`[opsbot] Draining 45 nodes in us-east-1a. Estimated completion: 2 minutes.`

### The Advantages of ChatOps

1. **Shared Visibility:** The entire response team sees exactly what command was run, who ran it, and what the infrastructure returned. The IC never has to ask if a zone was drained. They watch it happen in real time.
2. **Zero SSH Access:** ChatOps eliminates the need for developers to ever SSH into production. The bot (running securely inside the VPC with strictly scoped IAM roles) is the only entity interacting with the infrastructure. The attack surface shrinks dramatically.
3. **Frictionless Auditing:** The chat channel is the audit log. When the incident concludes, the postmortem timeline writes itself. You don't have to parse complex CloudTrail logs or hope the operator remembers what they typed. The exact sequence of events, inputs, and outputs is durably recorded in Slack.
4. **Mobile Triage:** Because the interface is a chat app, an on-call engineer can acknowledge an alert, run a diagnostic query, and potentially execute a safe remediation directly from their phone, without needing to open a laptop and connect to a VPN.

## 4. Emphasizing Decision-Making and Behavior

Moving operations into a shared conversational space fundamentally changes the psychology of incident response.

It acts as a massive force multiplier for junior engineers. A junior on-call can watch a senior engineer execute a complex debugging sequence via ChatOps during an incident. The junior engineer learns the commands, the syntax, and the diagnostic workflow simply by reading the channel history.

Crucially, it enforces peer review under pressure. Because commands are typed into a public channel, other engineers can immediately spot a dangerous typo (e.g., `prod` instead of `staging`) and flag it before the bot executes it.

To implement ChatOps safely, the bot must require explicit, out-of-band approvals for destructive actions. If an operator types `@opsbot db failover`, the bot should ping the Incident Commander with an "Approve / Reject" button directly in Slack. This marries the speed of automation with the strict governance of explicit human consent.
