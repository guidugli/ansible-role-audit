
# Review notes

## Main issues addressed

- Removed redundant task-level validate_argument_spec from tasks/main.yml.
- Added explicit defaults aligned with argument_specs auto-validation.
- Switched optional string public variables to empty-string semantics where appropriate.
- Added allow-list validation for requested rule templates.
- Added missing audit_max_restarts mapping and render path into auditd.conf.
- Made Molecule converge control privilege escalation instead of role tasks.
- Replaced broken generator scripts with syntactically valid, lint-safe versions.
- Added document starts and quoted 'on' in GitHub Actions workflows.
- Made update_release_metadata.sh run py_compile preflight checks before execution.
