# PDF Password Cracking — John the Ripper (JtR) Walkthrough

**Course:** Networkwalks Cybersecurity Training — Week 3: Project Modules
**Module:** W3-PM1 — Password Cracking with JTR
**Category:** Password Cracking / Offline Attacks
**Target:** Password-protected PDF file (`My-Locked-PDF3.pdf`)

## Objective

Recover the password protecting a locked PDF file by extracting its password hash and running a dictionary (wordlist) attack against it using John the Ripper's hash format.

## Tools Used

- **pdf2john / pdf2hashcat** — extracts a crackable hash from an encrypted PDF
- **John the Ripper (JtR)** — offline password cracking tool
- **Wordlist-based dictionary attack** — common password list

## Methodology

### 1. Extract the hash from the locked PDF

The target PDF (`My-Locked-PDF3.pdf`) was run through a PDF-to-hash converter that implements the same logic as `pdf2john`/`pdf2hashcat`. This produces a hash string in the `$pdf$` format that JtR/Hashcat can process.

Extracted hash:
```
$pdf$4*4*128*-1028*1*16*34eb542eff4e1b0b32d25ce15a9a7281*32*b77872bfc9a24fb2f845066283a8fc1b0021446990b9e4114071a4d9104984c1*32*e7572256e4b552cd57988f5134214b91920d94d7a6bf550ea94a2995c7f2ab02
```

The hash format breaks down (per the `$pdf$` scheme) into: PDF version/revision info, key length, permission flags, algorithm revision, and the owner/user encryption keys needed to validate password guesses offline.

### 2. Run a dictionary attack

With the hash isolated, a wordlist attack was run — iterating through a list of common/weak passwords and checking each candidate against the hash until a match was found.

Sample of the attack in progress:
```
[-] Trying: login        X
[-] Trying: starwars     X
[-] Trying: 121212       X
[-] Trying: bailey       X
[-] Trying: freedom      X
[-] Trying: shadow       X
[-] Trying: master       X
[-] Trying: 666666       X
[-] Trying: superman     X
[+] MATCH: 1qaz2wsx      ✓
```

- Attempts: 35 / 100 candidate passwords tried
- Speed: ~3 passwords/sec
- Result: match found at password **`1qaz2wsx`** (a classic "keyboard-walk" password — an easy-to-type pattern that still appears constantly in breach wordlists)

### 3. Validate

The recovered password (`1qaz2wsx`) was used to unlock the original PDF, confirming the crack.

## Result

| Item | Value |
|---|---|
| Hash format | `$pdf$` (pdf2john/pdf2hashcat compatible) |
| Attack type | Dictionary / wordlist attack |
| Candidates tried before match | 35 |
| Cracked password | `1qaz2wsx` |

## Flags Captured

```
nw{networkwalks_persistence_jtr_270521}
nw{networkwalks_flag_260821_1}
```

## Key Takeaways

- **Weak passwords are the bottleneck, not the crypto.** The PDF encryption itself wasn't broken — the password behind it was simply guessable because it followed a common keyboard pattern.
- **Dictionary attacks are fast against predictable passwords.** A relatively small wordlist (100 candidates) was enough to recover this password in 35 attempts.
- **Patience + the right wordlist** matters more than raw compute power for weak/reused passwords — targeted wordlists (common passwords, leaked credential lists, keyboard-walk patterns) dramatically improve crack speed over brute force.
- This exercise reinforces the mindset of a security tester: understand *why* a password is weak (predictability, pattern, reuse) rather than just that a tool cracked it.

## Disclaimer

This exercise was performed in a controlled training environment (Networkwalks Cybersecurity course) against a file created specifically for the exercise. Password cracking should only ever be performed on systems/files you own or have explicit authorization to test.
