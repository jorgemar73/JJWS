# Jiu-Jitsu Weighing System (JJWS)

**JJWS — Jiu-Jitsu Weighing System** is a local-first, multiplatform application concept for managing competitor weigh-ins at Brazilian Jiu-Jitsu tournaments. The project aims to connect a digital weighing indicator to a computer, manage competitor and tournament data, apply configurable competition rules, and provide an operator interface with a public display.

> **Project status:** Early development / prototype. Hardware communication, protocol decoding, and application modules are being developed and validated incrementally. Features described as planned are not necessarily implemented yet.

## Objectives

- Make tournament weigh-ins more consistent, traceable, and efficient.
- Read weight data directly from a compatible weighing indicator through a serial interface.
- Manage competitors, registrations, divisions, belts, and tournament information.
- Apply configurable weigh-in rules and tolerances.
- Keep the primary database local and authoritative, with optional synchronization between computers over a local network.
- Support offline operation and controlled data exchange using USB.
- Provide a clear operator workflow and a separate public display.
- Build a maintainable application that can eventually run on Linux and Windows.

## Planned Features

- **Serial scale integration:** read data from the TEK Rover 5 indicator through RS-232.
- **Live weight display:** show weight, unit, and indicator status.
- **Stable-weight detection:** accept a measurement only when the indicator reports a stable reading.
- **Competitor management:** registration/document lookup, competitor name, photograph, division, belt, and relevant tournament data.
- **Weigh-in records:** timestamped, auditable records associated with a competitor and event.
- **Rules engine:** configurable rule sets, weight limits, tolerances, and rule versions.
- **Operator interface:** registration lookup, weighing workflow, status, and error handling.
- **Public display:** show approved competitor information and weigh-in status without exposing unnecessary personal data.
- **Local database:** persistent data storage using SQLite.
- **LAN synchronization:** one main computer acts as the authoritative database/API server; client computers can submit weigh-ins.
- **Offline support:** local queues and controlled synchronization when connectivity is restored.
- **USB data exchange:** versioned import/export packages with validation and checksums.
- **Audit trail:** record corrections or voided weigh-ins with a reason rather than silently changing historical records.
- **Backup and recovery:** documented backup, restore, and diagnostics procedures.

## Hardware

### Target weighing indicator

- **Manufacturer/model:** TEK Rover 5
- **Interface:** RS-232 serial
- **Connector:** DB9 (male on the indicator, according to the available documentation)
- **Scale capacity:** stated as 350 kg for the project setup; confirm the exact scale and indicator configuration before deployment.
- **Initial communication mode:** continuous transmission (`CONT`), subject to bench testing.

### Serial communication settings

The indicator manual describes the following serial settings:

| Setting | Value |
|---|---|
| Baud rate | 1200, 2400, 4800, or 9600 |
| Data bits | 8 |
| Parity | None |
| Stop bits | 1 |
| Format | 8N1 |
| Transmission modes | `DIS`, `CONT`, `rEq` |

The documented frame is 14 bytes:

| Byte | Description |
|---|---|
| 1 | Start character: `>` |
| 2 | Status: `S` (stable), `M` (motion), `O` (overload), `U` (underload) |
| 3 | Weight type: `G` (gross) or `N` (net) |
| 4–10 | Seven ASCII characters representing weight |
| 11–12 | Unit |
| 13 | Carriage return (`CR`) |
| 14 | Line feed (`LF`) |

The protocol description is based on the available indicator manual. **Actual frames, serial settings, request behavior, DB9 pinout, and adapter compatibility must be verified with the physical device.** No checksum is documented in the available frame description.

Use a genuine USB-to-RS-232 adapter. A USB-to-TTL serial adapter is not equivalent and must not be connected directly to an RS-232 interface.

## Technology Stack

The initial development stack is intended to be simple, open-source, and suitable for local operation:

| Component | Technology |
|---|---|
| Language | Python 3 |
| Serial communication | PySerial |
| Web application / API | Flask |
| Database | SQLite |
| Database access | SQLAlchemy |
| Front end | HTML, CSS, JavaScript, Bootstrap |
| Live public display | Server-Sent Events (SSE), if appropriate |
| Testing | pytest |
| Packaging | PyInstaller |
| Version control | Git |

The final stack and dependencies may evolve as the prototype is tested.

## Architecture Overview

JJWS is planned as a modular, local-first system.

```text
                 ┌─────────────────────────┐
                 │    RS-232 Indicator    │
                 │       TEK Rover 5       │
                 └────────────┬────────────┘
                              │ Serial
                 ┌────────────▼────────────┐
                 │ Serial Reader / Decoder │
                 │  Validate and parse    │
                 └────────────┬────────────┘
                              │
                 ┌────────────▼────────────┐
                 │   Weighing Data Service │
                 │ Stable reading logic   │
                 └────────────┬────────────┘
                              │
       ┌──────────────────────┼──────────────────────┐
       │                      │                      │
┌──────▼───────┐     ┌────────▼────────┐    ┌────────▼────────┐
│ Operator UI  │     │  Public Display │    │ Rules / Workflow│
└──────┬───────┘     └─────────────────┘    └────────┬────────┘
       │                                              │
       └──────────────────────┬───────────────────────┘
                              │
                 ┌────────────▼────────────┐
                 │   Application / API     │
                 └────────────┬────────────┘
                              │
                 ┌────────────▼────────────┐
                 │    SQLite Database      │
                 │ Local authoritative DB  │
                 └─────────────────────────┘
```

For multi-computer tournaments, the intended model is one main computer hosting the authoritative database and API. Client computers communicate with it over the local network. SQLite database files should not be shared directly over a network filesystem.

## Development Roadmap

The project is being developed in small modules, with hardware validation before building the full application.

1. **Serial Monitor** — connect to the indicator and inspect raw incoming data.
2. **Frame Decoder** — parse frames and validate status, weight, and units.
3. **Weighing Data Service** — expose readings and stable-weight logic to the application.
4. **Tournament and Competitor Management** — manage events, registrations, and competitor records.
5. **Rules Engine** — implement configurable competition rules and rule versions.
6. **Weigh-In Workflow** — connect competitor identification, measurement, acceptance, and recording.
7. **Operator UI and Public Display** — build the user interfaces.
8. **Database and Persistence** — define and implement the data model and migrations.
9. **LAN API and Synchronization** — add authenticated client/server communication.
10. **USB Import/Export** — create validated, versioned exchange packages.
11. **Security and Privacy** — authentication, permissions, data minimization, and privacy controls.
12. **Backup, Recovery, and Diagnostics** — backup/restore and troubleshooting tools.
13. **Packaging and Deployment** — build and test distributable packages for supported operating systems.

Each module should have documented requirements, tests, and acceptance criteria before it is considered complete.

## Development Environment

The initial development environment is Linux Mint. Windows support is a target for later testing and packaging.

### Prerequisites

- Python 3 (a supported version will be specified as the project matures)
- Git
- A USB-to-RS-232 adapter and the TEK Rover 5 indicator for hardware testing
- A serial terminal or the project’s serial-monitor module

### Clone the repository

```bash
git clone https://github.com/<YOUR-GITHUB-USERNAME>/<YOUR-REPOSITORY>.git
cd <YOUR-REPOSITORY>
```

### Create and activate a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

On Windows PowerShell:

```powershell
py -m venv .venv
.venv\Scripts\Activate.ps1
```

### Install dependencies

Once a `requirements.txt` file is available:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

> Installation and run commands will be updated as the repository structure and application entry point are established.

## Testing

Testing is expected to include:

- Unit tests for frame parsing and validation.
- Tests for stable/unstable, overload, and underload states.
- Tests for invalid, incomplete, and malformed serial frames.
- Tests for weight units and numeric parsing.
- Tests for weigh-in rule boundaries and tolerances.
- Tests for database persistence and audit history.
- Tests for duplicate synchronization requests and offline queues.
- Integration tests using simulated serial data.
- Hardware tests using the actual indicator and scale.
- User acceptance tests during a controlled tournament simulation.

Hardware behavior must be validated separately from simulated tests.

## Data, Privacy, and Security

JJWS may process personal information, including competitor names, identification details, photographs, and weigh-in records. The application should:

- Collect only information necessary for tournament operation.
- Restrict access to personal information and administrative functions.
- Avoid displaying identification numbers or other sensitive information on public screens.
- Use authentication and authorization for networked functions.
- Protect data in transit when network deployment requires it.
- Maintain a clear audit trail for corrections and voided records.
- Document retention, deletion, backup, and recovery procedures.
- Be reviewed for applicable privacy and data-protection requirements before real-world deployment.

This repository is a software-development project and does not constitute legal or regulatory advice.

## Contributing

Contributions, suggestions, bug reports, and technical discussion are welcome.

Before submitting a change:

1. Open an issue describing the proposed change or problem.
2. Keep changes focused and modular.
3. Add or update tests where applicable.
4. Document hardware-specific assumptions and test conditions.
5. Do not include real competitor personal data, photographs, credentials, or private tournament records in issues, commits, or test fixtures.
6. Submit a pull request with a clear description of the change and its validation.

Please note that the project is currently in an early development stage. Interfaces, data structures, and module boundaries may change.

## Disclaimer

JJWS is an independent software project. It is not affiliated with, endorsed by, or officially supported by the TEK indicator manufacturer, AJP, IBJJF, or any other competition organization. Competition rules must be verified against the current official rules of the relevant organization and configured accordingly.

## License

This project is licensed under the **MIT License**. See the [`LICENSE`](LICENSE) file for details.

Copyright (c) 2026 JJWS contributors

---

**JJWS — JiuJitsu Weighing System**  
*Designed to make tournament weigh-ins more reliable, traceable, and efficient.*
