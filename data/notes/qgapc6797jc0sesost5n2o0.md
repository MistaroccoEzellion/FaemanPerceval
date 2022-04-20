

Similar to the `alarm` module, but uses `SIGKILL` instead of `SIGALRM`, which is not blockable.

## Usage

```sh
$ ./timeout 1 /bin/sh -c "echo hello; sleep 2; echo nope"
hello
<exited>
```

> The kill() system call can be used to send any signal to any process group or process.

> setpgid() sets the PGID of the process specified by pid to pgid.
       If pid is zero, then the process ID of the calling process is
       used.  If pgid is zero, then the PGID of the process specified by
       pid is made the same as its process ID.


```zig

const std = @import("std");
const stdout = std.io.getStdOut().writer();

pub fn main() !void {
    const args = std.process.argsAlloc(std.testing.allocator) catch unreachable;
    defer std.process.argsFree(std.testing.allocator, args);

    if (args.len < 3) {
        std.debug.print("usage: {s} <seconds> <command> [args...]", .{args[0]});
        std.process.exit(1);
    }

    try kill_timeout(std.fmt.parseInt(u32, args[1], 0) catch 0);

    return std.process.execv(std.testing.allocator, args[2..]);
}

fn kill_timeout(seconds: u32) !void {
    const child = try std.os.fork();

    if (child == 0) {
        std.time.sleep(@as(u32, seconds * 1_000_000_000));
        try std.os.kill(child, std.os.SIG.KILL);
    }
    _ = std.os.linux.syscall2(std.os.SYS.setpgid, 0, 0);
}

```

## build

```bash
zig build-exe timeout.zig -O ReleaseSmall --strip -fsingle-threaded 
```