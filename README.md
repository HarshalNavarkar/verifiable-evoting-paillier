# Verifiable Electronic Voting System with Ballot Privacy

Advanced B.Tech Cybersecurity academic prototype.

## Features
- Demo voter registration with salted password hashing (PIN verifier).
- One-vote-per-registered-voter enforcement using a database transaction.
- Paillier public-key encryption of each candidate selection.
- Ballot records stored without voter IDs.
- Public bulletin board with ciphertexts and a Merkle root.
- Receipt lookup with Merkle inclusion path.
- Admin-only demo results endpoint.
- Reset endpoint for repeatable classroom demonstrations.

## Requirements
- Python 3.10+
- `pip`

## Setup
```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate
pip install -r requirements.txt
```

Set an admin password before running:
```powershell
# Windows PowerShell
$env:VOTING_ADMIN_PASSWORD="replace-with-a-long-random-password"
python app.py
```
```bash
# macOS/Linux
export VOTING_ADMIN_PASSWORD='replace-with-a-long-random-password'
python app.py
```
Open http://127.0.0.1:5000.

The first run creates `keys.json` (private demo key) and `voting.db`. Protect both files. Do not commit them to version control. For a fresh demo, use the authenticated reset endpoint or stop the server and remove the database and keys only if you intend to create a new election/keypair.

## Demo workflow
1. Register voter IDs such as `voter001` with PINs of at least six characters.
2. Cast a vote for candidate 1, 2, or 3.
3. Save the returned `ballot_id` receipt.
4. Refresh the public board and use the receipt ID to check inclusion.
5. Set the admin password and calculate the tally.
6. Try to vote a second time with the same voter ID; the system should reject it.

## API
- `GET /api/health`
- `POST /api/demo/register` JSON: `{"voter_id":"voter001","pin":"demo-pin-123"}`
- `GET /api/public-key`
- `POST /api/vote` JSON: `{"voter_id":"voter001","pin":"demo-pin-123","choice":1}`
- `GET /api/board`
- `GET /api/verify/<ballot_id>`
- `GET /api/results` header `X-Admin-Password: ...`
- `POST /api/admin/reset` header `X-Admin-Password: ...`

## Cryptographic design
- Paillier encrypts the candidate number under a public key.
- The private key is used by the local demo tally service.
- The bulletin board hashes each `(ballot_id, ciphertext)` leaf and combines hashes into a Merkle tree.
- A receipt proves that the ballot record is included in the current board if its proof is checked against the published root.

## Important limitations (include in project viva/report)
This is a teaching prototype, not a secure election platform:
- It does not include a zero-knowledge proof that a ballot encodes exactly one valid candidate. The tally checks plaintext range after decryption.
- It decrypts ballots individually in the demo tally, which is not a fully privacy-preserving election tally.
- The identity database and ballot table are separate, but timestamps and traffic can still permit correlation.
- There is one local private key holder, no threshold trustees, and no independent public verification service.
- The Merkle root is served by the same application; a malicious operator could publish different roots to different users unless the root is independently witnessed.
- No coercion resistance, end-to-end challenge mechanism, formal verification, accessibility audit, or election certification.
- The public board exposes encrypted ballots; encryption alone does not prove ballot validity.
- The web app is for localhost only. Do not expose it to the internet or use real personal data.

## Advanced extension roadmap
1. Implement a verifier that independently recomputes the Merkle root and inclusion proofs.
2. Add a zero-knowledge range/validity proof that the encrypted vote is one of the permitted choices.
3. Replace individual decryption with homomorphic aggregation and a proof of correct decryption.
4. Use threshold Paillier key shares held by independent trustees.
5. Add signed, externally witnessed append-only roots and signed election configuration.
6. Add threat modelling using STRIDE, abuse cases, penetration tests, dependency pinning, and a reproducible build.
7. Write a privacy impact assessment and document assumptions, trust boundaries, and residual risks.

## Security hygiene
- Change the admin password.
- Keep `keys.json` secret and out of source control.
- Use HTTPS, CSRF protection, rate limits, strict input validation, secure session management, logging without ballot selections, and independent code review before any network deployment.
- Do not claim the project is suitable for real elections.
