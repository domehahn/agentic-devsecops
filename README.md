# Security Properties für Agentic Skills

63 deutschsprachige MDX-Beiträge im Stil der bestehenden Agentic-DevSecOps-Serie.

Jeder Beitrag enthält:

- Frontmatter mit `Security Property` Tag
- Erklärung der Property im Kontext von Agentic Skills
- negatives Beispiel
- positives Beispiel
- statische Erkennungsindikatoren
- technische Controls
- Grenzen der Erkennung
- DevSecOps-Einordnung
- `KeyTakeaway`

## Beiträge

1. [Instruction Override](01-instruction-override.mdx)
2. [Role/Context Manipulation](02-role-context-manipulation.mdx)
3. [Anti-Refusal](03-anti-refusal.mdx)
4. [Warning Suppression](04-warning-suppression.mdx)
5. [Guardrail Nullification](05-guardrail-nullification.mdx)
6. [Hidden/Invisible Instructions](06-hidden-invisible-instructions.mdx)
7. [Behavioral Steering / Manipulation](07-behavioral-steering-manipulation.mdx)
8. [Generic Physical Harm](08-generic-physical-harm.mdx)
9. [HTTP/Data Exfiltration](09-http-data-exfiltration.mdx)
10. [Env/Secret Harvesting](10-env-secret-harvesting.mdx)
11. [Filesystem Reconnaissance](11-filesystem-reconnaissance.mdx)
12. [Conversation/Context Exfiltration](12-conversation-context-exfiltration.mdx)
13. [Cloud Storage Exfiltration](13-cloud-storage-exfiltration.mdx)
14. [Excessive Permissions](14-excessive-permissions.mdx)
15. [sudo/root escalation](15-sudo-root-escalation.mdx)
16. [Credential-file access](16-credential-file-access.mdx)
17. [Docker Socket](17-docker-socket.mdx)
18. [Container Escape / privileged workload](18-container-escape-privileged-workload.mdx)
19. [Unrestricted Tool Access](19-unrestricted-tool-access.mdx)
20. [Approval Bypass](20-approval-bypass.mdx)
21. [Scope Creep](21-scope-creep.mdx)
22. [Missing Resource Bounds](22-missing-resource-bounds.mdx)
23. [Output → Execution](23-output-execution.mdx)
24. [Cross-context Output](24-cross-context-output.mdx)
25. [Output Limits](25-output-limits.mdx)
26. [Direct Prompt Leakage](26-direct-prompt-leakage.mdx)
27. [Indirect Prompt Leakage](27-indirect-prompt-leakage.mdx)
28. [Prompt → external/tool exfil](28-prompt-external-tool-exfil.mdx)
29. [Memory Poisoning](29-memory-poisoning.mdx)
30. [Context Stuffing](30-context-stuffing.mdx)
31. [Memory/State Manipulation](31-memory-state-manipulation.mdx)
32. [Self Modification](32-self-modification.mdx)
33. [Persistence](33-persistence.mdx)
34. [Trigger Abuse](34-trigger-abuse.mdx)
35. [Trigger Shadowing](35-trigger-shadowing.mdx)
36. [Tool Parameter Abuse](36-tool-parameter-abuse.mdx)
37. [Tool Chaining](37-tool-chaining.mdx)
38. [Unsafe Defaults](38-unsafe-defaults.mdx)
39. [Python AST AST1–AST9](39-python-ast-ast1-ast9.mdx)
40. [JS/TS Process Execution](40-js-ts-process-execution.mdx)
41. [Bash/Shell Analysis](41-bash-shell-analysis.mdx)
42. [Taint Tracking](42-taint-tracking.mdx)
43. [Unicode/Bidi/Confusables](43-unicode-bidi-confusables.mdx)
44. [Obfuscation](44-obfuscation.mdx)
45. [YARA/Malware](45-yara-malware.mdx)
46. [Dependency Pinning](46-dependency-pinning.mdx)
47. [OSV/CVE](47-osv-cve.mdx)
48. [Typosquatting](48-typosquatting.mdx)
49. [Abandoned Packages](49-abandoned-packages.mdx)
50. [Container Trust](50-container-trust.mdx)
51. [MCP Wildcards](51-mcp-wildcards.mdx)
52. [MCP Metadata Poisoning](52-mcp-metadata-poisoning.mdx)
53. [MCP Parameter Injection](53-mcp-parameter-injection.mdx)
54. [Description/Behavior mismatch](54-description-behavior-mismatch.mdx)
55. [MCP Rug Pull / mutable identity](55-mcp-rug-pull-mutable-identity.mdx)
56. [Underdeclared Capability](56-underdeclared-capability.mdx)
57. [Overdeclared Capability](57-overdeclared-capability.mdx)
58. [Agent Config Snooping](58-agent-config-snooping.mdx)
59. [MCP Config Snooping](59-mcp-config-snooping.mdx)
60. [Peer Skill Enumeration](60-peer-skill-enumeration.mdx)
61. [Cloud Metadata SSRF](61-cloud-metadata-ssrf.mdx)
62. [Internal/Loopback SSRF](62-internal-loopback-ssrf.mdx)
63. [Dynamic-target SSRF](63-dynamic-target-ssrf.mdx)
