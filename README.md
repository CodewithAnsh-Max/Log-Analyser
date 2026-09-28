# Log-Analyser

A Python command-line tool that analyzes system and application log files, finds errors and unusual activity, and summarizes the important events in a clear report.

Works on **Windows, Linux and macOS**, uses only the Python standard library, and needs no installation beyond Python itself.

## Features

- **Multi-format parsing**: automatically detects application logs, Linux syslog and Apache/Nginx access logs
- **Filtering and searching**: by severity level, keyword, regular expression, IP address and time range
- **Summary report**: severity breakdown, most frequent errors, top IP addresses, hourly activity timeline and critical events
- **Anomaly detection**:
  - Possible brute-force attacks (5+ failed logins from one IP within 5 minutes)
  - Error-rate spikes (an hour with far more errors than average)
  - HTTP 5xx bursts and 404 scanning from a single IP
  - Rare, one-off error messages
  - Root/admin activity at unusual hours
- **Windows support**: reads the live Windows Event Log (System, Application, Security) and handles UTF-16 files created by PowerShell
- **Export**: save the report as JSON
- **Colored output** in Windows Terminal, PowerShell and the VS Code terminal

## Requirements

- Python 3.9 or newer
- No external packages

## Quick Start

```bash
git clone https://github.com/YOUR-USERNAME/log-analyzer.git
cd log-analyzer
python log.py --demo
```

`--demo` generates a sample log with a planted attack and error spike, then analyzes it.

## Usage

```bash
# Analyze a file
python log.py mylog.txt

# Analyze several files (wildcards supported)
python log.py "C:\Logs\*.log"

# Show only errors and above, as raw lines
python log.py mylog.txt --min-level ERROR --show

# Search for a keyword or regex
python log.py mylog.txt -s timeout --show
python log.py mylog.txt -r "user=\w+" --show

# Filter by IP or time range
python log.py mylog.txt --ip 203.0.113.45
python log.py mylog.txt --since "2026-09-28 10:00" --until "2026-09-28 12:00"

# Save the report as JSON
python log.py mylog.txt --json report.json

# Read the Windows Event Log (Security needs an Administrator terminal)
python log.py --winlog System Application

# No arguments: interactive prompt
python log.py
```

### Options

| Option | Description |
|---|---|
| `files` | One or more log files (wildcards allowed) |
| `--level LEVEL` | Only these levels (repeatable) |
| `--min-level LEVEL` | This level and above (DEBUG, INFO, WARNING, ERROR, CRITICAL) |
| `-s`, `--search TEXT` | Case-insensitive keyword |
| `-r`, `--regex PATTERN` | Regular expression |
| `--ip ADDRESS` | Only entries involving this IP |
| `--since`, `--until` | Time range, format `YYYY-MM-DD HH:MM` |
| `--show` | Print matching lines instead of the summary |
| `--limit N` | Max lines printed with `--show` (default 50) |
| `--top N` | Rows in the top-N tables (default 8) |
| `--json FILE` | Also write the report as JSON |
| `--winlog LOG...` | Read Windows Event Logs |
| `--max-events N` | Events per Windows log (default 2000) |
| `--demo` | Generate and analyze sample data |

## Example Output

```
================================================================
LOG ANALYSIS REPORT
================================================================
Entries analysed : 429
Time range       : 2026-09-28 03:14:00 -> 2026-09-28 11:33:37

-- Severity breakdown --
  INFO        369 (86.0%) ##############################
  WARNING      44 (10.3%) ####
  ERROR        15 ( 3.5%) #
  CRITICAL      1 ( 0.2%) #

-- Most frequent errors --
     15x  db: Connection timeout to <ip>
      1x  app: Out of memory, worker killed

-- Anomalies / unusual activity --
  [HIGH  ] Possible brute-force: 203.0.113.45: 12 failed logins (burst starting 2026-09-28 09:30)
  [HIGH  ] Error spike: 2026-09-28 11:00: 16 errors (avg 3.2/hour)
  [MEDIUM] Off-hours privileged access: 03:14 line 1: auth: Accepted publickey for root
```

## Supported Log Formats

| Format | Example |
|---|---|
| Application log | `2026-09-28 10:15:01 ERROR db: Connection timeout` |
| Syslog | `Sep 28 10:15:01 host sshd[12]: Failed password for root from 1.2.3.4` |
| Web access log | `1.2.3.4 - - [28/Sep/2026:10:15:01 +0000] "GET /x" 500 123` |
| Windows Event Log | Read live with `--winlog` |

Lines in other formats are still read, and their severity is guessed from keywords such as "error", "failed" or "warning".

## How It Works

1. **Read**: opens each file with the correct encoding (UTF-8, UTF-8 BOM or UTF-16)
2. **Parse**: matches every line against known formats and extracts timestamp, level, source, message and IP
3. **Filter**: applies your level, keyword, regex, IP and time options
4. **Analyze**: counts levels and IPs, groups similar errors together (numbers and IPs are normalized) and builds the hourly timeline
5. **Detect**: runs rule-based anomaly checks
6. **Report**: prints a color-coded summary and optionally saves JSON

## Project Structure

```
log-analyzer/
├── log.py       # the complete tool
└── README.md
```

## Possible Improvements

- HTML report with charts
- Web interface for uploading log files
- Support for JSON-structured logs and IIS logs
- Live monitoring (`tail -f` style)
- Email or webhook alerts for high-severity anomalies

## License

MIT License. Free to use and modify.
