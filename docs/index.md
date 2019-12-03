# Gravwell SOAR Modules

This repository provides pre-built modules for use in [Gravwell scheduled scripts](https://dev.gravwell.io/docs/#!scripting/scriptingsearch.md).

To use a module within a script, you must first *include* the module, then it can be used. The example below demonstrates the use of a module (`alerts/email.ank`), to detect and report successful ssh logins.

```
time = import("time")

# SET THESE VARIABLES
var serverPath = "http://10.0.0.1"
var from = "gravwell.alerts@example.com"
var to = [ "admin@example.com" ]
var sub = "Gravwell SOAR"
var my_name = "Gravwell SOAR Agent"
var report_name = "Test"
var duration = 24 * time.Hour
var query = `tag=syslog words Accepted |
syslog Appname==sshd Hostname Message |
regex -e Message "Accepted (?P<method>\S+) for (?P<user>\S+) from (?P<ip>\S+)" |
stats count by Hostname method user ip |
table Hostname method user ip count`

require("alerts/email.ank")

alert = emailAlert
alert.EnableBatch()
alert.SetServerPath(serverPath)
alert.SetQueryParams(query, START.Add(-1*duration), START)
alert.SetEmailParams(from, to, sub)
alert.SetTitle(report_name)
alert.ThrottleOn("ip", time.Hour)
alert.ReingestAlerts("alerts", "testing", "SSH Login")
err = alert.Run()
println("Finished run", err)
return err
```

Gravwell will automatically fetch libraries specified with `include()` from github.com/gravwell/libs as needed, unless another source is specified in the Gravwell configuration. See [the Gravwell scripting documentation](https://dev.gravwell.io/docs/#!scripting/scriptingsearch.md) for more details on configuring libraries and external functions.