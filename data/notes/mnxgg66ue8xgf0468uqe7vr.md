


Sets an alarm for the underlying process.

This ensures that, without malicious interception, the process
should automatically die after the specified number of seconds.

This is automatically transferred to all child processes and across
session and process group boundaries, so there is no need to do
anything about child processes.

## Usage

    $ ./alarm 5 /bin/sh -c "echo hello; sleep 6; echo nope"
    hello
    <exited>

>The alarm() function shall cause the system to generate a SIGALRM signal for the process after the number of realtime seconds specified by seconds have elapsed.


> APPLICATION USAGE
The fork() function clears pending alarms in the child process. A new process image created by one of the exec functions inherits the time left to an alarm signal in the old process' image.

> The exec family of functions shall replace the current process image with a new process image. The new image shall be constructed from a regular, executable file called the new process image file. There shall be no return from a successful exec, because the calling process image is overlaid by the new process image

```zig
const std = @import("std");
const stdout = std.io.getStdOut().writer();

pub fn main() !void{
    const args = std.process.argsAlloc(std.testing.allocator) catch unreachable;
    defer std.process.argsFree(std.testing.allocator, args);
    
    if (args.len < 3){
        std.debug.print("usage: {s} <seconds> <command> [args...]", .{args[0]});
        std.process.exit(1);
    }

    // TODO: make alarm() in zig with std.os.raise() or std.os.kill()      
    _ = std.c.alarm(std.fmt.parseInt(u32, args[1], 0) catch 0); 

    // std.time.sleep(10e9); // 10 seconds

    return std.process.execv(std.testing.allocator, args[2..]);

}
```

## build

```bash
zig build-exe alarm.zig -O ReleaseSmall --strip -fsingle-threaded -lc
```