# ace-rest-ws

A sample Spring Boot SOAP web service (a customer-details service) used to demonstrate
exposing and consuming a SOAP/WSDL backend from IBM App Connect Enterprise (ACE) on
Cloud Pak for Integration. It ships with certificate generation for mutual TLS, plus
Docker Compose and OpenShift deployment templates for both non-secure and secure (mTLS) modes.

Maintained by **IBM Client Engineering**.

## Layout

- `soapserver/` — the Spring Boot SOAP web service, with Docker and OpenShift deployment templates.
- `cert-generation/` — script to generate the CA, certificates, keys and keystores for mTLS.
- `appendix/` — sample WSDL and a starter Kubernetes Secret manifest.
- `common/`, `env.sh` — shared shell helpers and environment configuration.

See [`soapserver/README.asciidoc`](soapserver/README.asciidoc) and
[`cert-generation/README.md`](cert-generation/README.md) for component-level instructions.
