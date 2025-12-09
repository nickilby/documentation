# Password Policy

**Version:** 1.0  
**Last Updated:** 2025  
**Next Review Date:** 2026-01-01  
**Owner:** IT Security Team

## Purpose

This policy establishes requirements for creating, managing, and protecting passwords to ensure the
security of user accounts and organizational systems.

## Scope

This policy applies to all user accounts, service accounts, and systems that use password-based
authentication. This includes employees, contractors, vendors, and any third-party users with system
access.

## Policy Statement

All passwords must meet complexity requirements, be protected from unauthorized disclosure, and be changed
regularly. Password sharing is prohibited. Multi-factor authentication (MFA) is required for privileged
accounts and sensitive systems.

## Password Requirements

### Password Complexity

#### Minimum Requirements

- **Length:** Minimum 12 characters (14+ for privileged accounts)
- **Character Types:** Must include at least 3 of the following:
  - Uppercase letters (A-Z)
  - Lowercase letters (a-z)
  - Numbers (0-9)
  - Special characters (!@#$%^&*()_+-=[]{}|;:,.<>?)

#### Prohibited Elements

- Dictionary words
- Personal information (name, username, date of birth)
- Common patterns (12345, qwerty, password)
- Repeating characters (aaa, 111)
- Sequential characters (abc, 123)

### Password Strength

#### Strong Password Characteristics

- Long and complex
- Unique and unpredictable
- Not reused across systems
- Not shared with others

#### Password Examples (Do Not Use)

- ❌ `Password123` (too common)
- ❌ `CompanyName2024` (contains company name)
- ❌ `JohnDoe123!` (contains personal information)
- ✅ `Tr7#mK9$pL2@vN4!` (strong, random)

## Password Management

### Password Creation

#### User Responsibilities

- Create strong, unique passwords
- Use password manager when possible
- Never share passwords
- Report suspected compromise

#### System Requirements

- Enforce complexity requirements
- Prevent password reuse
- Implement password history (last 12 passwords)
- Provide password strength indicator

### Password Storage

#### Prohibited Practices

- ❌ Writing passwords down in plain text
- ❌ Storing in unencrypted files
- ❌ Sharing via email or chat
- ❌ Using same password for multiple accounts

#### Recommended Practices

- ✅ Use password manager
- ✅ Encrypted storage if written down
- ✅ Secure password sharing tools (if necessary)
- ✅ Unique passwords per account

### Password Sharing

#### Prohibition

- Passwords must never be shared
- No shared accounts (except service accounts)
- No password disclosure to unauthorized persons
- No password sharing via insecure channels

#### Exceptions

- Service accounts with documented procedures
- Emergency access with proper authorization
- Secure password reset procedures

## Password Expiration and Rotation

### Standard User Accounts

- **Expiration:** 90 days
- **Warning:** 14 days before expiration
- **Grace Period:** 7 days after expiration
- **History:** Cannot reuse last 12 passwords

### Privileged Accounts

- **Expiration:** 60 days
- **Warning:** 14 days before expiration
- **No Grace Period:** Immediate lockout
- **History:** Cannot reuse last 24 passwords

### Service Accounts

- **Expiration:** 180 days (or per risk assessment)
- **Rotation:** Automated where possible
- **Documentation:** All service account passwords documented
- **Secure Storage:** Encrypted password vault

### Password Reset

- Users can reset passwords through self-service portal
- Identity verification required
- Temporary passwords expire after first use
- Force password change on first login

## Multi-Factor Authentication (MFA)

### MFA Requirements

#### Mandatory MFA

- All administrative/privileged accounts
- Remote access (VPN, remote desktop)
- Cloud services and SaaS applications
- Systems handling sensitive data
- Financial systems

#### MFA Methods

- Hardware tokens (preferred for high security)
- Mobile authenticator apps (TOTP)
- SMS (acceptable but less secure)
- Biometric authentication (where supported)

### MFA Implementation

- Enforce MFA for required accounts
- Provide user training
- Support multiple MFA methods
- Backup authentication methods

## Account Security

### Account Lockout

- **Lockout Threshold:** 5 failed attempts
- **Lockout Duration:** 30 minutes (or until reset by administrator)
- **Progressive Delays:** Increasing delay after failed attempts
- **Alert:** Notify user and security team

### Suspicious Activity

- Monitor for unusual login patterns
- Alert on login from new locations
- Detect brute force attempts
- Automatic account protection

### Password Compromise

- Immediate password reset required
- Review account activity
- Check for unauthorized access
- Report to security team

## Password Recovery

### Self-Service Password Reset

- Secure identity verification
- Multiple verification methods
- Account recovery questions (if used, must be strong)
- Email or SMS verification

### Administrative Password Reset

- Identity verification required
- Secure communication channel
- Temporary password with forced change
- Document reset reason

## Service Accounts

### Service Account Requirements

- Unique accounts (no shared passwords)
- Strong, complex passwords
- Regular password rotation
- Documented and tracked

### Service Account Management

- Maintain inventory of service accounts
- Document purpose and owner
- Regular review and cleanup
- Secure password storage

## Password Managers

### Approved Password Managers

- Enterprise password manager (if provided)
- Commercial password managers (approved list)
- Browser password managers (for non-sensitive accounts)

### Password Manager Security

- Master password must meet policy requirements
- Enable MFA for password manager
- Regular backups
- Secure synchronization

## Training and Awareness

### User Training

- Password policy requirements
- How to create strong passwords
- Password manager usage
- Security best practices

### Regular Communications

- Password policy reminders
- Security awareness updates
- Phishing prevention
- Incident examples

## Compliance and Enforcement

### Policy Compliance

- Automated enforcement where possible
- Regular compliance audits
- User education
- Disciplinary action for violations

### Violations

- Sharing passwords: Immediate account lockout and review
- Weak passwords: Forced password change
- Policy violations: Disciplinary action per HR policy

## Technical Controls

### System Configuration

- Enforce password complexity
- Implement password history
- Configure account lockout
- Enable password expiration
- Log password-related events

### Monitoring

- Monitor failed login attempts
- Detect password sharing
- Alert on suspicious activity
- Regular password audits

## Related Policies

- [Access Control Policy](./access-control-policy.md)
- [Security Policy](./security-policy.md)
- [Remote Access Policy](./remote-access-policy.md)

## Password Best Practices Summary

### Do's ✅

- Use long, complex, unique passwords
- Use a password manager
- Enable MFA where available
- Change passwords if compromised
- Report suspicious activity

### Don'ts ❌

- Don't share passwords
- Don't reuse passwords
- Don't use personal information
- Don't write passwords in plain text
- Don't use dictionary words
