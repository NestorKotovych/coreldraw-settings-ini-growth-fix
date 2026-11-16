# CorelDRAW 2022: settings.ini Growing Indefinitely — Investigation and Fix

## Summary

CorelDRAW was repeatedly increasing the size of its `settings.ini` configuration file, eventually reaching approximately **104 MB**.

The file grew by roughly **327,680 bytes (320 KiB)** during each application startup or shutdown cycle, even when no documents were opened.

Investigation revealed that the `MRUList` entry under `[AssetListOptions/Groups/FontList.TextProperties]` contained repeated font metadata and malformed serialized data.

**A full workspace reset using F8 resolved the issue in subsequent tests.**

## Environment

- Application: CorelDRAW Graphics Suite 2022
- Version: 24.5.0.731
- Operating system: Windows
- Installed fonts: approximately 1,133
- Configuration location:

  `%APPDATA%\Corel\CorelDRAW Graphics Suite 2022\Draw\Workspace\settings.ini`

## Symptoms

The investigation began with warnings relating to file locations and configuration settings, including a message stating that `settings.ini` was read-only.

The file was not marked read-only in Windows.

Inspection revealed that it had grown to approximately 104 MB.

## Root cause investigation

The unusually large part of the file was the following section:

```ini
[AssetListOptions/Groups/FontList.TextProperties]
MRUList=...
```

The value contained serialized metadata about fonts, encoded as four-digit hexadecimal Unicode values.

After decoding, the data resembled a JSON array of font descriptions.

Initially, the array was valid. On subsequent launches, additional data was appended after the closing bracket, making the entire value invalid.

Repeated cycles caused the file to grow continuously.

The same corrupted data was also found inside the `content/settings.ini` entry embedded in the `_default.cdws` workspace archive.

This demonstrated that the corrupted state was stored in both locations, although it did not establish which component originally introduced the corruption.

## Troubleshooting attempts

| Attempt | Result |
|---|---|
| Rename `settings.ini` and let CorelDRAW recreate it | Temporary improvement |
| Manually clear the font `MRUList` | Issue returned |
| Switch from Default to Illustration workspace | Issue persisted |
| Reduce recently used fonts setting from 5 to 1 | No improvement |
| Reset workspace using F8 | Issue no longer reproduced |

## Successful workaround: Reset the workspace

**Warning:** This procedure can reset customized toolbars, shortcuts, menus and other workspace preferences.

1. Close CorelDRAW.
2. Back up the entire `Workspace` directory.
3. Hold **F8** while launching CorelDRAW.
4. Confirm the prompt to reset the workspace to factory defaults.
5. Let CorelDRAW start normally.
6. Close and reopen the application to verify the result.
7. Monitor the size of `settings.ini`.

Do not restore a previously corrupted `.cdws` workspace file during testing.

## Verification

After the F8 reset, the following results were observed:

| Test | settings.ini size |
|---|---:|
| First launch and exit after reset | 11,734 bytes |
| Next launch and exit | 11,800 bytes |
| Create a temporary drawing, use fonts and graphic objects, then exit | Approximately 15.8 KB |

The previously observed 320 KiB growth per cycle did not recur during these tests.

The final `MRUList` was valid and contained only the fonts used during the test.

## Conclusion

The issue appears to involve corrupted or improperly serialized font-list settings associated with CorelDRAW's workspace configuration.

A full F8 workspace reset resolved the problem in this particular installation, while recreating `settings.ini` alone did not.

**The underlying software defect has not been conclusively identified.** The workaround should be treated as a confirmed local result, not a guaranteed universal solution.

## Tested

November 16, 2026.
