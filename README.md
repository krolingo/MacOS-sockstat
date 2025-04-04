# `sockstat` Script for macOS

This script mimics FreeBSD's `sockstat(1)` on macOS by using `lsof` to list active TCP and UDP listening sockets in a clean, readable format.

## 📄 Script

```sh
#!/bin/sh
# Usage: sockstat [-l4 | -l6]
#   -l4  Show only IPv4 connections
#   -l6  Show only IPv6 connections

if [ "$1" = "-l4" ]; then
    filter="IPv4"
elif [ "$1" = "-l6" ]; then
    filter="IPv6"
else
    filter=""
fi

# Print the header
printf "%-10s %-10s %-6s %-4s %-6s %s\n" "USER" "COMMAND" "PID" "FD" "PROTO" "LOCAL ADDRESS"

# Run lsof and filter output with awk
doas lsof -nP -iTCP -sTCP:LISTEN -iUDP | awk -v f="$filter" '{
    if ($1 == "COMMAND") next;
    if (f != "" && $5 != f) next;
    printf "%-10s %-10s %-6s %-4s %-6s %s\n", $3, $1, $2, $4, $8, $9;
}'
```

## ✅ How It Works

- `doas lsof -nP -iTCP -sTCP:LISTEN -iUDP`: Lists all TCP sockets in the LISTEN state and all UDP sockets.
  - `-nP`: Prevents name/port resolution for faster output.
  - `-iTCP`, `-iUDP`: Filters to TCP and UDP sockets only.
  - `-sTCP:LISTEN`: Limits TCP results to sockets in the LISTEN state.

- The script supports:
  - `-l4`: Show only IPv4 sockets.
  - `-l6`: Show only IPv6 sockets.

- Output columns:
  - `USER`: Owner of the process.
  - `COMMAND`: Name of the command.
  - `PID`: Process ID.
  - `FD`: File descriptor.
  - `PROTO`: Protocol type (TCP/UDP).
  - `LOCAL ADDRESS`: Address and port.

## 🛠️ Prerequisites

- macOS with `lsof` installed (default).
- `doas` configured — or replace `doas` with `sudo` if needed.

## 📥 Usage

Make it executable and place it in your `~/bin` or `/usr/local/bin`:

```sh
chmod +x sockstat
mv sockstat ~/bin/
```

Run it like:

```sh
sockstat
sockstat -l4
sockstat -l6
```


## 🧪 Attempting to Compile `sockstat` on macOS

We also attempted to compile the original FreeBSD `sockstat` utility on macOS.

### ✅ Result:
- The binary **did compile successfully** with some modifications to account for macOS headers and build differences.
- However, **the output was empty or non-functional**, due to:
  - Lack of access to the kernel socket structures used in FreeBSD.
  - macOS’s different system APIs and internal socket tracking mechanisms.

### 📌 Conclusion:
Because macOS lacks FreeBSD's `kern.ipc` and `sysctl` structures used by `sockstat`, we opted to recreate its output using `lsof`, which works consistently across macOS systems.

This shell-based `sockstat` script gives a familiar experience without needing to port C code from BSD.
