# A Staker's Cheat Sheet: Systemd Logs

This is a quick reference for any stakers that need to dig through their EL/CL client logs and are running a systemd Linux distribution like Ubuntu. If you followed one of Somer's guides or remember setting up a few `.service` files for your EL/CL then this content applies to you!

_NOTE:_ This reference does not apply to containerized setups like eth-docker. For these configurations the container engine manages the logs instead of systemd.

### Contents

* [View Log for Service](#view_log_for_service)
* [View Logs for Multiple Services](#view_logs_for_multiple_services)
* [View Live](#view_live)
* [View Most Recent N Lines](#view_most_recent_n_lines)
* [View Time Range](#view_time_range)
* [View Lines Containing Text](#view_lines_containing_text)
* [Write Log to File](#write_log_to_file)
* [Hide Extra Line Prefixes](#hide_extra_line_prefixes)
* [Mix and Match](#mix_and_match)
* [Special Thanks](#special_thanks)

<br/>

## View Log for Service<a id="view_log_for_service"></a>

This command tells `journalctl` to only return log messages corresponding to a specific systemd unit, where a service is a kind of unit. It's the most basic command, and all of the following sections build upon it.

_NOTE:_ The `journalctl` program is a part of systemd and is responsible for letting users retrieve entries from the systemd journal (aka. the thing that stores all of the service logs).

```
journalctl -u <service name>
```

The service name corresponds to the name of a `.service` file. For example, if we created a `geth.service` file by following a staking guide, the service name would be `geth`. Also, `journalctl` is lenient and will accept the full `geth.service` name with the extension present if we feel like typing more.

Examples:
```
journalctl -u geth
```
```
journalctl -u lighthousebeacon.service
```

### The Pager

All of these various log commands except for _View Live_ and _Write Log to File_ use a pager to present log messages in a scrollable and searchable terminal interface. Below is a basic list of hot keys to get us started, and additionally many basic vim hot keys are supported.
* `q` - quit
* `arrow keys` - scroll
* `page up/down` - scroll more
* `home` - go to first line of content
* `end` - go to last line of content
* `/text<enter>` - search for "text"
* `/<enter>` - go to next search hit
* `?<enter>` - go to previous search hit
* `h` - help

<br/>

## View Logs for Multiple Services<a id="view_logs_for_multiple_services"></a>

The `-u` flag can be specified multiple times to display log messages from multiple services ordered chronologically. This can be helpful when debugging issues involving multiple clients.

```
journalctl -u <service name 1> -u <service name 2> ...
```

Example:
```
journalctl -u nethermind -u lighthousebeacon -u lighthousevalidator
```

<br/>

## View Live<a id="view_live"></a>

This command will print log messages for a service live as they come in.

```
journalctl -u <service name> -f
```

Unlike the other commands in this guide, this one does not use the pager. It will keep live printing until something tells it to quit, like pressing `ctrl-c` in the terminal. Quitting the command only stops the printing of log messages. The underlying service will continue running.

Example:
```
journalctl -u nimbus -f
```

### Color Output

To get color coded output we can pipe the log output to the `ccze` utility, which is available as an Ubuntu package.

_NOTE:_ Because we are using a pipe here it will cause `journalctl` to not use the pager, and all of the log messages will be written directly to the terminal. For this reason we are listing this command as part of the _View Live_ section, where it is most useful.

Example:
```
journalctl -u nimbus -f | ccze
```

If there are issues with unicode characters, like ellipses, try the `-A` flag for raw ANSI mode.

Example:
```
journalctl -u lighthousevalidator -f | ccze -A
```

<br/>

## View Most Recent N Lines<a id="view_most_recent_n_lines"></a>

This command will display the most recent N log lines for a service, where N is a configurable number.

```
journalctl -u <service name> -n <number of lines>
```

Example:
```
journalctl -u erigon -n 1000
```

<br/>

## View Time Range<a id="view_time_range"></a>

This command will display log messages filtered down to a specific time range.

```
journalctl -u <service name> -S <"since" date/time> -U <"until" date/time>
```

### Absolute Values

A full datetime can be specified using the format "YYYY-MM-DD HH:MM:SS". Dates without the time component can also be used with the format "YYYY-MM-DD".

Example:
```
journalctl -u prysmvalidator -S "2022-09-15 06:43:00" -U "2022-09-16"
```

### Relative Values

Relative values can be specified with a `+` or `-` prefix to represent times in the future or past respectively. Additionally, the suffixes `d`, `h`, `min`, `s` represent days, hours, minutes, and seconds respectively.

Example:
```
journalctl -u teku -S -1d12h -U -45min30s
```

### Special Values

The special symbolic values `now`, `today`, and `yesterday` are accepted.

Example:
```
journalctl -u nethermind -S today -U -1h
```

<br/>

## View Lines Containing Text<a id="view_lines_containing_text"></a>

This command will display log messages that match a given sub-string or regular expression. It acts like grep.

```
journalctl -u <service name> -g <string or regex>
```

Example:
```
journalctl -u besu -g ERROR
```

<br/>

## Write Log to File<a id="write_log_to_file"></a>

This command uses a shell redirect to write log messages to a file. Writing longer segments of log output to a file is useful for things like uploading to Discord or including in a GitHub issue.

```
journalctl -u <service name> > <destination file path>
```

Example:
```
journalctl -u lodestar > /home/ubuntu/my_bug_report_log.txt
```

<br/>

## Hide Extra Line Prefixes<a id="hide_extra_line_prefixes"></a>

By default `journalctl` prints its own prefix at the beginning of each log message with information like the service name and a timestamp. For a simple home staking setup, however, this information is usually redundant and makes each line longer. By using the `cat` output mode we can turn this prefix off.

```
journalctl -u <service name> -o cat
```

Example:
```
journalctl -u akula -o cat
```

<br/>

## Mix and Match<a id="mix_and_match"></a>

Many of the flags in these commands can be mixed and matched. For one last example let's try getting all of the warnings from both Lighthouse services since yesterday and writing them to a file without the `journalctl` prefix.

Example:
```
journalctl -u lighthousebeacon -u lighthousevalidator -S yesterday -g WARN -o cat > /home/ubuntu/recent_lighthouse_warnings.txt
```

## Special Thanks<a id="special_thanks"></a>

* 1337 (Discord) - _View Logs for Multiple Services_, _Color Output_, _Write Log to File_, _Hide Extra Line Prefixes_ sections