This repository contains logging packages for Go:
 - [stdlog](#stdlog) is the main package of this repository, it is a simple and
   fast logger to the standard output.
 - [buflog](#buflog) and [golog](#golog) are customizable logging class which
   can be used as a standalone or as a building block for other loggers.
   [stdlog](#stdlog) is built upon them.
 - [log](#log) just provides a common interface for logging libraries.

You are more than welcome to ask questions on the [Go mailing-list](https://groups.google.com/d/topic/golang-nuts/Gif79T-1jNQ/discussion) and open issues here if you find bugs.


# stdlog

Package stdlog provides simple and fast logging to the standard output
(stdout) and is optimized for programs launched via a shell or cron. It can
also be used to log to a file by redirecting the standard output to a file.
This package is thread-safe.

Basic examples:

    logger := stdlog.GetFromFlags()
    logger.Info("Connecting to the server...")
    logger.Errorf("Connection failed: %q", err)

Will output:

    2014-04-02 18:09:15.862 INFO Connecting to the API...
    2014-04-02 18:10:14.347 ERROR Connection failed (Server is unavailable).

Log*() functions can be used to avoid evaluating arguments when it is
expensive and unnecessary:

    logger.Debug("Memory usage: %s", getMemoryUsage())
    if LogDebug() { logger.Debug("Memory usage: %s", getMemoryUsage()) }

If debug logging is off the getMemoryUsage() will be executed on the first
line while it will not be executed on the second line.

List of command-line arguments:

    -log=info
        Log events at or above this level are logged.
    -stderr=false
        Logs are written to standard error (stderr) instead of standard
        output.
    -flushlog=none
        Until this level is reached nothing is output and logs are stored
        in the memory. Once a log event is at or above this level, it
        outputs all logs in memory as well as the future log events. This
        feature should not be used with long-running processes.

The available levels are the eight ones described in RFC 5424 (debug,
info, notice, warning, error, critical, alert, emergency) and none.

Some use cases:
   - By default, all logs except debug ones are output to the stdout. Which
     is useful to follow the execution of a program launched via a shell.
   - A program launched by a crontab where the variable `MAILTO` is set
     with `-debug -flushlog=error` will send all logs generated by the
     program only if an error happens. When there is no error the email
     will not be sent.
   - `my_program > /var/log/my_program/my_program-$(date+%Y-%m-%d-%H%M%S).log`
     will create a log file in /var/log/my_program each time it is run.


# buflog

Package buflog provides a buffered logging class that accumulates logs in
memory until the flush threshold is reached which release stored logs, the
buffered logger then act as a normal logger.

Basic example:

    logger := buflog.New(os.Stdout, log.Info, log.Error)
    logger.Info("Connecting to the server...")   // Outputs nothing
    logger.Error("Connection failed")            // Outputs both lines


# golog

Package golog provides a customizable logging class which can be used as a
standalone or as a building block for other loggers.

Basic example:

    logger := golog.New(os.Stdout, log.Info)
    logger.Info("Connecting to the server...")
    logger.Errorf("Connection failed: %q", err)

Will output:

    2014-04-02 18:09:15.862 INFO Connecting to the API...
    2014-04-02 18:10:14.347 ERROR Connection failed (Server is unavailable).

Log*() functions can be used to avoid evaluating arguments when it is
expensive and unnecessary:

    logger.Debug("Memory usage: %s", getMemoryUsage())
    if logger.LogDebug() { logger.Debug("Memory usage: %s", getMemoryUsage()) }

If debug logging is off getMemoryUsage() will be executed on the first line
while it will not be executed on the second line.


# log

Package log provides a common interface for logging libraries.
