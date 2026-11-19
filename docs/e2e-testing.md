# End-to-end testing

The [ecosystem demos](ecosystem-support.md#ecosystem-examples) exercise the SBOM Action
from dependency installation through exported SPDX data to Atlas repository relationships.
These are test repositories, not packages to publish or attach to Atlas projects.

## Test setup

Each demo commits dependency manifests and lockfiles, an `expected.json` file with
package/version and repository expectations, a control package, a smoke/build check,
and workflows for baseline submission and read-only verification. Toolchains and
Action revisions are pinned in the workflows. Use the recorded fixture revision
when reproducing a run.

Dependency graph must be enabled. Gradle also requires Automatic dependency submission;
see [the setup guide](ecosystem-support.md#enable-dependency-collection).
Allow GitHub's asynchronous dependency collection to finish before exporting a
changed fixture. A run can otherwise export an earlier dependency graph.

## Test stages

Check these stages separately:

1. **Build and resolve:** install or build the fixture with its pinned dependencies.
2. **SBOM export:** check expected package identities, versions and dependency paths. A successful
   export or a high package count alone does not establish coverage.
3. **Atlas processing:** wait for the submission to be processed, then check the repository's
   `depends-on` API response for the expected repository IDs.

The demo repositories must stay unattached to Atlas projects. Their verification checks enforce this.

For a downloaded SBOM, run the demo's checker from its repository directory:

```sh
python3 check.py sbom --sbom /path/to/export.spdx.json
```

The demos locate submission records by matching Atlas's semantic hash of the document name,
packages and relationships. This supports deduplicated submissions without treating export timestamps
or namespaces as dependency changes. Submission-record lookup has a separate 90-second budget.

## Wait for processing

The demo checker polls every 15 seconds for up to five minutes. Once processing completes,
it checks dependency relationships. Missing relationships, failed processing and API errors fail
verification.

If the submission remains pending at the deadline, the checker records `pending-timeout` and
`verification_complete: false`. The workflow does not fail solely for that delay, but it is
**not a verified coverage pass**.

Run the demo's **Verify processed Atlas submission** workflow later with the saved submission ID,
at the same fixture revision. The verifier only reads Atlas data; it does not resubmit the SBOM
or request reprocessing. The dependency endpoint reports the current repository graph, rather than
an immutable graph snapshot tied to the selected submission.

You can also run the read-only check locally:

```sh
python3 check.py atlas --submission-id SUBMISSION_ID
```

Use `--processing-wait-seconds` and `--poll-interval` to change the polling budget.
Check `status` and `verification_complete` in the saved result rather than relying only on the
workflow's green status.

## Keep verification evidence

The workflows save the exported SBOM, expectations, submission information and check results.
Keep the fixture commit and workflow-run references with the results so the check can be reproduced.
Baseline artifacts are retained for 30 days; download any evidence you need before it expires.

GitHub's reverse **Dependents** page is not a substitute for checking the exported SBOM and Atlas
relationships. Dependencies submitted through snapshots may appear in an SBOM without appearing
on that page.
