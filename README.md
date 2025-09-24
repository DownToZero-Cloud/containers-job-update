# containers-job-update

GitHub composite action to update an existing DTZ Containers job. The action first fetches the current job, overlays any provided inputs, and posts the update. Only `api_key` and `job_id` are required; all other fields are optional.

- API base URL is fixed to `https://containers.dtz.rocks`
- API version is fixed to `2021-02-21`

## Usage

```yaml
- uses: DownToZero-Cloud/containers-job-update@main
  with:
    api_key: ${{ secrets.DTZ_API_KEY }}
    job_id: job-abc123
    name: nightly-backup
    schedule_type: precise
    schedule_cron: '0 2 * * *'
    env_variables: '{"LOG_LEVEL":"debug","SECRET":{"plainValue":"s3cr3t"}}'
```

### Second example: Update the image repository (and/or tag/digest)

```yaml
- uses: DownToZero-Cloud/containers-job-update@main
  with:
    api_key: ${{ secrets.DTZ_API_KEY }}
    job_id: job-abc123
    container_image: myregistry/webapp@sha256:abc3534
```

## Inputs

- `api_key` (required): DTZ API key (`X-API-KEY`).
- `job_id` (required): JobId to update.
- `name` (optional): Job name. Omit to keep as-is.
- `container_image` (optional): Container image repo, e.g. `nginx` or `myrepo/app`.
- `container_pull_user` (optional): Registry username. Set to empty string `""` to clear.
- `container_pull_pwd` (optional): Registry password. Set to empty string `""` to clear.
- `schedule_type` (optional): One of `relaxed`, `precise`, or `none`.
- `schedule_cron` (optional): Cron expression. Set to empty string `""` to clear.
- `schedule_repeat` (optional): ecoMode repetition bounds in the format `min(<freq>) max(<freq>)`. Allowed frequencies: `hourly`, `daily`, `weekly`, `monthly`. Set to empty string `""` to clear.
- `env_variables` (optional): JSON object of env vars. Example: `{"KEY":"value","SECRET":{"plainValue":"x"}}`.
  - Supports plain strings, `EncryptedValue` and `PlainValue` shapes as per the API spec.

Notes:
- If an optional input is omitted, the current value from the fetched job is kept.
- Empty string inputs for `container_pull_user`, `container_pull_pwd`, `schedule_cron`, and `schedule_repeat` explicitly clear those fields.

## Outputs

- `job`: Compact JSON string of the updated job object returned by the API.

Example to capture the updated job:

```yaml
- id: update
  uses: DownToZero-Cloud/containers-job-update@main
  with:
    api_key: ${{ secrets.DTZ_API_KEY }}
    job_id: job-abc123
- name: Show updated job
  run: |
    echo '${{ steps.update.outputs.job }}' | jq .
  shell: bash
```

## More Examples

- Clear schedule and registry credentials:

```yaml
- uses: DownToZero-Cloud/containers-job-update@main
  with:
    api_key: ${{ secrets.DTZ_API_KEY }}
    job_id: job-abc123
    schedule_cron: ""
    schedule_repeat: ""
    container_pull_user: ""
    container_pull_pwd: ""
```

- Set ecoMode bounds using `schedule_repeat`:

```yaml
- uses: DownToZero-Cloud/containers-job-update@main
  with:
    api_key: ${{ secrets.DTZ_API_KEY }}
    job_id: job-abc123
    schedule_type: relaxed
    schedule_repeat: 'min(daily) max(weekly)'
```

- Update only environment variables:

```yaml
- uses: DownToZero-Cloud/containers-job-update@main
  with:
    api_key: ${{ secrets.DTZ_API_KEY }}
    job_id: job-abc123
    env_variables: '{
      "LOG_LEVEL":"info",
      "DATABASE_URL":{"plainValue":"postgresql://user:pwd@host:5432/db"}
    }'
```

## How it works

1. Fetches current job: `GET /api/2021-02-21/job/{job_id}`
2. Builds an overlay from provided inputs.
3. Merges overlay over fetched job to ensure existing values remain present when not overridden.
4. Updates job: `POST /api/2021-02-21/job/{job_id}`

## Requirements

- Runs on Bash with `curl` and `jq` available (present on GitHub-hosted Ubuntu runners).

## Security

- Always pass `api_key` via GitHub Encrypted Secrets, e.g. `${{ secrets.DTZ_API_KEY }}`.