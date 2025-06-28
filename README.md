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

## Security Considerations and Threats

WB-TOTP leverages the same cryptographic foundations as standard TOTP (RFC 6238), but introduces new security and usability trade-offs due to its word-based, human-verifiable nature.

This section outlines the main threats, attack vectors, and recommended mitigations tailored to WB-TOTP.

### Threat Vectors

#### 1. Weak or Predictable Dictionaries
If the dictionary:

- Is too short (e.g. < 100 words),

- uses predictable ordering (alphabetical),

- or comes from a known/common wordlist (e.g. a public demo),

...then the resulting tokens will have low entropy, leading to:

- Repeated or predictable word combinations

- Higher risk of token collisions

- Offline brute-force attacks if the attacker obtains the dictionary

##### Recommendations:

- Use ≥100 unique words randomly selected from a larger pool

- Ensure randomized word order

- Never reuse the same dictionary across different users or sessions

#### 2. Insecure Dictionary Distribution
If the dictionary (in the URI secret field or QR code) is shared over insecure channels like email or unencrypted messaging, attackers may intercept it and generate valid tokens.

##### Recommendations:

- Share dictionaries only over secure, end-to-end encrypted channels

- Prefer in-person QR scanning or secure key exchange

- Treat the dictionary as you would treat a password or TOTP seed

#### 3. Dictionary Reuse
Reusing the same dictionary across multiple users or sessions enables token correlation and replay attacks, since the same words may be regenerated across interactions.

##### Recommendations:

- Generate a unique dictionary per pair or session

- Rotate dictionaries regularly for long-term trust relationships

#### 4. Time Drift and Desync
As with TOTP, WB-TOTP depends on accurate clock synchronization. If client and verifier are more than one time-step apart, tokens will fail to match.

##### Recommendations:

- Ensure NTP or OS-level clock sync

- Optionally allow a ±1 time-step tolerance in low-trust cases

---

## Key Features

🔐 Secure: Built on HMAC with support for SHA1/SHA256/SHA512

⏱ Time-synced: Just like TOTP, tokens are time-dependent

🧠 Human-optimized: Pronounceable, understandable, memorable

📱 Works on phones, CLI tools, and offline devices

♿ Boosts accessibility in real-world verbal interactions

### Entropy and Token Strength
In WB-TOTP, the strength of a token depends on two factors:

- The size and uniqueness of the dictionary

- The number of words used per token

Each token represents a selection of n words chosen deterministically based on a secure HMAC. The larger the dictionary and the more words included in each token, the higher the entropy — which directly reduces the risk of collisions or offline brute-force attacks.

#### Estimating Entropy

Token entropy can be estimated as:
```
Entropy ≈ number of words × log₂(dictionary size)
```

For example:

|Dictionary Size|2-word Token|3-word Token|4-word Token|
|---|---|---|---|
|100 words|~13.3 bits|~20 bits|~26.6 bits|
|256 words|~16 bits|~24 bits|~32 bits|
|512 words|~18 bits|~27 bits|~36 bits|

Even 2-word tokens can be secure when combined with time-based expiration and human verbal validation. However, adding a third or fourth word provides stronger resistance against offline analysis or dictionary leaks — especially when tokens are used in higher-value or longer-lived authentication contexts.

### Entropy Comparison: WB-TOTP vs Standard TOTP

|Token Type|Dictionary Size|Number of Words / Digits|Entropy per Word / Digit (bits)|Total Entropy (bits)|Notes|
|---|---|---|---|---|---|
|WB-TOTP (2 words)|100|2 words|6.64|13.28|Lower than standard TOTP but usable with verbal validation and time sync|
|WB-TOTP (3 words)|100|3 words|6.64|19.92|Close to entropy of 6-digit TOTP|
|Standard TOTP (6 digits)|10 (digits 0-9)|6 digits|3.32|19.93|Common 6-digit numeric code|

#### Key points:

- A 2-word token from a 100-word dictionary provides around 13 bits of entropy, which is less than the typical 6-digit TOTP's ~20 bits.

- However, WB-TOTP tokens are intended for human verbal verification with a short validity window, which significantly reduces the risk of brute force attacks.

- Increasing the token length to 3 words brings entropy close to the 6-digit TOTP level (~20 bits).

- Expanding the dictionary size beyond 100 words or using 4+ words per token can increase entropy further, offering stronger security when needed.

This balance allows WB-TOTP to remain user-friendly and easy to communicate verbally while still providing a meaningful level of security for time-sensitive authentication.

---

## Accessibility and Localization

To maximize the effectiveness of WB-TOTP in diverse real-world scenarios, special attention should be given to accessibility and localization of the word dictionary.

### Language and Cultural Adaptation

- **Localized Wordlists:** Generate word dictionaries tailored to the users’ native languages and cultural context. This ensures that words are easy to pronounce, remember, and understand, reducing errors during verbal communication.

- **Phonetic Simplicity:** Select words that are phonetically distinct and have minimal risk of confusion with similar-sounding words in the target language, especially in noisy or low-quality audio environments.

- **Avoid Ambiguous or Offensive Terms:** Ensure the dictionary excludes words that might be ambiguous, confusing, or culturally sensitive to maintain trust and clarity.

### Accessibility Considerations

- **Pronunciation for People with Speech or Hearing Impairments:** Consider alternative wordlists or token generation methods that accommodate users with speech impediments or hearing difficulties, such as simpler syllables or providing visual backup options.

- **Support for Multilingual Users:** For users fluent in multiple languages, allow switching dictionaries or combining words from multiple languages that are mutually understandable to the communicating parties.

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

