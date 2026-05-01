# Development Guide

This guide documents the current PySide6-based development conventions for the IDA patching plugin.

## Runtime Target

- IDA Pro 9.x
- Python 3 as provided by IDA
- PySide6 and shiboken6 as provided by IDA
- Bundled Keystone files under `keystone/`

The plugin is intended to run inside an IDA Qt process. Local Python checks can validate syntax, but they may not be able to import IDA modules or PySide6 unless they use IDA's Python environment.

## Project Layout

- `core.py`: patching state, action registration, and persistence.
- `actions.py`: IDA action handlers and context-menu entry points.
- `asm.py`: Keystone-backed assembly helpers.
- `ui/preview.py`: controller/model for the interactive patch preview.
- `ui/preview_ui.py`: PySide6 widgets for the patch preview dockable.
- `ui/save.py`: controller/model for applying patches to a target file.
- `ui/save_ui.py`: PySide6 widgets for the apply-patches dialog.
- `util/ida.py`: IDA SDK helpers and popup/menu integration.
- `util/qt.py`: the only place that imports Qt bindings directly.

## Qt Binding Rules

- Import Qt classes through `patching.util.qt`; do not import PySide6 directly from feature modules.
- Use `wrap_instance(...)` from `util/qt.py` when wrapping IDA-owned Qt pointers.
- Use scoped PySide6 enums, for example:
  - `QtCore.Qt.AlignmentFlag.AlignRight`
  - `QtCore.Qt.WindowType.WindowSystemMenuHint`
  - `QtCore.Qt.Key.Key_Down`
  - `QtGui.QFont.StyleHint.Monospace`
- Use `QDialog.exec()` instead of the old `exec_()` name.
- Clear stylesheets with an empty string, not `None`.

## Validation Checklist

Run a syntax-only check from the repository root without writing new `__pycache__` files:

```powershell
python -c "import pathlib; [compile(p.read_text(encoding='utf-8'), str(p), 'exec') for p in pathlib.Path('.').rglob('*.py')]; print('syntax ok')"
```

When running inside IDA, verify the Qt binding layer:

```python
from patching.util import qt
print(qt.QT_AVAILABLE, qt.QT_BINDING)
```

Expected result in IDA 9.x is `True PySide6`.

Manual smoke tests in IDA:

- Open a supported x86/x64/Arm/Arm64 database.
- Confirm patching actions appear in the disassembly context menu.
- Open the Assemble dialog and move through preview lines with the up/down keys.
- Edit an instruction and confirm the bytes/status field refreshes.
- Apply patches through the save dialog and confirm the dialog closes on success.
- Open the preview context menu and confirm irrelevant default IDA actions are filtered.

## Notes For Future Changes

Keep the Qt compatibility boundary small. If a future IDA release changes Qt ownership, pointer wrapping, or enum behavior, update `util/qt.py` first and only touch UI modules when their direct API usage changes.
