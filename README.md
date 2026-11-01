# cvescan/action

Scan your lockfile for known CVEs on every push and pull request. Builds a
timestamped, commit-pinned audit trail on [cvespy.com](https://cvespy.com) —
the evidence trail you show an auditor — and can fail the build on critical
findings.

The same data powers your public trust page at `cvespy.com/t/your-company`.

## Usage

```yaml
name: CVEScan
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: cvespy/action@v1
        with:
          api-key: ${{ secrets.CVESCAN_API_KEY }}
          min-severity: 'high'
```

Get an API key (`cv_...`) from [cvespy.com/dashboard](https://cvespy.com/dashboard)
and store it as the `CVESCAN_API_KEY` repository secret.

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `api-key` | yes | — | CPESpy API key (from cvespy.com/dashboard) |
| `api-url` | no | `https://api.cvespy.com` | API base URL |
| `min-severity` | no | `high` | Threshold for annotations and outputs |
| `fail-on-critical` | no | `true` | Fail the workflow when critical CVEs are found |
| `fail-on-malware` | no | `true` | Fail the workflow when a known-malicious package (OSV `MAL-` advisory) is found — independent of `min-severity` and `fail-on-critical` |
| `comment-on-pr` | no | `true` | Post (and update) a summary comment on pull requests |
| `include-dev-deps` | no | `false` | Include dev dependencies in the scan |
| `github-token` | no | `${{ github.token }}` | Token used for PR comments |

## Outputs

| Output | Description |
| --- | --- |
| `cve-count` | Total CVEs found (at or above `min-severity`) |
| `critical-count` | Number of critical CVEs |
| `high-count` | Number of high CVEs |
| `malware-count` | Number of known-malicious packages found |
| `report` | Full vulnerability report (JSON) |

## What you get

- Malicious packages called out first, in the job summary and PR comment,
  with what to do: don't merge, remove the package, rotate credentials.
- A job summary table with severity counts and a CISA KEV flag.
- Error annotations for malware; warning annotations for every critical/high CVE.
- One updating comment per PR (no comment spam across pushes).
- Every scan lands in your audit trail at cvespy.com with repo, branch and
  commit SHA — the timestamped record SOC 2 / ISO 27001 auditors ask for.

## Failing the build

By default the job fails when a **known-malicious package** is found — always,
regardless of `min-severity` or `fail-on-critical` — and when critical CVEs are
found. To keep a build green while you investigate a malware report, set
`fail-on-malware: 'false'` (not recommended). Tune CVE failures with:

```yaml
- uses: cvespy/action@v1
  with:
    api-key: ${{ secrets.CVESCAN_API_KEY }}
    min-severity: 'high'
    fail-on-critical: 'false'
```

## Self-hosted runners / private networks

The action calls `https://api.cvespy.com` directly. If your network or runner
blocks that host, set `api-url` to your API origin.

## License

MIT
