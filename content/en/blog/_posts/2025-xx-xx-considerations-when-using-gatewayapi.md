---
layout: blog
title: "What you should be considering when selecting a Gateway API implementation"
date: 2025-11-06T09:00:00-08:00
slug: selecting-a-gwapi-controller
author: >
    [Ricardo Katz](https://github.com/rikatz) (Red Hat),
    [Rob Scott](https://github.com/robscott) (Google)
---

# Choosing Your Gateway API Implementation

Selecting a Gateway API implementation can be challenging. With many options available, each using different technologies and offering unique features, there are several key factors that can inform your decision.

This guide covers characteristics that organizations commonly consider when selecting a Gateway API implementation.

## I. Technical Considerations

Technical characteristics include whether an implementation meets feature requirements, aligns with team expertise, and satisfies performance needs.

### Feature Set Conformance

Every Gateway API release includes conformance reports that show which features each implementation supports (fully or partially). You can find the reports for Gateway API 1.4 [here](https://gateway-api.sigs.k8s.io/implementations/v1.4/).

These reports are generated through an automated testing process developed by Gateway API contributors. Any project can run these tests and submit their results to be included in the official implementations list. The list is continuously updated as new implementations are added.

Additionally, picking a conformant implementation and sticking with features that are marked as  “core” or “extended” [support level](https://gateway-api.sigs.k8s.io/concepts/conformance/#2-support-levels) will dramatically decrease the cost of migration in the future.

### Underlying Technology

The proxy technology that powers an implementation can be relevant, particularly in on-premises environments or when strict control over infrastructure is required. Many current open-source implementations are built on [Envoy Proxy](https://www.envoyproxy.io/), a graduated CNCF project. However, other implementations use different proxy technologies.

Team familiarity with a specific proxy, or an organization's existing technology stack, may influence the choice.

### Integration Level

Cloud providers or Container Network Interfaces (CNI) may already include a Gateway API implementation. Understanding what's available in existing infrastructure can inform the evaluation.

Integrated solutions typically provide:
* Provider-managed scalability and maintenance
* Deeper integration with the CNI, potentially unlocking enhanced features
* Reduced operational complexity

External implementations may offer:
* More feature completeness or faster access to new Gateway API features
* Greater portability across different environments
* More control over versioning and upgrade timing

The decision between integrated and external implementations often depends on feature requirements, operational preferences, and whether tight platform integration or flexibility is prioritized.

### Other Technical Factors

Additional questions that may be relevant to an evaluation:

* **Documentation**: Is the implementation well-documented? Can teams easily find the information needed to deploy and operate it?
* **Community and Maintenance** (for open-source): Is the project actively maintained? Do maintainers respond to issues promptly? Is the project welcoming to contributions?
* **Performance**: Does the implementation meet organizational latency and throughput requirements?
* **Extensibility**: Can the implementation be customized or extended if needed?
* **Platform Compatibility**: Does the implementation support the required Kubernetes versions, CPU architectures (amd64, arm64), and operating system requirements?

## II. Non-Technical Considerations

Non-technical characteristics affect how an implementation is distributed, supported, and maintained over time.

### Governance Model and Licensing

For open-source implementations, governance structure and licensing are common considerations. Projects vary widely:

* **Small maintainer teams**: May innovate quickly but carry higher sustainability risk
* **Corporate-backed**: Typically well-resourced but decision-making may reflect corporate priorities
* **Foundation-governed** (e.g., CNCF): Generally offer more stability, transparent governance, and long-term sustainability

The governance model affects:
* How new features are prioritized and added
* How security vulnerabilities are handled
* The project's long-term viability

Additionally, implementations use different open-source licenses (Apache 2.0, MIT, GPL, etc.), which may affect organizations depending on their legal requirements and policies around license compatibility.

Organizations often weigh governance models and licensing against their risk tolerance, values, and requirements for decision-making transparency.

### Service Level Agreements (SLAs) and Commercial Support

Organizations with mission-critical workloads may evaluate whether they need guaranteed response times, uptime commitments, or dedicated support resources. Some implementations offer:

* Paid support plans with defined SLAs
* Corporate backing and professional services
* Clear escalation paths for critical issues

The choice between community support and commercial agreements often depends on your organization's risk profile, internal expertise, and the criticality of the workloads running on the implementation.

---

## Making Your Decision

No single implementation is right for everyone. Different organizations prioritize different factors based on their specific contexts:

* **Feature requirements**: Conformance reports show what each implementation supports
* **Team expertise**: Familiarity with underlying technologies can affect operational success
* **Existing infrastructure**: Available integrated options may influence the evaluation
* **Risk and support needs**: Governance models and support options vary across implementations

Organizations often start by identifying their must-have features and constraints, then evaluate implementations against those criteria. The trade-offs between the remaining options can be assessed based on the factors outlined in this guide.



