---
tags: [reti, capitolo-8, sicurezza, crittografia, TLS, kurose-ross]
capitolo: 8
titolo: Sicurezza nelle Reti di Calcolatori
libro: "Reti di Calcolatori e Internet - Kurose & Ross (8a ed.)"
data_creazione: 2026-09-29
---

# Capitolo 8 — Sicurezza nelle Reti di Calcolatori

> [!NOTE] Proteggere la comunicazione
> La sicurezza di rete affronta quattro obiettivi fondamentali: riservatezza, integrità, autenticità e disponibilità. Questo capitolo copre crittografia, autenticazione, firme digitali, SSL/TLS, IPsec e firewall.

---

## 8.1 Cos'è la Sicurezza di Rete?

### Quattro Obiettivi

| Obiettivo | Definizione | Minaccia |
|---|---|---|
| **Riservatezza** | Solo mittente e destinatario leggono il messaggio | Intercettazione (eavesdropping) |
| **Integrità** | Il messaggio non è stato alterato in transito | Manomissione |
| **Autenticazione** | Le parti sono chi dicono di essere | Spoofing, impersonation |
| **Disponibilità** | I servizi sono accessibili | DoS, DDoS |

### Modello Attacker-Defender

**Trudy** (intruder) si trova nel mezzo tra **Alice** e **Bob**:
- **Ascolto passivo** — intercetta messaggi (eavesdrop)
- **Attacco attivo** — modifica, inietta, cancella messaggi
- **Replay** — ri-invia messaggi validi catturati in precedenza
- **DoS** — nega il servizio agli utenti legittimi

---

## 8.2 Crittografia

### Crittografia Simmetrica

Mittente e destinatario usano la **stessa chiave segreta K**:
```
Messaggio m ──[enc con K]──> Cipher c ──[dec con K]──> m
```

**Cifrari a blocchi:**
- Operano su blocchi di n bit
- Usano **S-box** (sostituzione non lineare) e permutazioni

**DES (Data Encryption Standard):**
- Blocchi da 64 bit, chiave da 56 bit
- **Insicuro** — chiave troppo corta, attacchi brute-force fattibili

**3DES:**
- Applica DES 3 volte con 3 chiavi diverse
- Sicuro ma lento

**AES (Advanced Encryption Standard):**
- Blocchi da 128 bit, chiavi da 128/192/256 bit
- Standard attuale — resistente a tutti gli attacchi noti
- Usato in WiFi WPA2/WPA3, TLS, disco cifrato

**Modalità operative:**
- **ECB** (Electronic Code Book) — ogni blocco cifrato indipendentemente. **Non sicuro** (stesso plaintext → stesso ciphertext)
- **CBC** (Cipher Block Chaining) — XOR con blocco precedente prima di cifrare. **IV** (Initialization Vector) per randomizzare
- **CTR** (Counter) — converte cifrario a blocchi in cifrario di flusso

### Crittografia Asimmetrica (a Chiave Pubblica)

Alice ha una coppia di chiavi: **K⁺_A** (pubblica) e **K⁻_A** (privata)

**Proprietà fondamentale:**
- Cifri con chiave pubblica → solo la chiave privata decifra
- Cifri con chiave privata → solo la chiave pubblica decifra
- Computazionalmente impraticabile ricavare K⁻ da K⁺

**RSA (Rivest-Shamir-Adleman):**

*Generazione chiavi:*
1. Scegli due numeri primi grandi p, q
2. n = p × q
3. e, d tali che e×d mod (p-1)(q-1) = 1
4. Chiave pubblica: (n, e) | Chiave privata: (n, d)

*Cifratura/decifrazione:*
```
Cifratura:  c = m^e mod n
Decifrazione: m = c^d mod n
```

*Sicurezza:* basata sulla difficoltà di fattorizzare n in p×q (problema NP-hard in pratica).

**RSA è lento** → usato per scambiare la chiave simmetrica (session key), poi si usa AES.

---

## 8.3 Integrità e Autenticazione dei Messaggi

### Hash Crittografici

Funzione `H(m)` con proprietà:
1. **Efficienza** — facile calcolare H(m)
2. **One-way** — impossibile trovare m dato H(m)
3. **Collision resistance** — impossibile trovare m₁≠m₂ con H(m₁)=H(m₂)

**Algoritmi:**
- **MD5** — 128 bit, **deprecato** (collisioni trovate nel 2004)
- **SHA-1** — 160 bit, **deprecato** (collisioni 2017)
- **SHA-256, SHA-3** — 256+ bit, **sicuri**

### MAC — Message Authentication Code

Garantisce **integrità + autenticazione** con chiave simmetrica condivisa:

```
Alice: MAC = H(m + s)   ← s = shared secret
       invia (m, MAC)

Bob: ricalcola H(m + s)
     se uguale al MAC ricevuto → m è integro e da Alice
```

Usato in: HMAC (standard), TLS record integrity.

### Firma Digitale

Garantisce **integrità + autenticazione + non ripudio** con crittografia asimmetrica:

**Firma:**
```
Alice calcola: sig = K⁻_A(H(m))   ← cifra l'hash con la chiave privata
Alice invia: (m, sig)
```

**Verifica:**
```
Bob decifra: H' = K⁺_A(sig)       ← usa chiave pubblica di Alice
Bob calcola: H = H(m)
se H == H' → firma valida → messaggio è di Alice, integro
```

### PKI — Public Key Infrastructure

**Problema:** come fidarsi che K⁺_A sia davvero la chiave pubblica di Alice?

**Certificato digitale (X.509):**
- Emesso da una **CA** (Certificate Authority)
- Contiene: nome soggetto, chiave pubblica, CA firmataria, validità
- Firmato digitalmente dalla CA → verificabile con la chiave pubblica della CA

**Catena di fiducia:**
```
Root CA (trust anchor pre-installato nel SO/browser)
  └── Intermediate CA (cert firmato da Root)
        └── Server cert (cert firmato da Intermediate)
```

**Certificate Revocation:**
- **CRL** (Certificate Revocation List) — lista di cert revocati
- **OCSP** (Online Certificate Status Protocol) — verifica real-time

---

## 8.4 Sicurezza al Livello Applicazione — PGP ed Email Sicura

**PGP (Pretty Good Privacy):**

Per inviare email sicura da Alice a Bob:
1. Alice genera chiave simmetrica di sessione `Ks`
2. Alice cifra il messaggio con `Ks` (AES-CBC)
3. Alice cifra `Ks` con la chiave pubblica di Bob `K⁺_B` (RSA)
4. Alice firma l'hash del messaggio con `K⁻_A`
5. Invia: `K⁺_B(Ks) || Ks(m) || K⁻_A(H(m))`

Bob: decifra `Ks` con `K⁻_B`, decifra messaggio, verifica firma.

---

## 8.5 SSL/TLS — Sicurezza al Livello Trasporto

**TLS** (Transport Layer Security) protegge la comunicazione TCP.  
Usato da HTTPS (HTTP su TLS), SMTPS, IMAPS, ecc.

### TLS Handshake (TLS 1.3)

```
Client                              Server
  │                                    │
  │──── ClientHello ───────────────────>│
  │     (versioni TLS supportate,       │
  │      cipher suites, key_share)      │
  │                                    │
  │<─── ServerHello ────────────────────│
  │<─── Certificate ────────────────────│
  │<─── CertificateVerify ──────────────│
  │<─── Finished ───────────────────────│
  │                                    │
  │──── Finished ──────────────────────>│
  │                                    │
  │═══════════ Dati cifrati ════════════│
```

**Cosa stabilisce il TLS Handshake:**
1. Versione TLS e cipher suite (es: TLS_AES_256_GCM_SHA384)
2. Scambio chiavi (ECDHE — Elliptic Curve Diffie-Hellman Ephemeral)
3. Autenticazione server (certificato)
4. **Session keys** per cifratura dei dati

**Perfect Forward Secrecy (PFS):**
- Con ECDHE, le chiavi di sessione sono effimere
- Anche compromettendo la chiave privata futura → le sessioni passate rimangono sicure

### Record TLS

```
[TLS Header | Payload cifrato con AEAD (AES-GCM o ChaCha20-Poly1305)]
```

**AEAD** (Authenticated Encryption with Associated Data) = cifratura + integrità in un'unica primitiva.

---

## 8.6 IPsec — Sicurezza al Livello Rete

**IPsec** protegge i datagrammi IP. Usato tipicamente per **VPN**.

### Due Protocolli IPsec

| Protocollo | Sigla | Funzione |
|---|---|---|
| **Authentication Header** | AH | Integrità + autenticazione (no riservatezza) |
| **Encapsulating Security Payload** | ESP | Integrità + autenticazione + **riservatezza** |

### Due Modalità

| Modalità | Uso | Come funziona |
|---|---|---|
| **Transport** | Host-to-host | Protegge solo il payload IP originale |
| **Tunnel** | Gateway-to-gateway (VPN) | Incapsula l'intero datagramma IP in un nuovo datagramma |

**VPN con Tunnel Mode:**
```
[Originale: IP_A → IP_B | Payload]
         ↓ IPsec Tunnel Mode
[Nuovo: IP_GW1 → IP_GW2 | ESP | IP_A → IP_B | Payload (cifrato)]
```

### Security Association (SA)

Prima di comunicare, le due parti stabiliscono una **SA** tramite **IKE** (Internet Key Exchange):
- Algoritmo di cifratura e chiave
- Algoritmo di autenticazione e chiave
- SPI (Security Parameter Index) per identificare la SA

---

## 8.7 Sicurezza al Livello Collegamento — WPA3

**802.11i / WPA3:**
- **SAE** (Simultaneous Authentication of Equals) — handshake basato su Diffie-Hellman, resistente a dictionary attack
- **4-way handshake** → genera PTK (Pairwise Transient Key) per cifratura unicast
- **CCMP** (AES in CCM mode) — cifratura e integrità per ogni frame

---

## 8.8 Firewall e IDS

### Firewall

Separa la rete interna (fidata) da Internet (non fidata).

**Packet filter (stateless):**
- Filtra i pacchetti in base a: IP sorg/dest, porta sorg/dest, protocollo, flag TCP
- Regole: permit/deny

```
# Esempio regole
permit tcp any 193.168.1.0/24 eq 80    # HTTP verso web server
permit tcp any 193.168.1.0/24 eq 443   # HTTPS verso web server
deny   tcp any any eq 23               # Blocca Telnet
deny   any any                          # Default deny
```

**Stateful packet filter:**
- Tiene traccia delle connessioni TCP (Connection Tracking)
- Permette il traffico di risposta solo per connessioni aperte dall'interno

**Application Gateway (proxy):**
- Lavora a livello applicazione (layer 7)
- Ispeziona il payload HTTP, FTP, ecc.
- Più lento ma più potente

### IDS/IPS

**IDS** (Intrusion Detection System) — rileva attacchi e avvisa  
**IPS** (Intrusion Prevention System) — rileva e blocca attivamente

**Tipi:**
- **Signature-based** — confronta con firme di attacchi noti (veloce, non rileva 0-day)
- **Anomaly-based** — profila il comportamento normale, rileva deviazioni (rileva 0-day, falsi positivi)

---

## Concetti Chiave

> [!IMPORTANT] Da memorizzare
> - **AES** = cifratura simmetrica standard; **RSA** = asimmetrica per scambio chiavi
> - **MAC** = integrità + autenticazione con chiave simmetrica
> - **Firma digitale** = cifra l'hash con chiave privata → non ripudio
> - **TLS** usa ECDHE per PFS + AES-GCM per dati
> - **IPsec ESP in tunnel mode** = VPN
> - **WPA3 SAE** = resistente a dictionary attack
> - **SHA-256** sicuro; **MD5 e SHA-1** deprecati

## Confronto Protocolli di Sicurezza

| Protocollo | Livello | Riservatezza | Integrità | Autenticazione |
|---|---|---|---|---|
| PGP | Applicazione | ✓ | ✓ | ✓ |
| TLS/SSL | Trasporto | ✓ | ✓ | ✓ (server) |
| IPsec (ESP) | Rete | ✓ | ✓ | ✓ |
| IPsec (AH) | Rete | ✗ | ✓ | ✓ |
| WPA3 | Collegamento | ✓ | ✓ | ✓ |

## Link Interni

- [[Capitolo 7 - Reti Wireless e Mobili]]
- [[Capitolo 1 - Reti di Calcolatori e Internet]]
