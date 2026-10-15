# SchemaSmith Deploy

State-based schema deployment for **SQL Server, PostgreSQL, MySQL and MariaDB**, as a GitHub Action.
Point it at a [SchemaSmith](https://github.com/Schema-Smith/SchemaSmith) schema package and it runs
SchemaQuench against your target: preview the changes on a pull request, apply them on merge, and get the
exit code, the log directory and the summary file back as outputs.

No install step on the runner. The action downloads the matching self-contained SchemaQuench binary for the
runner's OS and architecture at run time, so it works on `ubuntu-latest`, `windows-latest` and `macos-latest`
alike.

## Quick start

```yaml
- name: WhatIf on PR
  if: github.event_name == 'pull_request'
  uses: Schema-Smith/schemasmith-deploy@v2.7.3
  with:
    mode: whatif
    product-path: ./schema
    server: ${{ secrets.DB_SERVER }}
    user: ${{ secrets.DB_USER }}
    password: ${{ secrets.DB_PASSWORD }}

- name: Deploy on merge
  if: github.ref == 'refs/heads/main'
  uses: Schema-Smith/schemasmith-deploy@v2.7.3
  with:
    mode: deploy
    product-path: ./schema
    server: ${{ secrets.DB_SERVER }}
    user: ${{ secrets.DB_USER }}
    password: ${{ secrets.DB_PASSWORD }}
```

**Pin a release tag.** The tag pins both the action and the SchemaQuench version it runs, so a deploy that
worked yesterday runs the same engine today. Leave `version` unset and it follows the tag; set it to run a
different SchemaSmith release on purpose.

## Inputs

| Input | Default | Description |
|---|---|---|
| `mode` | `deploy` | `deploy`, `whatif`, `validate`, `test-connection` or `preview-targets`. |
| `product-path` | `.` | Path to the schema package (its `SchemaPackagePath`). |
| `server` | | Target server. Required for every mode except `validate`. |
| `user` | | Target user. Leave blank for Windows authentication on SQL Server. |
| `password` | | Target password. Passed to SchemaQuench through the environment, never on the command line. |
| `version` | the action's tag | SchemaSmith release to run, for example `v2.7.2` or `latest`. |
| `extra-args` | | Raw `--Key:value` overrides appended to the SchemaQuench command line: port, template, database and schema filters, `Drop*` toggles, connection properties, `MaxThreads`, and anything else SchemaQuench accepts. |

## Outputs

| Output | Description |
|---|---|
| `exit-code` | SchemaQuench exit code: `0` success, `2` failed or partial. |
| `log-dir` | Directory containing the run logs. |
| `summary-path` | Path to `SchemaQuench - Summary.md` (empty if the run produced none). Post it as a PR comment to show reviewers exactly what a merge would change. |

## Example: post the WhatIf summary on the pull request

```yaml
- name: WhatIf
  id: whatif
  uses: Schema-Smith/schemasmith-deploy@v2.7.3
  with:
    mode: whatif
    product-path: ./schema
    server: ${{ secrets.DB_SERVER }}
    user: ${{ secrets.DB_USER }}
    password: ${{ secrets.DB_PASSWORD }}

- name: Comment the summary
  if: steps.whatif.outputs.summary-path != ''
  env:
    GH_TOKEN: ${{ github.token }}
  run: gh pr comment "${{ github.event.pull_request.number }}" --body-file "${{ steps.whatif.outputs.summary-path }}"
```

## Documentation

- [CI/CD integration guide](https://github.com/Schema-Smith/SchemaSmith/blob/main/docs/end-user/guide/07-cicd-integration.md): pipeline patterns, pre-flight checks, operational profiles, secret handling.
- [SchemaQuench reference](https://github.com/Schema-Smith/SchemaSmith/blob/main/docs/end-user/reference/schemaquench.md): every switch `extra-args` can pass, and the engine version compatibility matrix.
- [SchemaSmith](https://github.com/Schema-Smith/SchemaSmith): the CLI tools, the schema package format, and the demos.

This repository is published from the SchemaSmith repository at each release; its tags match SchemaSmith's
release tags. Issues and pull requests for the action go to
[Schema-Smith/SchemaSmith](https://github.com/Schema-Smith/SchemaSmith/issues).

## License

SchemaSmith Community License (SSCL) 2.0. See [LICENSE](LICENSE).
