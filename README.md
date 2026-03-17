# PNPM falsely reports "Command not found" on Windows

If `pnpm --filter not-current-folder-package exec` exited with an error, PNPM falsely claims the command was not found

## Reproduction

1. Clone this repository

   ```sh
   git clone https://github.com/maxpatiiuk/pnpm-windows-false-not-found
   cd pnpm-windows-false-not-found
   ```

2. Install dependencies: `pnpm install`

3. Reproduce the error:

   Windows (wrong error message):

   ```log
   > pnpm --filter sub-package exec prettier --check test.js
   Checking formatting...
   [warn] test.js
   [warn] Code style issues found in the above file. Run Prettier with --write to fix.
   C:\Users\mak13180\site\pnpm-windows-false-not-found\sub-package:
    ERR_PNPM_RECURSIVE_EXEC_FIRST_FAIL  Command "prettier" not found
   ```

   macOS/linux (correct error message):

   ```log
   > pnpm --filter sub-package exec prettier --check test.js
   Checking formatting...
   [warn] test.js
   [warn] Code style issues found in the above file. Run Prettier with --write to fix.
   /Users/mak13180/s/e/pnpm-windows-false-not-found/sub-package:
    ERR_PNPM_RECURSIVE_EXEC_FIRST_FAIL  Command failed with exit code 1: prettier --check test.js
   ```

## Investigation

I think the bug occurs here: https://github.com/pnpm/pnpm/blob/4a36b9a1105ea4a22e32484d20245cf9cfd281ce/exec/commands/src/exec.ts#L397-L413

On macOS/linux it checks the command output to determine if command failed due to command being not found.

On Windows it uses `which`. Is it possible `which` is invoked from a wrong folder when pnpm was called with `--filter`?

Alternatively, consider looking for the `is not recognized as an internal or external command` substring instead. This is the expected output if the command is really not found:

```log
> pnpm --filter sub-package exec prettier --check test.js
'prettier' is not recognized as an internal or external command,
operable program or batch file.
C:\Users\mak13180\site\pnpm-windows-false-not-found\sub-package:
 ERR_PNPM_RECURSIVE_EXEC_FIRST_FAIL  Command "prettier" not found
```
