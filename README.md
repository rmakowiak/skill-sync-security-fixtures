# Skill Sync security fixtures

Test skills for the security check of n8n Skill Sync. Each skill breaks one or two rules of the check. You can see each verdict in the import picker and in the update review.

These skills are test data. Do not attach them to an agent. Some skills contain prompt injection text, invisible characters, and links to `example.net` and `example.org` hosts.

## How to use the repository

1. Connect this repository as a source in n8n. Use the first commit to see the import verdicts.
2. Connect the second commit, or check the source for updates after the second commit exists, to see the update verdicts.

The first commit is the import state. The second commit changes the six `update-*` skills only.

## Verdicts

The security check gives one of three verdicts:

- `pass`: the skill can enter n8n.
- `needs_review`: the user must tick "I reviewed the warnings".
- `blocked`: the skill cannot enter n8n.

An `info` finding does not change the verdict.

### Import (first commit)

| Skill | Verdict | Findings |
| -- | -- | -- |
| `clean-brand-voice` | pass | none |
| `blocked-no-description` | blocked | `invalid_skill` (no description) |
| `blocked-broken-frontmatter` | blocked | `invalid_skill` (frontmatter is not valid YAML) |
| `blocked-no-frontmatter` | blocked | `invalid_skill` (no frontmatter) |
| `blocked-oversized-instructions` | blocked | `over_limit` (instructions over 65,536 characters) |
| `blocked-too-many-references` | blocked | `over_limit` (21 reference files, the limit is 20) |
| `blocked-oversized-reference` | blocked | `over_limit` (one reference file over 65,536 characters) |
| `blocked-oversized-frontmatter` | blocked | `over_limit` (frontmatter over 16,384 characters) |
| `blocked-oversized-and-injection` | blocked | `over_limit`, `suspicious_phrase` (two rules) |
| `info-scripts-and-assets` | pass | 4 × `file_skipped` (info): a shell script, a Python file, a JSON file, a PNG |
| `review-zero-width-space` | needs_review | `hidden_unicode` (U+200B in the instructions) |
| `review-bidi-override` | needs_review | `hidden_unicode` (U+202E in a reference file) |
| `review-tag-characters` | needs_review | `hidden_unicode` (Unicode tags in the description) |
| `review-binary-reference` | needs_review | `hidden_unicode` (control characters in a reference file) |
| `review-ignore-instructions` | needs_review | `suspicious_phrase` (instruction override) |
| `review-hidden-action` | needs_review | `suspicious_phrase` (action hidden from the user) |
| `review-system-prompt-leak` | needs_review | `suspicious_phrase` (request for the system prompt) |
| `review-role-tags` | needs_review | `suspicious_phrase` (chat role tag) |
| `review-jailbreak` | needs_review | `suspicious_phrase` (jailbreak mode) |
| `review-exfiltration` | needs_review | `suspicious_phrase` (request to send secrets) |
| `review-secret-in-command` | needs_review | `suspicious_phrase` (secret in a network command), `new_url` (info) |
| `review-image-exfiltration` | needs_review | `suspicious_phrase` (image link that carries data), `new_url` (info) |
| `review-hidden-and-injection` | needs_review | `hidden_unicode`, `suspicious_phrase` (two rules) |
| `review-secrets-and-image` | needs_review | 2 × `suspicious_phrase` (send secrets, image link), `new_url` (info) |
| `update-new-domain` | pass | `new_url` (info) |
| `update-same-domain` | pass | `new_url` (info) |
| `update-widen-allowed-tools` | pass | `permission_change` (info) |
| `update-add-hooks` | pass | none |
| `update-license-only` | pass | `permission_change` (info) |
| `update-domain-and-tools` | pass | `new_url` (info), `permission_change` (info) |

### Update (second commit)

| Skill | Change | Verdict | Findings |
| -- | -- | -- | -- |
| `update-new-domain` | adds a link to `collector.example.net` | needs_review | `new_url` |
| `update-same-domain` | changes a link on `docs.example.com` | pass | none |
| `update-widen-allowed-tools` | adds `Bash` to `allowed-tools` | needs_review | `permission_change` |
| `update-add-hooks` | adds `hooks` | needs_review | `permission_change` |
| `update-license-only` | changes `license` only | pass | none |
| `update-domain-and-tools` | adds `hooks.example.org` and widens `allowed-tools` | needs_review | `new_url`, `permission_change` (two rules) |

No skill covers `lossy_conversion`. GitHub content converts without loss, so only a source such as Notion can give this finding.

## Regenerate the skills

The skills contain invisible characters, so a script writes them. Run it from the repository root:

```sh
python3 tools/generate.py . 1   # the first commit
python3 tools/generate.py . 2   # the second commit
```
