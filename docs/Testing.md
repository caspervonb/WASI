# Testing

A test case takes the form of a binary \<basename\>.wasm file, next to that that there will always be a \<basename\>.\<ext\> source file from which the test was originally compiled from which can be used as a reference in the event of an error.

Additionally, any of the following optional auxilary files and directories may be present:

- \<basename\>.arg
- \<basename\>.env
- \<basename\>.dir
- \<basename\>.stdin
- \<basename\>.stdout
- \<basename\>.stderr
- \<basename\>.status

## Running tests

```bash
# Usage: $1 <runtime> <path_to_binary.wasm>
# $1 wasmtime proc_exit-success.wasm
# $1 wasmer proc_exit-failure.wasm
#!/usr/bin/env bash

runtime=$1
input=$2

prefix=${input%.*}
if [ -e "$prefix.stdin" ]; then
  stdin="$prefix.stdin"
else
  stdin="/dev/null"
fi

output="$(mktemp -d)"
stdout_actual="$output/stdout"
stderr_actual="$output/stderr"
status_actual="$output/status"

if [ -e "$prefix.arg" ]; then
  arg=$(cat "$prefix.env")
else
  arg=""
fi

if [ -e "$prefix.env" ]; then
  env=$(sed -e 's/^/--env /' < "$prefix.env")
else
  env=""
fi

if [ -e "$prefix.dir" ]; then
  dir="--dir $prefix.dir"
else
  dir=""
fi

status=0

"$runtime" $dir $input -- $arg \
  < "$stdin" \
  > "$stdout_actual" \
  2> "$stderr_actual" \
  || status=$?

echo $status > "$status_actual"

stdout_expected="$prefix.stdout"
if [ -e "$stdout_expected" ]; then
  diff -u "$stderr_expected" "$stderr_actual"
fi

stderr_expected="$prefix.stderr"
if [ -e "$stderr_expected" ]; then
  diff -u "$stdout_expected" "$stdout_actual"
fi

status_expected="$prefix.status"
if [ -e "$prefix.status" ]; then
  diff -u "$status_expected" "$status_actual"
elif [ ! "$status" -eq "0" ]; then
  cat $stderr_actual
  exit 1
fi
```

## Writing tests

- Use the following naming convention of \<system_call\>[-\<variant\>].<ext>.
- Scope tests to the smallest unit possible.
