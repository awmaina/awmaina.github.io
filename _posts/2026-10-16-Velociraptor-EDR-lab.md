---
layout: post
title: "DFIR Lab Analysis EDR/Velociraptor"
author: Alex Maina Weru
date: 2026-10-06
categories: [DFIR Fundamentals, Velociraptor]
tags: [DFIR, Threat, velociraptor]
image:
  path: assets/pcap/v.jpeg
  alt: threat
---

# Velociraptor EDR Lab — Detailed Write-Up

> **Source note:** The supplied Blue Cape lesson URL redirected to the public course overview. That page says lesson content is unavailable unless the learner has access to the course. I could not view or retrieve the lab video, its transcript, or the lab VM. This document is therefore a practical, source-backed write-up for the lab topic named in the URL—not a timestamped summary of the unseen video. It does not claim that the steps below are the instructor’s exact sequence. [1]

## Lab purpose

This lab is about using Velociraptor as an endpoint response and investigation platform: identify an endpoint, select targeted artifacts, collect host evidence, inspect collection results, and use the findings to guide follow-up investigation. Velociraptor lets responders run artifact-based collections on one or more endpoints and use VQL—the Velociraptor Query Language—to shape endpoint queries and hunts. [2] [6]

The course overview describes a broader DFIR course with case files for local practice and an optional ready-to-use lab VM. It lists Velociraptor among the tools covered. Use the course-provided case files or VM if available; the generic scenario below is not a claim about the course’s actual evidence set. [1]

## Learning objectives

By the end of the exercise, you should be able to:

1. Explain the difference between a Velociraptor artifact and a forensic artifact found on a computer.
2. Locate a client, launch a narrowly scoped artifact collection, and review the result tables, logs, and uploaded files.
3. Correlate process, persistence, event-log, and file evidence into a defensible timeline.
4. Distinguish an observed fact from an investigative hypothesis.
5. Preserve a record of what was collected, from which endpoint, when, with what parameters, and where the exported results were stored.
6. Decide when a single-client collection is enough and when a controlled hunt across a wider set of clients is justified.

## Scenario and investigation question

Use an isolated lab endpoint that you are authorized to examine. For practice, assume an alert has raised concern that an unknown program ran on a Windows workstation and may have established persistence. The initial question is:

> **What evidence on this endpoint supports or contradicts the possibility of suspicious execution and persistence, and what additional collection is warranted?**

Treat that as a working hypothesis, not as a finding. The exercise is successful even if the evidence is inconclusive, provided the collection is scoped, the observations are recorded accurately, and uncertainty is made explicit.

## What Velociraptor is doing

Velociraptor artifacts package one or more VQL queries and associated configuration in a human-readable definition. In client/server operation, the server compiles the selected artifact and sends the resulting query logic to the client for execution. The artifact definition is managed on the server; the client returns results and, when requested, files. [2]

Do not confuse a **Velociraptor artifact**—a reusable collection definition—with a **forensic artifact**, such as a scheduled-task record, event log, prefetch file, or registry value. The former is the query/collection mechanism; the latter is the evidence source the investigator may examine. [2]

A collection may return structured rows, uploaded files, and execution logs. The GUI’s collection view separates these kinds of output, so inspect all of them rather than relying only on the first result table. [3]

## Preparation and evidence discipline

Before launching a collection:

- Confirm the lab VM or endpoint is isolated and authorized for testing. Record the case or exercise identifier, endpoint name or client ID, operating system, time zone, and the time shown by the management console.
- Write down the question the collection is intended to answer. Prefer a small set of targeted artifacts over a broad “collect everything” approach.
- Capture the selected artifact names, parameters, time ranges, file paths, and any resource limits. Read the collection request in the GUI before launch; the request preview provides a final opportunity to understand what will run. [3]
- Note that live response is not passive: querying a running endpoint and transferring files can consume resources and may affect the system’s state. Keep collection scope proportionate to the question.
- Use a lab copy of evidence where possible. Store exports in a case-specific, access-controlled folder. Record who handled the data and when, and calculate hashes for downloaded evidence files if your lab procedure supports it.

NIST describes computer forensics as a process that may draw on files, operating systems, network traffic, and applications, and cautions that its guide is not a complete step-by-step investigation manual or legal advice. Follow the organization’s evidence-handling and legal requirements for real incidents. [7]

## Lab procedure

### 1. Confirm the endpoint and establish a baseline

In the Velociraptor GUI, locate the lab client and verify that it is the intended machine. Record its client identifier and available host details. If the endpoint is offline or not reporting, do not interpret missing collection results as proof that the host is clean; record the connectivity limitation and retry only according to the lab’s rules.

Start with a small baseline collection. Search the artifact catalog for an endpoint-information or host-information artifact appropriate to the operating system. Review the artifact description and parameters before selecting it. Capture basic context such as hostname, operating system, logged-on users, and collection time if those fields are offered.

### 2. Collect a targeted process view

Search the artifact catalog for a process-list artifact. The official hunting documentation uses `Windows.System.Pslist` as an example of a process artifact that can be collected across clients. Confirm that the artifact is present in your server’s catalog and review its current description before use. [4]

Launch the collection against the single lab client first. Inspect the returned rows for process name, path, command line, user, process identifier, parent process, and start time where available. Do not label a process malicious just because its name is unfamiliar. Record concrete reasons for concern, such as an unexpected execution path, suspicious parent-child relationship, unusual command-line arguments, or a mismatch with the lab scenario.

If the artifact does not provide parent-process context or other fields needed to answer the question, note the gap. Choose a second targeted artifact or file collection only when it will resolve that gap.

### 3. Check likely persistence and execution evidence

Search for artifacts related to scheduled tasks, services, autoruns, and other operating-system persistence locations that match the platform and question. The official client-collection tutorial demonstrates `Windows.System.TaskScheduler` and shows how artifact parameters can be reviewed or adjusted. [3]

For each result that may be relevant, record the exact name, path or registry location, configured command, user context, timestamps, and any referenced executable. Compare those facts with the process listing and the lab’s alert context. A scheduled task or service is not inherently malicious; the question is whether its configuration, origin, timing, and execution evidence fit an unauthorized activity hypothesis.

Where appropriate, collect the underlying task definition or other raw source file as well as parsed rows. A parsed result is convenient for searching, while the source file can help validate how the parsed fields were derived. Keep the upload focused and avoid collecting unrelated personal or business data.

### 4. Review event logs and build a timeline

Search for the platform’s event-log artifacts and use their descriptions to select the relevant channels and time range. The course overview identifies Windows events and timeline analysis among the wider course’s assessment areas, but it does not reveal which logs or events the inaccessible lab video uses. [1]

Correlate the available timestamps across process evidence, persistence records, and event-log findings. Record the source and time zone for each timestamp. Distinguish event time from collection time, and account for clock skew or conversion differences where they may affect ordering.

A useful timeline entry states what the source actually shows, for example: “At time X, source Y recorded event Z for account A.” Keep interpretation separate: “This may indicate execution by the scheduled task” is an assessment, not a direct observation, unless the evidence establishes that link.

### 5. Review every collection output

Open the completed collection and review:

- **Result tables:** Which artifacts returned rows? What fields and values are relevant? Are there empty results, parsing errors, or unexpected gaps?
- **Logs:** Did the query complete? Were there warnings, timeouts, access-denied messages, or resource-limit events?
- **Uploaded files:** What files were transferred? Are they expected and relevant to the investigation question?
- **Collection metadata:** Which client ran the collection, which artifacts and parameters were used, and what was the recorded status and timing?

Velociraptor’s collection documentation describes these separate views and the ability to inspect result sets and uploaded files. Treat a successful launch as different from a successful, complete, and interpretable collection. [3]

### 6. Decide whether to expand the investigation

Use the first collection to refine the next question. If a suspicious path, task, service, or user is identified, perform a focused follow-up collection on the same host. Avoid widening scope merely because the tool makes it easy.

If the evidence supports looking for the same behavior on multiple endpoints, consider a hunt. A hunt groups one or more artifact collections across a set of clients. The Velociraptor documentation recommends testing artifact selection on one or two endpoints before applying it broadly. Hunts have an expiration time and may continue to apply to eligible clients while active; they are not simply a one-time completed batch. [4]

Before starting a broader hunt, define the target population, matching criteria, time window, expected data volume, and stop/expiry point. Prefer targeted result rows to bulk file retrieval when possible. Large uploads can scale rapidly across a fleet. [4]

### 7. Export and document the evidence

Export the result tables and any relevant uploaded files using the lab’s supported export workflow. Keep original exports unchanged. If you create working copies, label them as working copies and retain a link to the originals.

For each exported item, record:

- case identifier and endpoint/client ID;
- artifact name, collection ID, parameters, and any relevant VQL or artifact version details visible in the interface;
- collection start/end time and time zone;
- result status, warnings, and known gaps;
- original export filename, storage location, and file hash where available;
- analyst, date, and any transformation performed before analysis.

The purpose is reproducibility: another analyst should be able to understand what was collected, what the tool reported, and how the interpretation was reached.

## Findings worksheet

Use the following prompts in the lab notebook or report:

### Collection facts

- Which endpoint was examined?
- Which artifacts were launched, and what question did each answer?
- What parameters and time ranges were set?
- Did the collections finish? What warnings or errors appeared?
- Which result rows and files were saved?

### Evidence observations

For every potentially relevant observation, capture the source, exact value, timestamp, and context. Examples include a process path and command line, a task action and trigger, or a log record. Avoid unsupported conclusions such as “the host was compromised” when the evidence only establishes that an unusual object exists.

### Assessment

State whether the evidence supports, weakens, or does not resolve the working hypothesis. Explain how observations correlate and identify alternative explanations. List the next collection that would most efficiently resolve the remaining uncertainty.

### Limitations

Record offline endpoints, missing logs, permission failures, collection timeouts, clock uncertainty, incomplete data sources, or uncollected volatile information. A lack of findings in one artifact is not proof that no suspicious activity occurred; it may mean the relevant data was unavailable, outside the time window, or not queried.

## Expected outcome and completion criteria

A strong lab result is not simply “the tool ran.” The exercise is complete when you can show that:

1. The correct lab endpoint was selected and the collection scope was appropriate.
2. Artifact descriptions and parameters were reviewed before execution.
3. Results, uploaded files, and execution logs were examined.
4. Observations are separated from interpretations and tied to their sources.
5. Findings are correlated into a timeline with timestamp caveats recorded.
6. The final assessment answers the investigation question without overstating certainty.
7. Exports and collection details are preserved in a reproducible case record.

## Common mistakes to avoid

- **Launching a fleet-wide hunt before testing:** Validate the artifact and its parameters against a small, authorized target set first. [4]
- **Collecting too much:** Broad file collection increases endpoint impact and storage volume. Start with focused queries and expand only when findings justify it. [2] [4]
- **Treating empty output as a clean bill of health:** Check collection status, logs, access, time range, and artifact scope before drawing that conclusion.
- **Confusing a parsed result with the complete underlying evidence:** Collect source files when needed to validate or deepen analysis, and preserve them separately.
- **Ignoring time zones and clock differences:** Timestamp comparison is only meaningful when its source and time basis are understood.
- **Calling something malicious based on its name alone:** Validate paths, command lines, context, timestamps, and corroborating evidence.
- **Assuming Velociraptor replaces every forensic method:** It provides endpoint collection and analysis capabilities, but the correct toolset depends on the question and evidence source. Some investigations may require other network, disk, or memory acquisition and analysis methods. [5] [7]

## References

[1]: https://bluecapesecurity.com/courses/dfir-foundations-techniques-readiness/ "Blue Cape Security DFIR Foundations and Techniques course overview"
[2]: https://docs.velociraptor.app/docs/artifacts/ "Velociraptor Artifacts"
[3]: https://docs.velociraptor.app/docs/clients/artifacts/ "Velociraptor Collecting Artifacts"
[4]: https://docs.velociraptor.app/docs/hunting/ "Velociraptor Hunting"
[5]: https://docs.velociraptor.app/docs/forensic/ "Velociraptor Forensic Analysis"
[6]: https://www.cisa.gov/resources-tools/services/velociraptor "CISA Velociraptor overview"
[7]: https://csrc.nist.gov/pubs/sp/800/86/final "NIST SP 800-86, Guide to Integrating Forensic Techniques into Incident Response"

