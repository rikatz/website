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

## Should You Migrate from Ingress to Gateway API?

Before selecting a Gateway API implementation, it's important to understand the relationship between Ingress and Gateway API. Gateway API is the successor to Ingress, designed to address many of the limitations that emerged as Kubernetes use cases evolved. While both can manage ingress traffic, they take different approaches.

### Why Gateway API Was Created

Ingress has served the Kubernetes community well, but certain limitations became apparent over time:

* **Limited expressiveness**: Many advanced routing requirements (header-based routing, traffic splitting, cross-namespace references) could only be achieved through vendor-specific annotations, reducing portability.

* **Role-based separation**: Ingress combines infrastructure and routing concerns in a single resource, making it harder to delegate responsibilities in multi-tenant environments.

* **Protocol support**: While Ingress focuses on HTTP/HTTPS, modern applications increasingly need standardized ways to manage TCP, UDP, gRPC, and other protocols.

* **Standardization gaps**: The heavy reliance on annotations for advanced features meant that migrating between implementations often required significant reconfiguration.

Gateway API was developed to address these limitations while maintaining Kubernetes principles of declarative configuration and extensibility.

### When Ingress May Still Be Appropriate

There are some scenarios where continuing with Ingress may be reasonable:

* **Very simple, static configurations**: For applications with basic HTTP routing needs that will never require advanced traffic management, and where the simplicity of a single resource type is valued.

* **Short remaining lifecycle**: Applications that will be decommissioned in the near term may not justify the migration effort.

* **Implementation gaps**: In rare cases where a specific Gateway API implementation doesn't yet support a feature that your Ingress controller provides (though this gap is narrowing as Gateway API implementations mature).

* **Organizational transition period**: While preparing for Gateway API adoption, continuing with Ingress may be appropriate until the organization has completed its evaluation and migration planning.

### Benefits of Adopting Gateway API

For most use cases, Gateway API offers significant advantages:

* **Better portability**: Conformance testing and standardized features reduce lock-in and make it easier to switch implementations when needed.

* **Richer routing capabilities**: Header manipulation, request mirroring, traffic splitting, and other advanced features are part of the API specification rather than vendor-specific extensions.

* **Improved multi-tenancy**: The separation between Gateway (infrastructure) and Routes (application routing) allows cluster operators and application teams to work independently with appropriate boundaries.

* **Protocol flexibility**: Consistent patterns across HTTP, TCP, UDP, TLS, and gRPC reduce the learning curve and configuration complexity.

* **Active development**: Gateway API continues to evolve with new features and improvements, while Ingress is in maintenance mode.

* **Production-ready**: Gateway API v1.0 was released in October 2023, and many implementations are mature and battle-tested in production environments.

### Migration Considerations

When planning a migration from Ingress to Gateway API:

1. **Inventory your current setup**: Document the features and configurations you currently use, paying particular attention to any vendor-specific annotations.

2. **Evaluate Gateway API equivalents**: Most common Ingress patterns have direct equivalents in Gateway API, though the syntax differs. More complex annotation-based configurations may require researching the implementation's documentation.

3. **Consider migration tooling**: The [ingress2gateway](https://github.com/kubernetes-sigs/ingress2gateway) project is an ongoing community effort to provide automated conversion of Ingress resources to Gateway API. While still a work in progress, it may help with the initial conversion and serve as a learning tool.

4. **Plan for coexistence**: Many Gateway API implementations can run alongside Ingress controllers, allowing for gradual migration and testing before fully committing.

5. **Test thoroughly**: As with any infrastructure change, comprehensive testing in non-production environments is essential before migrating production traffic.

## Getting Started with Gateway API

If you've decided to explore Gateway API, here's a simple comparison showing how the same functionality looks in Ingress versus Gateway API.

### Ingress Example

In Ingress, you define a single resource that includes both the listener configuration and routing rules:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: default
spec:
  ingressClassName: nginx  # References the Ingress controller
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: example-service
            port:
              number: 80
```

### Gateway API Example

Gateway API separates infrastructure configuration (Gateway) from application routing (HTTPRoute):

**Step 1**: Most Gateway API implementations install one or more default `GatewayClass` resources during deployment. You can verify this with:

```bash
kubectl get gatewayclass
```

**Step 2**: Create a `Gateway` that references the `GatewayClass`. This defines the infrastructure (listeners, protocols, ports):

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example-gateway
  namespace: default
spec:
  gatewayClassName: example  # References the installed GatewayClass
  listeners:
  - name: http
    protocol: HTTP
    port: 80
```

**Step 3**: Create an `HTTPRoute` that references the `Gateway` and defines your routing rules:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-route
  namespace: default
spec:
  parentRefs:
  - name: example-gateway
  hostnames:
  - example.com
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: example-service
      port: 80
```

### Key Differences

* **Separation of concerns**: Gateway API separates infrastructure (Gateway) from routing (HTTPRoute), allowing infrastructure teams to manage listeners and protocols while application teams manage routing rules independently.

* **Cross-namespace references**: HTTPRoutes can reference Gateways in different namespaces (with appropriate RBAC permissions), enabling better multi-tenancy models where a shared Gateway serves multiple application namespaces.

* **Typed resources**: Gateway API uses protocol-specific resource types (HTTPRoute, TCPRoute, TLSRoute, etc.) with fields tailored to each protocol, rather than a single multipurpose resource.

* **Standardized configuration**: Advanced features like header modification, request mirroring, and traffic splitting are part of the API specification rather than implementation-specific annotations, improving portability between implementations.

## I. Technical Considerations

Technical characteristics include whether an implementation meets feature requirements, aligns with team expertise, and satisfies performance needs.

### Feature Set Conformance

Conformance testing is one of the most important distinguishing features of Gateway API compared to Ingress. Every Gateway API release includes conformance reports that show which features each implementation supports (fully or partially). You can find the reports for Gateway API 1.4 [here](https://gateway-api.sigs.k8s.io/implementations/v1.4/).

#### Why Conformance Matters

Conformance reports provide objective, verifiable information about implementation capabilities:

* **Feature transparency**: Instead of relying solely on marketing materials or documentation, conformance reports provide test-backed evidence of which features work in each implementation. This helps organizations make informed decisions based on actual behavior rather than claimed support.

* **Consistency verification**: When an implementation claims conformance for a feature, you can be confident that it behaves according to the Gateway API specification. This reduces the risk of subtle behavioral differences that can cause issues during development or migration.

* **Version compatibility**: Conformance reports are tied to specific Gateway API versions, making it clear which implementations support which API versions. This is particularly important when planning upgrades or when using newer Gateway API features.

* **Risk assessment**: By comparing conformance reports, you can identify which implementations have broader feature coverage or focus on specific areas. This helps match implementation strengths to your organization's requirements.

#### When Conformance Reports Are Helpful

Conformance reports become particularly valuable in several scenarios:

* **Initial selection**: When evaluating multiple implementations, conformance reports provide a standardized comparison of feature support, helping narrow down options that meet your technical requirements.

* **Migration planning**: Before migrating from Ingress or between Gateway API implementations, conformance reports help identify whether the target implementation supports all features you currently use or plan to use.

* **Cross-environment consistency**: Organizations running workloads across multiple environments (on-premises, cloud providers, edge) can use conformance reports to select implementations that provide consistent feature sets, even if they use different underlying technologies.

* **Vendor evaluation**: When a cloud provider or vendor offers a Gateway API implementation, conformance reports provide independent verification of their feature support, complementing vendor-provided documentation.

* **Risk management**: For risk-averse organizations, choosing implementations with high conformance scores for core features can provide confidence that foundational capabilities are well-tested and specification-compliant.

#### Reducing Migration Costs Through Conformance

One of the key benefits of Gateway API's conformance program is its impact on future migration costs. By choosing a conformant implementation and using features marked as "core" or "extended" [support level](https://gateway-api.sigs.k8s.io/concepts/conformance/#2-support-levels), organizations can significantly reduce the effort required to migrate between implementations later.

Here's why this matters:

* **Portable configurations**: Features with "core" or "extended" support levels are expected to work consistently across conformant implementations. Configurations using only these features typically require minimal or no changes when migrating to another conformant implementation.

* **Reduced testing burden**: When moving between conformant implementations, testing can focus on performance characteristics and edge cases rather than fundamental feature compatibility. This substantially reduces the testing effort compared to migrations between non-conformant solutions.

* **Avoiding lock-in**: Implementation-specific features (those outside the conformance scope) create lock-in by definition. By limiting their use or isolating them, organizations maintain flexibility to change implementations if requirements, costs, or vendor landscapes shift.

* **Predictable upgrades**: Conformant implementations commit to supporting the specified behavior across versions, making it easier to upgrade both the implementation and the Gateway API version with confidence.

This doesn't mean avoiding all implementation-specific features—many provide valuable capabilities—but understanding which parts of your configuration are portable and which create dependencies helps inform long-term planning and risk management.

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



