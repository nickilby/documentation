# Data Classification Policy

**Version:** 1.0  
**Last Updated:** 2025  
**Next Review Date:** 2026-01-01  
**Owner:** IT Security Team

## Purpose

This policy establishes a framework for classifying organizational data based on sensitivity, value, and regulatory requirements to ensure appropriate protection measures are applied.

## Scope

This policy applies to all data created, received, stored, processed, or transmitted by the organization, regardless of format (electronic, paper, or other media).

## Policy Statement

All organizational data must be classified according to its sensitivity and business value. Classification determines the security controls, handling procedures, and access requirements for data protection.

## Data Classification Levels

### Public (Level 1)

#### Definition
- Information intended for public disclosure
- No harm if disclosed
- No confidentiality requirements

#### Examples
- Public website content
- Marketing materials
- Public announcements
- Published reports

#### Handling Requirements
- No special protection required
- Can be freely shared
- Standard copyright protection
- Public access allowed

#### Labeling
- **Label:** PUBLIC
- **Color Code:** Green (optional)
- **Storage:** No restrictions

### Internal (Level 2)

#### Definition
- Information for internal use only
- Not intended for public disclosure
- Limited harm if disclosed

#### Examples
- Internal policies and procedures
- General business communications
- Internal reports
- Organizational charts

#### Handling Requirements
- Access limited to employees and authorized personnel
- Standard security controls
- No external sharing without authorization
- Standard retention policies

#### Labeling
- **Label:** INTERNAL
- **Color Code:** Yellow (optional)
- **Storage:** Standard file shares, email

### Confidential (Level 3)

#### Definition
- Sensitive information requiring protection
- Unauthorized disclosure could cause harm
- Business, legal, or competitive impact

#### Examples
- Financial data and budgets
- Customer lists and contact information
- Business plans and strategies
- Contract terms and agreements
- Employee information (non-PII)

#### Handling Requirements
- Access on need-to-know basis
- Encryption at rest and in transit
- Secure storage and transmission
- Access logging and monitoring
- Secure disposal

#### Labeling
- **Label:** CONFIDENTIAL
- **Color Code:** Orange (optional)
- **Storage:** Encrypted storage, restricted access

### Restricted (Level 4)

#### Definition
- Highly sensitive information
- Severe harm if disclosed
- Regulatory or legal protection required
- Highest level of protection

#### Examples
- Personal Identifiable Information (PII)
- Protected Health Information (PHI)
- Payment Card Information (PCI)
- Intellectual property and trade secrets
- Security credentials and passwords
- Legal and regulatory data

#### Handling Requirements
- Strict access controls (need-to-know)
- Strong encryption (at rest and in transit)
- Enhanced monitoring and logging
- Secure storage (encrypted, access-controlled)
- Secure transmission only
- Regular access reviews
- Secure disposal (shredding/certified deletion)
- Compliance with regulations (GDPR, HIPAA, PCI-DSS)

#### Labeling
- **Label:** RESTRICTED
- **Color Code:** Red (optional)
- **Storage:** Highly secure, encrypted, access-controlled

## Data Classification Process

### Classification Criteria

#### Sensitivity Factors
- Confidentiality requirements
- Regulatory requirements
- Business impact of disclosure
- Legal protection requirements
- Competitive sensitivity

#### Value Factors
- Business criticality
- Replacement cost
- Intellectual property value
- Customer trust impact

### Classification Assignment

#### Responsibility
- **Data Owner:** Responsible for classification
- **Data Creator:** Initial classification
- **IT Security:** Review and validation
- **Compliance:** Regulatory classification

#### Classification Steps
1. Identify data type and content
2. Assess sensitivity and value
3. Determine regulatory requirements
4. Assign classification level
5. Apply appropriate labels
6. Document classification decision

### Reclassification

#### When to Reclassify
- Change in sensitivity or value
- Regulatory requirements change
- Business context changes
- Periodic review identifies need

#### Reclassification Process
- Review current classification
- Assess new requirements
- Update classification
- Update labels and controls
- Document reclassification

## Data Handling Requirements

### Storage

#### Public Data
- Standard storage
- No encryption required
- Standard backup

#### Internal Data
- Standard storage with access controls
- Encryption recommended
- Standard backup

#### Confidential Data
- Encrypted storage required
- Access-controlled storage
- Secure backup with encryption
- Regular access reviews

#### Restricted Data
- Strong encryption required
- Highly access-controlled storage
- Secure backup with strong encryption
- Continuous monitoring
- Regular access reviews

### Transmission

#### Public Data
- Standard transmission
- No encryption required

#### Internal Data
- Encrypted transmission recommended
- Secure protocols (HTTPS, SFTP)

#### Confidential Data
- Encrypted transmission required
- Secure protocols (TLS, VPN)
- No unencrypted transmission

#### Restricted Data
- Strong encryption required
- Secure protocols only (TLS 1.2+, VPN)
- Certificate-based encryption
- No unencrypted transmission

### Access Control

#### Public Data
- Public access allowed
- No access restrictions

#### Internal Data
- Employee access
- Standard authentication
- Access logging

#### Confidential Data
- Need-to-know access
- Strong authentication
- Access logging and monitoring
- Regular access reviews

#### Restricted Data
- Strict need-to-know access
- Multi-factor authentication
- Enhanced access logging
- Continuous monitoring
- Quarterly access reviews

### Sharing and Distribution

#### Public Data
- Can be freely shared
- No restrictions

#### Internal Data
- Internal sharing allowed
- External sharing requires authorization
- Standard sharing methods

#### Confidential Data
- Internal sharing with authorization
- External sharing requires approval
- Secure sharing methods only
- Non-disclosure agreements (NDAs)

#### Restricted Data
- Sharing requires approval
- External sharing rarely permitted
- Secure sharing methods only
- NDAs required
- Document all sharing

### Retention and Disposal

#### Retention Requirements
- Based on classification and regulations
- Business requirements
- Legal and regulatory requirements
- Documented retention schedules

#### Secure Disposal

##### Public Data
- Standard deletion
- No special requirements

##### Internal Data
- Secure deletion
- Overwrite storage media

##### Confidential Data
- Secure deletion (overwrite)
- Physical destruction of media
- Certificate of destruction

##### Restricted Data
- Certified secure deletion
- Physical destruction of media
- Certificate of destruction
- Compliance with regulations
- Audit trail of disposal

## Data Labeling

### Electronic Data
- Metadata tags
- File headers/footers
- Folder labels
- Email labels
- Database field labels

### Physical Data
- Document headers/footers
- Folder labels
- Container labels
- Color coding (optional)

### Label Format
- Classification level (PUBLIC, INTERNAL, CONFIDENTIAL, RESTRICTED)
- Handling instructions
- Owner information
- Retention date (if applicable)

## Roles and Responsibilities

### Data Owner
- Classify data appropriately
- Approve access requests
- Review access regularly
- Ensure compliance

### Data Custodian (IT)
- Implement security controls
- Manage access controls
- Monitor data access
- Maintain systems

### Data Users
- Follow handling requirements
- Protect data in their possession
- Report incidents
- Comply with policies

### IT Security
- Review classifications
- Implement security controls
- Monitor compliance
- Provide guidance

## Training and Awareness

### Requirements
- Data classification training for all staff
- Role-specific training
- Regular awareness updates
- Handling procedures training

## Compliance

### Regulatory Requirements
- GDPR (EU data protection)
- HIPAA (health information)
- PCI-DSS (payment card data)
- Industry-specific regulations

### Compliance Measures
- Classification aligned with regulations
- Appropriate protection measures
- Audit trail maintenance
- Regular compliance reviews

## Monitoring and Auditing

### Access Monitoring
- Monitor access to classified data
- Log all access attempts
- Detect unauthorized access
- Regular access reviews

### Compliance Audits
- Regular classification audits
- Handling procedure reviews
- Access control audits
- Regulatory compliance audits

## Incident Response

### Data Breach
- Follow [Incident Response Policy](./incident-response-policy.md)
- Immediate containment
- Assess impact based on classification
- Regulatory notifications if required
- Document and remediate

## Related Policies

- [Security Policy](./security-policy.md)
- [Access Control Policy](./access-control-policy.md)
- [Backup and Recovery Policy](./backup-recovery-policy.md)
- [Incident Response Policy](./incident-response-policy.md)

## Classification Decision Tree

1. **Is data intended for public disclosure?**
   - Yes → **PUBLIC**
   - No → Continue

2. **Does data contain PII, PHI, PCI, or trade secrets?**
   - Yes → **RESTRICTED**
   - No → Continue

3. **Would unauthorized disclosure cause significant harm?**
   - Yes → **CONFIDENTIAL**
   - No → **INTERNAL**

---

*Note: This decision tree is a guide. Always consider regulatory requirements and business context when classifying data.*

