# WB-TOTP (Word-Based Time-Based One-Time Password)

**WB-TOTP** is a human-friendly alternative to the standard TOTP (RFC 6238) that generates **time-based one-time passwords using readable words** instead of numeric codes.

> Use the same time-based HMAC logic as TOTP  
> Use tokens like `moon-coffee` as outputs that are easier to say, hear, and remember

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

1. A **unique dictionary** (list of words) is generated with 100 or more simple words, in a **unique ordered list**. This dictionary is **encoded in base32**, creating a compact and secure string.

Example:
```
// Dictionary of simple words

"cat, dog, book, sun, tree, car, milk, ball, hat, fish"

// Encoded dictionary by base32

ImNhdCwgZG9nLCBib29rLCBzdW4sIHRyZWUsIGNhciwgbWlsaywgYmFsbCwgaGF0LCBmaXNoIg==

```

2. A WB-TOTP URI is created containing the encoded dictionary and relevant configuration parameters. This URI is shared with the other user, who stores the dictionary locally and securely.

Once both users have the same dictionary, they can independently generate synchronized word tokens based on time.


### ⬇️ Verify the identity

Once both users have the same secret dictionary stored, verifying someone’s identity becomes a simple, secure process. The verification is stateless, synchronized by time, and uses human-readable tokens.

#### How tokens are generated

To generate a token, both users independently follow the same deterministic steps:

1. Decode the dictionary
The base32-encoded dictionary string is decoded into a list of words (in a specific order).

```
// From base32

ImNhdCwgZG9nLCBib29rLCBzdW4sIHRyZWUsIGNhciwgbWlsaywgYmFsbCwgaGF0LCBmaXNoIg==

// To dictionary

"cat, dog, book, sun, tree, car, milk, ball, hat, fish"
```

2. Get current timestamp
Compute the current time interval based on the period (default: 30 seconds).

```
timecode = floor(currentUnixTime / period)
```

3. Generate HMAC hash
Use the decoded dictionary (as raw data) as the key.
Generate a HMAC using the configured algorithm (default: SHA1) with timecode as the message:

```
hash = HMAC-SHA1(key: dictionaryBytes, message: timecode)
```

4. Extract word indexes
Derive indexes from the hash to select words from the dictionary.
If the same index repeats, the second index is incremented (+1) to avoid word duplication.

5. Return words as token
The selected words are combined as the final token:
```
Token = "cat-sun"
```
Because the input values (dictionary, time, algorithm, period) are exactly the same on both devices, the output token will match — without any online communication.

---

## Dictionary Generation

This is guidance on how to generate the dictionary of words, you can do another approach always following these rules. 

Given two users want to connect, they:

1. Randomly select 100 words or more.

```
// Original dictionary with 255 simple words

"cat, dog, book, sun, tree, car, milk, ball, hat, fish, ..., crown" (255)

// Select 100 random words from the base dictionary

"apple, hat, bird, shoe, house, water, book, dog, lamp, ..., cat" (100)
```

2. Apply a random but deterministic order.

```
"lamp, bird, apple, house, dog, shoe, cat, water, hat, ..., book"
```

3. Encode it in Base32 to generate the dictionary secret.

```
// Base32-encoded dictionary

ImNhdCwgZG9nLCBib29rLCBzdW4sIHRyZWUsIGNhciwgbWlsaywgYmFsbCwgaGF0LCBmaXNoLCBsYW1wLCBiaXJkLCBhcHBsZSwgaG91c2UsIGRvZywgc2hvZSwgY2F0LCB3YXRlciwgaGF0LCBib29rLCAiY2F0LCBkb2csIGJvb2ssIHN1biwgdHJlZSwgY2FyLCBtaWxrLCBiYWxsLCBoYXQsIGZpc2gsIGxhbXAsIGJpcmQsIGFwcGxlLCBob3VzZSwgZG9nLCBzaG9lLCBjYXQsIHdhdGVyLCBoYXQsIGJvb2ssICJjYXQsIGRL==
```

Each user pair generates a unique dictionary, resulting in personalized tokens. This keeps each user-to-user channel secure and personalized.

---

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

#### Recommendations

Sharing information via QR code is better than using the URI directly, since the base32 hash generated can be very large and complex to share as a link or text.

---

### 🧪 Example
Two users use the following WB-TOTP URI:

```
otpauth://wbtotp/Pablo?secret=ImNhdCwgZG9nLCBib29rLCBzdW4sIHRyZWUsIGNhciwgbWlsaywgYmFsbCwgaGF0LCBmaXNoIg==&issuer=David&algorithm=SHA1&words=2&period=30
```

Then, at time 12:00:00, both generate the token:

```
"cat-sun"
```

At 12:00:31, the token changes automatically:

```
"fish-car"
```

---

## Key Features

🔐 Secure: Built on HMAC with support for SHA1/SHA256/SHA512

⏱ Time-synced: Just like TOTP, tokens are time-dependent

🧠 Human-optimized: Pronounceable, understandable, memorable

📱 Works on phones, CLI tools, and offline devices

♿ Boosts accessibility in real-world verbal interactions

---

## Comparison with others 2FA methods

|Feature|TOTP/HOTP|WB-TOTP|
|---|---|---|
|Output|Numeric (e.g., 932871)|Words (e.g., sun-coffee)|
|Human-friendly|Low|High|
|Use in conversations|Difficult|Natural|
|Usability|Technical|Accessible & verbal|
|Security|Strong|Strong|

---

## Implementations

TBD

---

## Based on

[RFC 6238 - Time-Based One-Time Passwords](https://datatracker.ietf.org/doc/html/rfc6238)

[Google Authenticator URI format](https://github.com/google/google-authenticator/wiki/Key-Uri-Format)

