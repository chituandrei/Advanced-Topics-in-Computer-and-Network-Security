# Quantum-Safe Transport: Migrating TLS, HTTPS, QUIC and SSH to Post-Quantum Cryptography

**Course:** Advanced Topics in Computer and Network Security (Master, Year 2)
**General theme:** Transport Layer Security: SSL, TLS, SSH, HTTPS
**Selected topic:** Post-quantum migration of transport-layer security protocols

## Team

| Member |
|--------|
| **Costin Bosoaga** |
| **Stefan Anton**  |
| **Luca Patrascu** |
| **Andrei Chitu** |


## Motivation

Almost all secure communication on the Internet relies on public-key cryptography. TLS (HTTPS), QUIC and SSH use ECDHE/X25519 for key exchange and RSA/ECDSA for authentication. Shor's algorithm would break all of these on a large enough quantum computer.

The threat is already relevant today because of **"harvest now, decrypt later"**: an adversary who records encrypted traffic now could decrypt it years from now. Migrating the Internet takes many years, so it has already started:

- **August 2024:** NIST published the first PQC standards: **ML-KEM** (FIPS 203), **ML-DSA** (FIPS 204) and **SLH-DSA** (FIPS 205).
- **Browsers and CDNs:** Chrome, Firefox and Cloudflare use the hybrid **X25519MLKEM768** key exchange by default.
- **Libraries:** OpenSSL 3.5 ships ML-KEM, ML-DSA and hybrid key exchange.
- **SSH:** OpenSSH moved to hybrid PQ key exchange by default (`sntrup761x25519-sha512` in 9.0, `mlkem768x25519-sha256` in 10.0).

Post-quantum cryptography is not a drop-in replacement, though:

| Component | Classical | Post-quantum |
|-----------|-----------|--------------|
| Key exchange public key | X25519: 32 B | ML-KEM-768: 1184 B |
| Signature | ECDSA P-256: ~64 B | ML-DSA-65: 3309 B |

Larger handshakes split the ClientHello across packets, which has broken some servers and middleboxes. They also cost extra round trips and are a real problem for constrained devices. The algorithms themselves are young: **SIKE**, a NIST finalist candidate, was broken in 2022 on a single laptop, and **KyberSlash** (2024) showed timing leaks in real ML-KEM implementations.

## Research Questions

1. **Readiness:** How far has the PQ migration progressed across TLS, QUIC and SSH, both in the standards and in real deployments?
2. **Cost:** What is the measurable cost of PQ and hybrid handshakes in latency, bandwidth and CPU, and how does it change on poor networks?
3. **Authentication:** Key exchange is already migrating, but authentication is not. Why, and what are the options (ML-DSA, KEMTLS, Merkle Tree Certificates)?
4. **Robustness:** What new risks does the migration introduce, such as implementation side channels, downgrade attacks on hybrid negotiation and middlebox ossification?
5. **Protocol comparison:** How do TLS 1.3, QUIC and SSH each handle the transition, and which design makes migration easiest (crypto-agility)?

## Milestones and Grading

Final grade = **(2·M1 + M2 + M3) / 4**

| Milestone | Weight | Content | Deadline |
|-----------|--------|---------|----------|
| **M1**: Overview | 50% | State of the art, comparison of solutions and implementations, personal comments. Evaluated weekly during classes. Includes a presentation by all team members. | 01.12.2026 |
| **M2**: Improvements | 25% | Proposed improvements to the topic or to a particular case. | 12.12.2026 |
| **M3**: Scientific paper | 25% | Full paper combining M1 and M2, plus implementation details and results. | End of semester (before the exam) |

- A milestone is gradable only if a written paper exists.
- M2 and M3 are graded only after M1 has been graded.
- Each paper is uploaded to the course forum by one team member. Hard deadline: 11.01.2026, as stated in the course description. This is most likely a typo for 11.01.2027, so confirm it with the professor.

## Plan

### M1: Overview

| Week | Topic | Lead |
|------|-------|------|
| 1 | Topic introduction, motivation, the quantum threat | All |
| 2 | TLS 1.3, HTTPS and QUIC handshakes; NIST PQC standards (ML-KEM, ML-DSA, SLH-DSA) | Costin, Stefan |
| 3 | Hybrid key exchange in TLS and QUIC (X25519MLKEM768); PQ in SSH | Costin, Luca |
| 4 | PQ authentication and certificates; implementation comparison (OpenSSL 3.5 / oqs-provider, BoringSSL, rustls, wolfSSL, OpenSSH) | Luca, Andrei |
| 5 | Attacks and pitfalls: SIKE break, side channels (KyberSlash), downgrade, middlebox issues | Stefan, Andrei |
| 6 | Deployment status, comparison tables, personal conclusions (before 01.12.2026) | All |

### M2: Improvements (candidate directions)
- **Smaller authentication:** KEMTLS (KEM-based authentication instead of handshake signatures), certificate compression, Merkle Tree Certificates
- **Constrained environments:** choosing PQ parameter sets for IoT devices and lossy or high-latency links (e.g. mobile or satellite)
- **Crypto-agility:** a set of recommendations or a configuration checklist for migrating an organization's TLS and SSH infrastructure

### M3: Implementation and Evaluation

**Experiment 1: Lab benchmark**
- Testbed: OpenSSL 3.5 + oqs-provider, OpenSSH 10, Linux network emulation (`tc netem`)
- Compare classical vs. hybrid vs. pure PQ handshakes for TLS 1.3 and SSH
- Metrics: handshake latency, bytes on the wire, number of packets, CPU time
- Conditions: varying packet loss, delay and bandwidth

**Experiment 2: Internet adoption scan**
- Scan the top websites from the Tranco list for X25519MLKEM768 support
- Scan a sample of public SSH servers for PQ key exchange support
- Report adoption by hosting provider, CDN and server software

**Experiment 3: Compatibility and robustness (optional)**
- Reproduce the effect of large, fragmented ClientHello messages on servers and middleboxes
- Inspect hybrid handshakes in Wireshark and analyze downgrade behavior

## Repository Structure (planned)

```
.
├── README.md
├── M1-overview/        # overview paper + weekly presentations
├── M2-improvements/    # improvement proposals paper
├── M3-paper/           # final scientific paper (LaTeX)
├── experiments/
│   ├── benchmark/      # lab testbed scripts and configs
│   ├── scan/           # Internet adoption scanner
│   └── results/        # raw data and plots
└── references/         # bibliography (BibTeX) and notes
```

## References

1. E. Rescorla, *The Transport Layer Security (TLS) Protocol Version 1.3*, RFC 8446, 2018.
2. J. Iyengar, M. Thomson, *QUIC: A UDP-Based Multiplexed and Secure Transport*, RFC 9000, 2021.
3. NIST, *FIPS 203: Module-Lattice-Based Key-Encapsulation Mechanism Standard*, 2024.
4. NIST, *FIPS 204: Module-Lattice-Based Digital Signature Standard*, 2024.
5. NIST, *FIPS 205: Stateless Hash-Based Digital Signature Standard*, 2024.
6. D. Stebila, S. Fluhrer, S. Gueron, *Hybrid key exchange in TLS 1.3*, IETF draft-ietf-tls-hybrid-design.
7. K. Kwiatkowski, P. Kampanakis, B. Westerbaan, D. Stebila, *Post-quantum hybrid ECDHE-MLKEM Key Agreement for TLSv1.3*, IETF draft-ietf-tls-ecdhe-mlkem.
8. C. Paquin, D. Stebila, G. Tamvada, *Benchmarking Post-Quantum Cryptography in TLS*, PQCrypto 2020.
9. P. Schwabe, D. Stebila, T. Wiggers, *Post-Quantum TLS Without Handshake Signatures*, ACM CCS 2020.
10. D. Sikeridis, P. Kampanakis, M. Devetsikiotis, *Post-Quantum Authentication in TLS 1.3: A Performance Study*, NDSS 2020.
11. W. Castryck, T. Decru, *An Efficient Key Recovery Attack on SIDH*, EUROCRYPT 2023.
12. D. J. Bernstein et al., *KyberSlash: Exploiting secret-dependent division timings in Kyber implementations*, 2024.
13. Open Quantum Safe project: https://openquantumsafe.org
