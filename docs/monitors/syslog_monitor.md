/// DECLARE path=/help/monitors/syslog-monitor
/// DECLARE title=Syslog Monitor
/// DECLARE section=help
/// DECLARE subsection=monitors

# Syslog Monitor

The Syslog monitor allows the Scalyr Agent to act as a syslog server, proxying logs from any application or device
that supports syslog. It can recieve log messages via the syslog TCP or syslog UDP protocols.

@class=bg-warning docInfoPanel: An *agent monitor plugin* is a component of the Scalyr Agent. To use a plugin,
simply add it to the ``monitors`` section of the Scalyr Agent configuration file (``/etc/scalyr/agent.json``).
For more information, see [Agent Plugins](/help/scalyr-agent#plugins).


## Sample Configuration

This sample will configure the agent to accept syslog messages on TCP port 601 and UDP port 514, from localhost
only:

    monitors: [
      {
        module:                    "scalyr_agent.builtin_monitors.syslog_monitor",
        protocols:                 "tcp:601, udp:514",
        accept_remote_connections: false
      }
    ]

You can specify any number of protocol/port combinations. Note that on Linux, to use port numbers 1024 or lower,
the agent must be running as root.

You may wish to accept syslog connections from other devices on the network, such as a firewall or router which
exports logs via syslog. Set ``accept_remote_connections`` to true to allow this.

Additional options are documented in the Configuration Reference section, below.


## Log files and parsers

By default, all syslog messages are written to a single log file, named ``agentSyslog.log``. You can use the
``message_log`` option to specify a different file name (see Configuration Reference).

If you'd like to send messages from different devices to different log files, you can include multiple syslog_monitor
stanzas in your configuration file. Specify a different ``message_log`` for each monitor, and have each listen on a
different port number. Then configure each device to send to the appropriate port.

syslog_monitor logs use a parser named ``agentSyslog``. To set up parsing for your syslog messages, go to the
[Parser Setup Page](/parsers) and click one of the action buttons for the agentSyslog parser, such as
{{menuRef:Ask us to create for you}}. If you are using multiple syslog_monitor stanzas, you can specify a different
parser for each one, using the ``parser`` option.


## Sending messages via syslog

To send messages to the Scalyr Agent using the syslog protocol, you must configure your application or network
device. The documentation for your application or device should include instructions. We'll be happy to help out;
please drop us a line at [support@scalyr.com](mailto:support@scalyr.com).


### Rsyslogd

To send messages from another Linux host, you may wish to use the popular ``rsyslogd`` utility. rsyslogd has a
powerful configuration language, and can be used to forward all logs or only a selected set of logs.

Here is a simple example. Suppose you have configured Scalyr's Syslog Monitor to listen on TCP port 601, and you
wish to use rsyslogd on the local host to upload system log messages of type ``authpriv``. You would add the following
lines to your rsyslogd configuration, which is typically in ``/etc/rsyslogd.conf``:

    # Send all authpriv messasges to Scalyr.
    authpriv.*                                              @@localhost:601

Make sure that this line comes before any other filters that could match the authpriv messages. The ``@@`` prefix
specifies TCP.


## Viewing Data

Messages uploaded by the Syslog Monitor will appear as an independent log file on the host where the agent is
running. You can find this log file in the [Overview](/logStart) page. By default, the file is named "agentSyslog.log".


## Configuration Reference

|||# Option                       ||| Usage
|||# ``module``                   ||| Always ``scalyr_agent.builtin_monitors.syslog_monitor``
|||# ``protocols``                ||| Optional (defaults to ``tcp:601``). Lists the protocols and ports on which the \
                                      agent will accept messages. You can include one or more entries, separated by \
                                      commas. Each entry must be of the form ``tcp:NNN`` or ``udp:NNN``. Port \
                                      numbers are optional, defaulting to 601 for TCP and 514 for UDP.
|||# ``accept_remote_connections``||| Optional (defaults to false). If true, the plugin will accept network \
                                      connections from any host; otherwise, it will only accept connections from localhost.
|||# ``message_log``              ||| Optional (defaults to ``agent_syslog.log``). Specifies the file name under which \
                                      syslog messages are stored. The file will be placed in the default Scalyr log \
                                      directory, unless it is an absolute path.
|||# ``parser``                   ||| Optional (defaults to ``agentSyslog``). Defines the parser name associated with \
                                      the log file.
|||# ``max_log_size``             ||| Optional (defaults to 50 MB). How large the log file will grow before it is rotated. \
                                      Set to zero for infinite size. Note that rotation is not visible in Scalyr; it is \
                                      only relevant for managing disk space on the host running the agent. However, a \
                                      very small limit could cause logs to be dropped if there is a temporary network \
                                      outage and the log overflows before it can be sent to Scalyr.
|||# ``max_log_rotations``        ||| Optional (defaults to 2). The maximum number of log rotations before older log \
                                      files are deleted. Set to zero for infinite rotations.
|||# ``log_flush_delay``          ||| Optional (defaults to 1.0). The time to wait in seconds between flushing the log \
                                      file containing the syslog messages.
