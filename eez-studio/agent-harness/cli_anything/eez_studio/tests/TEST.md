# EEZ Studio Harness Test Plan

## Test Inventory Plan

- `test_core.py`: 8 unit tests planned.
- `test_full_e2e.py`: 4 end-to-end tests planned.

## Unit Test Plan

### `core.project`

- Create a native LVGL `.eez-project` JSON document.
- Validate required EEZ Studio sections: `settings.general`, `settings.build`, `userPages`, `scpi`.
- Save/load round trip.
- Mutate general settings and build destination.
- Add pages and LVGL widgets.

### `core.scpi`

- Add SCPI subsystems, commands, and parameters.
- Preserve query response metadata for commands ending with `?`.

### `core.session`

- Track undo/redo snapshots across native project mutations.

### CLI JSON

- Use Click/subprocess execution to verify `--json project new` emits machine-readable output and creates a valid project.

## E2E Test Plan

### Native CLI Workflow

- Create a saved `.eez-project`.
- Add a label, button, SCPI subsystem, and SCPI query command through the installed CLI.
- Reload and validate JSON structure.

### Backend Status Probe

- Call `backend status --json` to verify the CLI reports backend availability as structured JSON.

### Real Backend Inspect

- Call `lvgl backend-inspect` through the CLI. This must invoke EEZ Studio's built `docker-build-lib.js` via Node. If `EEZ_STUDIO_SOURCE` is not configured or EEZ Studio is not built, the test must fail loudly with setup instructions.

### Simulator Output Verification

- Validate generated simulator output by checking `index.html`, `index.js`, and WebAssembly magic bytes for `index.wasm`.

## Realistic Workflow Scenarios

**Embedded panel scaffold**

- Simulates: embedded GUI developer scaffolding a panel project.
- Operations chained: create project, add LVGL widgets, save, validate.
- Verified: native JSON structure, page/widget counts, output file presence.

**SCPI instrument command model**

- Simulates: test engineer adding a measurable instrument command surface.
- Operations chained: add subsystem, add query, add parameter when applicable.
- Verified: subsystem and command arrays match EEZ Studio SCPI model.

**Backend LVGL project inspection**

- Simulates: build automation reading EEZ project settings before a manufacturing/test export.
- Operations chained: prepare destination directory, invoke real EEZ Studio Node backend.
- Verified: backend exit code and structured project info from `docker-build-lib.js`.

## Test Results

Commands run from `eez-studio/agent-harness`:

```bash
python3 -m json.tool ../../registry.json
python3 -m compileall cli_anything/eez_studio
python3 -m pip install -e .
python3 -m pytest cli_anything/eez_studio/tests/test_core.py -v
python3 -m pytest cli_anything/eez_studio/tests/test_full_e2e.py -v -s
```

Unit test result:

```text
cli_anything/eez_studio/tests/test_core.py::test_create_project_has_native_sections PASSED
cli_anything/eez_studio/tests/test_core.py::test_save_load_round_trip PASSED
cli_anything/eez_studio/tests/test_core.py::test_set_general_and_destination PASSED
cli_anything/eez_studio/tests/test_core.py::test_add_page_and_widgets PASSED
cli_anything/eez_studio/tests/test_core.py::test_scpi_subsystem_command_parameter PASSED
cli_anything/eez_studio/tests/test_core.py::test_session_undo_redo PASSED
cli_anything/eez_studio/tests/test_core.py::test_cli_json_project_new PASSED
cli_anything/eez_studio/tests/test_core.py::test_cli_json_mutation_autosaves PASSED

8 passed in 0.10s
```

Full E2E result:

```text
cli_anything/eez_studio/tests/test_full_e2e.py::TestCLISubprocessE2E::test_help PASSED
cli_anything/eez_studio/tests/test_full_e2e.py::TestCLISubprocessE2E::test_native_project_scpi_workflow PASSED
cli_anything/eez_studio/tests/test_full_e2e.py::TestCLISubprocessE2E::test_backend_status_json PASSED
cli_anything/eez_studio/tests/test_full_e2e.py::TestCLISubprocessE2E::test_real_backend_inspect_required FAILED

1 failed, 3 passed in 2.86s
```

Backend failure evidence:

```text
EEZ Studio backend is required for this E2E test.
Set EEZ_STUDIO_SOURCE to a built https://github.com/eez-open/studio checkout.

stderr:
{
  "error": "EEZ Studio backend is not available.\n\nInstall/build the real target software and point this harness at it:\n  git clone https://github.com/eez-open/studio.git\n  cd studio\n  npm install\n  npm run build\n  export EEZ_STUDIO_SOURCE=/absolute/path/to/studio\n\nFor full LVGL simulator builds, Docker must also be installed and running.\n",
  "type": "RuntimeError"
}
```

## Summary Statistics

- Unit tests: 8 passed, 0 failed.
- Full E2E tests: 3 passed, 1 failed because the real EEZ Studio backend is unavailable in this environment.
- Registry JSON validation: passed.
- Python compile validation: passed.
- Editable install: passed.

## Coverage Notes

The native `.eez-project` editing path, session undo/redo, SCPI model edits, CLI subprocess execution, and JSON output are covered. The real EEZ Studio backend path is implemented and intentionally remains a failing E2E gate until a built EEZ Studio checkout is provided through `EEZ_STUDIO_SOURCE`.
