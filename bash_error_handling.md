### Error Handling in Linux Bash Scripting

Error handling in Bash scripts is crucial for making scripts robust, especially in production environments where failures (e.g., network issues, missing files, or command failures) can occur. Idiomatic approach to strict error handling, particularly useful for scripts involving external commands like `ssh`. I'll break it down step by step, explain what each part does, why it's useful, and how it fits together.

#### 1. `set -euo pipefail` (at the top of the script)
This is a "strict mode" declaration placed near the beginning of your script (right after `#!/bin/bash`). It enables several Bash options to catch errors early and prevent silent failures. You can set it with a single command:

```bash
#!/bin/bash
set -euo pipefail
```

- **Breakdown**:
  - **`set -e`**: Exit immediately if any command exits with a non-zero status (indicating failure). Without this, the script continues running even if a command fails, which can lead to cascading errors.
    - *Why?* It stops the script on the first failure, avoiding partial or incorrect execution.
    - *Example*: If `ls nonexistent_file` fails (exit code 2), the script halts.
  - **`set -u`**: Treats unset or empty variables as errors during expansion (e.g., `${var}` where `var` is unset).
    - *Why?* Prevents subtle bugs from undefined variables, which default to empty strings and can cause unexpected behavior.
    - *Example*: Running `echo $MISSING_VAR` without `-u` prints nothing; with `-u`, it exits with an error.
  - **`set -o pipefail`** (or just `pipefail`): In a pipeline (e.g., `cmd1 | cmd2`), the exit status is the last non-zero status from any command in the pipe, not just the final one.
    - *Why?* Pipelines can mask failures—if `cmd1` fails but `cmd2` succeeds, the pipeline exits 0 without `pipefail`.
    - *Example*: `false | true` exits 0 normally, but with `pipefail`, it exits 1 (from `false`).

- **Overall benefit**: This trio makes your script "fail-fast" and more predictable. It's a best practice for any non-interactive script. Note: You can disable it temporarily with `set +e` if needed for specific sections.

#### 2. `trap 'echo "Error on line $LINENO"' ERR` (trapping errors)
The `trap` command sets up a handler for specific signals or events. Here, it's catching the `ERR` pseudo-signal, which triggers whenever a command exits with a non-zero status (complementing `set -e`).

```bash
trap 'echo "Error on line $LINENO"' ERR
```

- **Breakdown**:
  - **`trap`** : Defines a function or command to run when a signal/event occurs.
  - **`'echo "Error on line $LINENO"'`**: The command to execute on error. It prints a message with the line number (`$LINENO`) where the error happened.
    - `$LINENO` is a special Bash variable that expands to the current line number in the script.
  - **`ERR`**: The event to trap—any command failure (non-zero exit).

- **Why use it?** 
  - With `set -e`, the script exits on error, but `trap ERR` lets you add custom cleanup or logging *before* exiting. This provides debugging info (e.g., "Error on line 42") without manual checks everywhere.
  - It's lightweight and script-wide—no need to wrap every command.

- **Enhanced example** (for more robust handling):
  ```bash
  trap 'echo "Error on line $LINENO: Command failed"; exit 1' ERR
  ```
  This exits with code 1 (convention for general errors) and could be extended for cleanup like `rm -f temp_file`.

- **Caveats**: `ERR` only works in Bash 3+ (common on modern Linux). It doesn't trap errors in subshells or functions unless you propagate it (e.g., `set -E`).

#### 3. Explicit SSH Exit Code Checking: `if ! ssh ...; then echo "SSH failed"; exit 1; fi`
Even with the above, some commands (like `ssh`) need explicit checks because they might not always trigger `set -e` in expected ways (e.g., if used in a subshell or with redirects). This pattern manually verifies the exit code.

```bash
if ! ssh user@host 'remote_command'; then
    echo "SSH failed"
    exit 1
fi
```

- **Breakdown**:
  - **`ssh user@host 'remote_command'`**: Runs a command on a remote host via SSH.
  - **`if ! ...`**: The `!` negates the exit status—`true` if `ssh` fails (non-zero exit), `false` if it succeeds.
    - SSH exit codes: 0 = success; 1+ = various failures (e.g., connection refused, auth failed).
  - **`then echo "SSH failed"; exit 1; fi`**: On failure, print a message and exit with 1 (standard error code).

- **Why explicit for SSH?**
  - SSH can fail silently in pipelines or backgrounds, and `set -e` might not catch remote-side errors perfectly.
  - It allows custom logic, like retrying or emailing alerts, instead of just exiting.
  - Best practice: Always check exit codes for critical external tools (e.g., `git`, `curl`, `scp`).

- **Improved version** (with more context):
  ```bash
  ssh_output=$(ssh user@host 'remote_command 2>&1')
  if [ $? -ne 0 ]; then
      echo "SSH failed: $ssh_output"
      exit 1
  fi
  echo "$ssh_output"  # Use the output if successful
  ```
  Here, `$?` captures the immediate exit code, and we capture stderr (`2>&1`) for better diagnostics.

#### How It All Fits Together: A Complete Example Script
Here's a minimal script incorporating all three for an SSH-based task (e.g., remote file check):

```bash
#!/bin/bash
set -euo pipefail  # Strict mode

trap 'echo "Error on line $LINENO: Script failed"' ERR  # Error logging

# Example: SSH to remote host and check a file
remote_host="user@example.com"
if ! ssh "$remote_host" '[ -f /path/to/file ]'; then
    echo "SSH check failed on $remote_host"
    exit 1
fi

echo "All good! File exists remotely."
```

- **Execution flow**:
  1. Strict mode catches most errors automatically.
  2. Trap logs the exact line if something slips through.
  3. SSH gets explicit handling for reliability.

#### Tips for Best Practices
- **Test thoroughly**: Run with `bash -x script.sh` to trace execution.
- **Exit codes**: Use 0 for success, 1-125 for errors (126=command not executable, 127=not found).
- **Alternatives**: For complex scripts, consider functions with local error handling or tools like `fail` wrappers.
- **Limitations**: These work great for linear scripts but may need tweaks for loops/conditionals (use `|| true` to ignore specific failures).

This approach makes your Bash scripts more professional and less prone to "heisenbugs." If you have a specific script snippet or scenario, I can refine this further!
