# ace-rest-ws

A small Spring Boot / Spring-WS SOAP service that exposes a `CustomerDetails` web service — the concrete backend that an IBM App Connect Enterprise (ACE) flow calls to create a customer.

*Part of the IBM Client Engineering Cloud Pak for Integration production-deployment demo: this is the SOAP backend the ACE JSON-to-SOAP integration targets, so the demo has a real system to invoke rather than a mock.*

## What this is

The demo's headline pattern is "expose a modern REST/JSON API in front of a legacy SOAP system." ACE does the REST-to-SOAP mediation; **something has to answer the SOAP call on the other side.** This repo is that something: a contract-first `customerDetailsRequest` service that validates the inbound SOAP payload, mints a customer ID, and returns `status: success`. It runs plain HTTP for quick demos and mutual-TLS (mTLS) for the secure, production-shaped variant.

## What's inside

- **`soapserver/`** — the service itself. Java 8 / Spring Boot 2.4.3, built with Gradle.
  - `src/main/resources/customer_details.xsd` — the contract. The build runs JAXB (`xjc`) to generate request/response classes from it, and Spring publishes the matching WSDL live at `/ws/customerDetails.wsdl`.
  - `src/main/java/.../CustomerDetailsEndpoint.java` — the endpoint; namespace `http://ibm.com/CustomerDetails/`, POST to `/ws`, returns a UUID customer id.
  - `WebServiceConfig.java` — wires the `MessageDispatcherServlet` and a `PayloadValidatingInterceptor` that rejects any request/response that doesn't match the XSD.
  - `Dockerfile` — multi-stage build (Gradle build stage + slim JRE runtime).
  - `src/main/resources/application-secure.yaml` — the mTLS profile (port 8443, `client-auth: need`, JKS key/trust stores).
  - `deployment/docker/` (Docker Compose, secure and non-secure) and `deployment/ocp/` (OpenShift `Template` YAML: Deployment + Service + Route, plus `generate_image.sh`).
- **`cert-generation/`** — `generate.sh` spins up a container that produces the CA, server/client certs, keys and JKS stores used by the mTLS profile.
- **`appendix/`** — the published `customerDetails.wsdl` and a starter Kubernetes `Secret` manifest.
- **`common/`, `env.sh`** — shared shell helpers and the certificate/deployment configuration (common name, SANs, org/locality, store paths).

## Why it's built this way

- **Contract-first, not code-first.** The XSD is the single source of truth: JAXB generates the Java types and Spring serves the WSDL from the same schema, so the interface ACE imports can never drift from what the server actually accepts. Change the contract in one file and everything downstream regenerates.
- **The server enforces the contract.** `PayloadValidatingInterceptor` validates both directions against the XSD. That makes the demo honest — a malformed message from the ACE flow gets a real SOAP fault, exactly like a legacy backend would behave, instead of silently passing.
- **Two fidelity levels, one codebase.** A non-secure HTTP mode for a fast local `docker-compose up`, and an mTLS mode (client-auth *required*) for showing the production-grade, mutually-authenticated call path. Certs are generated reproducibly from `env.sh`, so the secure story is repeatable, not hand-crafted.
- **Modernized runtime.** The `soapserver/Dockerfile` was moved off the archived AdoptOpenJDK / `openjdk:8-alpine` base to **Eclipse Temurin**, with **fully-qualified image names** (`docker.io/library/...`) so it builds cleanly under buildah's short-name resolution on OpenShift — keeping the legacy Java 8 backend buildable on a current 2026 toolchain.
- **Deploys the way the rest of the demo deploys.** OpenShift `Template` objects (Deployment/Service/Route) mean the backend stands up on the same cluster, in a namespace, reachable over a Route — the same footprint ACE and the other integration components use.

## How it fits the bigger picture

In the end-to-end story, a client calls a **REST/JSON API**, **ACE** mediates JSON-to-SOAP, and **this service** is the SOAP endpoint that ACE invokes to actually create the customer and hand back an id. It is deliberately the "legacy backend" in the picture — the thing you'd normally be unable to change — which is what makes the ACE modernization layer in front of it worth demonstrating. Deploy it (secure or non-secure) first, point the ACE flow's SOAP request node at its Route, and the create-customer path runs against a genuine backend from front door to system of record.

Maintained by IBM Client Engineering.
