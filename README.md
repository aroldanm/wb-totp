# WB-TOTP (Word-Based Time-Based One-Time Password)

**WB-TOTP** is a human-friendly alternative to the standard TOTP (RFC 6238) that generates **time-based one-time passwords using readable words** instead of numeric codes.

> ✅ Uses the same time-based HMAC logic as TOTP  
> ✅ But outputs tokens like `moon-coffee` that are easier to say, hear, and remember

---

## Why WB-TOTP?

### 1. Strong cryptography, better usability

WB-TOTP is based on secure cryptography (HMAC + time sync), but optimized for human interaction:

- Easier to pronounce, share, and recall
- Ideal for voice-based or informal settings
- Reduces the chance of miscommunication and human error

### 2. Protects against voice impersonation

With modern AI, anyone's voice can be cloned. Trusting a voice alone is no longer secure:

- WB-TOTP adds a verifiable, time-based verbal token
- Only someone with the shared secret can generate the right words
- Even perfect voice impersonators can’t guess the token

---

## Use Cases

### 📞 Voice Authentication

- Someone calls you pretending to be a known contact.
- You ask for the current word token to verify their identity.
- You verify that the code they share is the same one you generated on your end.

### 🤝 Peer-to-Peer Verification

- Two people who don't know each other verify their identity using a spoken token.
- If both have the same set of words, the identity is verified.

### 📦 Package Delivery Verification

- A courier arrives claiming to deliver an item.
- Request the WB-TOTP token (shared when placing the order).
- They verify the code before delivering the package.

---

## How it works

### ⬆️ Generate the secret

Before two users can verify each other using WB-TOTP, they must share a common dictionary of words:

1. A **unique dictionary** (list of words) is generated with 100 or more simple words, in an **unique ordered list**. This dictionary is **encoded in base32**, creating a compact and secure string.

Example:
```
// Dictionary of simple words

"cat, dog, book, sun, tree, car, milk, ball, hat, fish"

// Encoded dictionary by base32

ImNhdCwgZG9nLCBib29rLCBzdW4sIHRyZWUsIGNhciwgbWlsaywgYmFsbCwgaGF0LCBmaXNoIg==

```

3. A WB-TOTP URI is created containing the encoded dictionary and relevant configuration parameters. This URI is shared with the other user, who stores the dictionary locally and securelly.

Once both users have the same dictionary, they can independently generate synchronized word tokens based on time.

#### URI Format

WB-TOTP extends the otpauth:// URI format with the following structure:

```
otpauth://wbtotp/[LABEL]?secret=[DICTIONARY]&issuer=[ISSUER]&algorithm=[ALGORITHM]&words=[WORDS]&period=[PERIOD]

```
| Parameter | Description | Is required |
|---|---|---|
|`LABEL`|Name or identifier of the recipient — the person whose identity will be verified|YES|
|`DICTIONARY`|A `BASE32`-encoded string containing the shared and ordered dictionary used to generate the tokens|YES|
|`ISSUER`|Name or identifier of the sender — the person who generated and shared the dictionary|YES|
|`ALGORITHM`|Hash algorithm used to generate tokens. Default: `SHA1`. Alternatives: `SHA256`, `SHA512`|NO|
|`WORDS`|Number of words per token. Default: `2`. Must be ≥1|NO|
|`PERIOD`|Duration of each token in seconds. Default: `30`. Alternatives: `60`|NO|

##### Examples

This URI represents a dictionary created by David, intended to verify the identity of Pablo, using 2 words per token, with a new token generated every 30 seconds.

```
otpauth://wbtotp/Pablo?secret=ImNhdCwgZG9nLCBib29rLCBzdW4sIHRyZWUsIGNhciwgbWlsaywgYmFsbCwgaGF0LCBmaXNoIg==&issuer=David&algorithm=SHA1&words=2&period=30
```

or short URI

```
otpauth://wbtotp/Pablo?secret=ImNhdCwgZG9nLCBib29rLCBzdW4sIHRyZWUsIGNhciwgbWlsaywgYmFsbCwgaGF0LCBmaXNoIg&issuer=David
```

#### Recomendations

Sharing information using a QR code is a better option than using the URI directly, since the base32 hash generated can be very large and complet to share as a link or text.


### ⬇️ Verify the identity

Lorem ipsum Lorem ipsum Lorem ipsum Lorem ipsum Lorem ipsum Lorem ipsum Lorem ipsum Lorem ipsum Lorem ipsum Lorem ipsum 

---
