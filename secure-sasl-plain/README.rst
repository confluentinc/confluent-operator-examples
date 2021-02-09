Secure deployment with SASL/Plain
=================================

This example covers the following deployment scenario:
- Kafka authentication with SASL PLAIN
- Authorization with RBAC
- Network encryption with TLS for all external and internal traffic

Identity is provided by the open source OpenLDAP server, for which an example deployment spec is provided.
Note: OpenLDAP is not included as part of Confluent Platform.

=======================
Customize the Template
=======================

A templated deployment spec is provided - `deployment-template.yaml`.

You'll provide the following inputs that customize the template to fit your environment:

- __domain__: Domain that external access will be provided on.
- __domain_prefix__: (Optional) Prefix used in front of __domain__
- TLS certificates:
  - For each component, provide the following:
    1. ``cacerts``: One or more concatenated Certificate Authorities (CAs) for the component to trust the certificates presented by the Kafka brokers. 
    2. ``fullchain``: A fullchain consists of a root CA, any intermediate CAs, and finally the certificate for the component.   
    3. ``privkey``: Contains the private key associated with the component certificate.

==================
TLS certificates
==================

Each Confluent component has been configured for TLS encryption. These certificates must have the Subject Alternative Name (SAN) entries appropriately configured to cover the following:

- The hostnames that the components will be available externally at
  - SAN entries should match ``*._DOMAIN_``
- The hostnames used in the internal Kubernetes Network
  - SAN entries should match ``*.<KAFKA_DEPLOYMENT_NAME>.<KUBERNETES_NAMESPACE>.svc.cluster.local``
  - Example is ``*.kafka.confluent.svc.cluster.local`` , where ``KAFKA_DEPLOYMENT_NAME=kafka`` and ``KUBERNETES_NAMESPACE=confluent``
- The hostname used in the internal Kubernetes Network that Jolokia is available at
  - SAN entries should match ``*.<KAFKA_DEPLOYMENT_NAME>.<KUBERNETES_NAMESPACE>``
  - Example is ``*.kafka.confluent`` , where ``KAFKA_DEPLOYMENT_NAME=kafka`` and ``KUBERNETES_NAMESPACE=confluent``

Here, Confluent is configured with ``interbrokerTLS`` as true, and thus the certificates used for Kafka brokers must have the Certificate Name (CN) be the Kafka super user.

=======================
Deployment Architecture
=======================

Confluent Operator will configure the following 4 listeners. All are configured with TLS, have a specific authentication mechanism, and are bound to a port:

1. External listener 
   - This is available for external (to the Kubernetes namespace) clients to communicate with Kafka. 
   - Authentication mechanism is SASL/Plain.
   - Port is 9092
2. Internal listener
   - This is available for internal (to the Kubernetes namespace) clients to communicate with Kafka. 
   - Authentication mechanism is SASL/Plain.
   - Port is 9071
3. Token listener
   - This is used by the Confluent components to communicate with Kafka.
   - Authentication mechanism is SASL/OauthBearer.
   - Port is 9073
4. Replication listener
   - This is used by the Kafka brokers to communicate with each other.
   - Authentication mechanism is SASL/Plain.
   - Port is 9072
