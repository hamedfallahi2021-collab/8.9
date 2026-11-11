# RiskPilot NT8 Diagnostic Center — Project Inputs

This repository is the public development workspace for the independent, read-only NinjaTrader 8 Diagnostic Center.

## Start here

1. Read [RP-DIAG-01-v2.md](prompts/RP-DIAG-01-v2.md) in full. It contains the original 907-line prompt followed by mandatory v2 overrides and execution governance.
2. [RP-DIAG-01-original.md](prompts/RP-DIAG-01-original.md) preserves the original prompt verbatim for reference.
3. Review [ARCHIVE_INVENTORY.md](reference/ARCHIVE_INVENTORY.md) for the supplied reference archives, sizes, and SHA-256 hashes.

## Repository roles

- This repository (`hamedfallahi2021-collab/8.9`) is the destination for the Diagnostic Center implementation.
- `RiskPilot.BOT-main.zip` is the supplied RiskPilot.BOT source snapshot and is reference-only. Do not modify production source through this project.
- `1.zip` is the supplied historical NinjaTrader runtime evidence archive. It is historical evidence, not a live runtime session.
- The exact Git commit represented by the source ZIP is not established unless the archive itself provides verifiable metadata.

## Important archive upload status

The two original ZIP attachments were inspected in the working session, but this GitHub integration supports UTF-8 text file creation and does not provide a binary repository-file upload operation. Therefore, **the ZIP payloads themselves have not yet been committed to this repository**. Their exact names, sizes, and SHA-256 hashes are recorded in `reference/ARCHIVE_INVENTORY.md`. Do not treat a manifest entry as proof that the archive is present in GitHub.

To complete the binary upload, add the two ZIP files to the paths specified in the inventory using Git or GitHub's web upload interface. Verify their SHA-256 hashes after upload.

## Safety

- The AddOn must remain strictly read-only.
- `Trading = Disabled` is invariant.
- No orders, positions, accounts, connections, strategies, or RiskPilot production settings may be mutated.
- Do not commit credentials, tokens, proprietary NinjaTrader DLLs, or personal secrets.
- Do not merge a PR without the owner's explicit authorization.

## Repository visibility

This repository is intentionally public. Only sanitized source/reference material may be committed. Review archives for credentials, tokens, private account identifiers, and other sensitive material before uploading them publicly.
