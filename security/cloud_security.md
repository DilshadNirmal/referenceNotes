# What is Cloud Security?

## Overview

Cloud security is about protecting cloud-hosted data, applications, and infrastructure from threats while maintaining compliance with regulations. It hinges on a shared responsibility model:

- **Security of the Cloud** — Handled by the cloud provider (e.g., AWS): includes physical data centers, network infrastructure, and foundational services.
- **Security in the Cloud** — Handled by you, the customer: covers operating systems, applications, data, and configurations.

> “The more control you have, the more responsibility you take on.”

### Shared Responsibility Model

![AWS Shared Responsibility Model](../images/cloud_security/shared-responsibility-model.png)

_(Source: AWS re:Invent presentations, shared under Creative Commons)_

---

## Types of Data & Their Impact

- **Confidential** – Bring your own encryption keys.
- **Sensitive** – You manage the keys.
- **Public** – Lower security needs, though still requires monitoring.

Data classification shapes your security architecture.

---

## Core Principles of Cloud Security

### 1. Data Protection

- Encrypt data at rest and in transit.
- Securely manage encryption keys.
- Provide backups, disaster recovery, and resilience.

### 2. Identity and Access Management (IAM)

- Enforce least privilege.
- Use multi-factor authentication (MFA).
- Utilize role-based access control.

### 3. Network Security

- Implement network and micro-segmentation.
- Utilize firewalls and IDPS.
- Protect against DDoS.
- Use private endpoints.

### 4. Monitoring & Logging

- Centralize logs.
- Enable real-time alerts for anomalies.
- Leverage SIEM tools for analysis.

### 5. Compliance & Regulatory Requirements

- Align with industry-specific regulations.
- Regularly audit and document compliance.

---

## Common Cloud Security Challenges

- Data breaches & unauthorized access
- Insecure APIs/interfaces
- Insider threats
- Denial-of-service attacks
- Advanced persistent threats (APTs)

---

## Modern Security Practices

- **DevSecOps** — Integrate security throughout build and deploy phases.
- **Incident Response** — Be prepared to detect, respond, and recover from incidents.

---

**Summary:**  
Cloud security is a partnership—understand your part, mitigate risk, and maintain control.
