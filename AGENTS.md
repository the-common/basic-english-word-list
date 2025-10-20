# AI agent instruction file

This file contains instructions and guidelines for AI agents interacting with this repository. It outlines the expected behavior, coding standards, and collaboration protocols to ensure effective and efficient contributions.

## Bash coding style

The following coding style guidelines should be followed when writing Bash scripts:

### Shebang

Use `#!/usr/bin/env bash` as the shebang line for portability across different environments.

### Parameter naming

* Use lowercase letters and underscores for variable names (e.g., `my_variable`) to enhance readability.
* Only use uppercase letters for variables whose values are sourced from the environment.

### Parameter expansion

Use `${var}` instead of `$var` for variable references to improve readability and avoid ambiguity.

This also applies to positional parameters, e.g., use `${1}` instead of `$1`.

### Message reporting

* Use `printf` instead of `echo` for formatted output.
* Prepend log level tags in the following format(except for help text):
    + `Info:`
    + `Warning:`
    + `Error:`
    + `FATAL:`
    + `DEBUG:`

### Linting

Use ShellCheck for linting.

### Function arguments

Use `local var="$1"; shift` to assign function arguments to local variables.  This allows cleaner diffs when adding or removing arguments.

### Defensive interpreter behavior

The following shell options should be set at the beginning of each script to ensure robust error handling:

```bash
set -o errexit   # Exit on most errors (see the manual)
set -o nounset   # Disallow expansion of unset variables
```

Do not set `pipefail`.

If the script contains functions, also include:

```bash
set -o errtrace  # Ensure the error trap is inherited
```

### Conditional expressions

* Use the `test` shell built-in for conditional expressions.  For example, use `if test -f "file"` instead of `if [[ -f "file" ]]`.

  The only exception is when using regex matching, which requires `[[ ... ]]`.  When doing so always define a regex_pattern variable instead of embedding the regex directly in the conditional expression.
* Do not use AND/OR lists syntax.

### Script template

The following script should be used for script creation, rewrites, and style references:

```bash
#!/usr/bin/env bash
# _script_description_
#
# Copyright _copyright_effective_year_ _copyright_holder_name_ <_copyright_holder_contact_>
# SPDX-License-Identifier: CC-BY-SA-4.0

init(){
    printf \
        'Info: Operation completed without errors.\n'
}

printf \
    'Info: Configuring the defensive interpreter behaviors...\n'
set_opts=(
    # Terminate script execution when an unhandled error occurs
    -o errexit
    -o errtrace

    # Terminate script execution when an unset parameter variable is
    # referenced
    -o nounset
)
if ! set "${set_opts[@]}"; then
    printf \
        'Error: Unable to configure the defensive interpreter behaviors.\n' \
        1>&2
    exit 1
fi

printf \
    'Info: Checking the existence of the required commands...\n'
required_commands=(
    realpath
)
flag_required_command_check_failed=false
for command in "${required_commands[@]}"; do
    if ! command -v "${command}" >/dev/null; then
        flag_required_command_check_failed=true
        printf \
            'Error: This program requires the "%s" command to be available in your command search PATHs.\n' \
            "${command}" \
            1>&2
    fi
done
if test "${flag_required_command_check_failed}" == true; then
    printf \
        'Error: Required command check failed, please check your installation.\n' \
        1>&2
    exit 1
fi

printf \
    'Info: Configuring the convenience variables...\n'
if test -v BASH_SOURCE; then
    # Convenience variables may not need to be referenced
    # shellcheck disable=SC2034
    {
        printf \
            'Info: Determining the absolute path of the program...\n'
        if ! script="$(
            realpath \
                --strip \
                "${BASH_SOURCE[0]}"
            )"; then
            printf \
                'Error: Unable to determine the absolute path of the program.\n' \
                1>&2
            exit 1
        fi
        script_dir="${script%/*}"
        script_filename="${script##*/}"
        script_name="${script_filename%%.*}"
    }
fi
# Convenience variables may not need to be referenced
# shellcheck disable=SC2034
{
    script_basecommand="${0}"
    script_args=("${@}")
}

printf \
    'Info: Setting the ERR trap...\n'
trap_err(){
    printf \
        'Error: The program has encountered an unhandled error and is prematurely aborted.\n' \
        1>&2
}
if ! trap trap_err ERR; then
    printf \
        'Error: Unable to set the ERR trap.\n' \
        1>&2
    exit 1
fi

init
```
