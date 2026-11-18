# IoT Server — ESP32 Telemetry and Device Control

A Node.js and TypeScript server that connects an ESP32 device to a web dashboard through MQTT and HTTP. The project demonstrates an end-to-end IoT data flow: sensor readings travel from hardware to a server and dashboard, while control commands travel back to the device.

The current example uses a DHT22 temperature and humidity sensor and an LED output. Keeping the first version deliberately small makes the device-to-server-to-browser and browser-to-device paths easier to understand and test.

## What it does

- Receives sensor telemetry from an ESP32 over MQTT.
- Parses incoming messages into typed readings.
- Keeps the latest reading available to the API.
- Stores historical readings in PostgreSQL through Prisma.
- Exposes telemetry and device endpoints over HTTP.
- Publishes device commands through MQTT.
- Includes a mock device workflow for development without physical hardware.

## Architecture

```text
ESP32 + DHT22
     |
     | MQTT telemetry
     v
 MQTT broker
     |
     v
Node.js / TypeScript server <---- HTTP ----> Web dashboard
     |
     +---- PostgreSQL (Prisma)

Web dashboard ---- HTTP command ----> Server
                                      |
                                      | MQTT command
                                      v
                                  MQTT broker
                                      |
                                      v
                                  ESP32 LED
```

The browser communicates with the server over HTTP; it does not connect directly to the MQTT broker. Broker credentials therefore remain on the server.

## Message flow

### Telemetry: device to dashboard

1. The ESP32 reads the DHT22 sensor.
2. It publishes a JSON payload to `spectral/esp32/{deviceId}/sensors/dht`.
3. The server receives the MQTT message and parses it into a typed `DHT22Reading`.
4. The latest reading is made available in memory and persisted to PostgreSQL.
5. The dashboard requests the latest reading through the HTTP API.

### Control: dashboard to device

1. The dashboard sends a request to `POST /api/devices/{deviceId}/led`.
2. The server validates the requested LED state.
3. The server publishes an `ON` or `OFF` command to the device's MQTT topic.
4. The ESP32 receives the command and updates the LED output.

## MQTT topic contract

| Topic | Direction | Payload |
|---|---|---|
| `spectral/esp32/{deviceId}/sensors/dht` | Device → server | JSON temperature and humidity reading |
| `spectral/esp32/{deviceId}/health` | Device → server | `alive` heartbeat |
| `spectral/esp32/{deviceId}/commands/led` | Server → device | `ON` or `OFF` |

The topic and payload form the message contract between the device and server. A local mock publisher can use the same contract when physical hardware is unavailable.

## Technology

- Node.js
- TypeScript
- MQTT
- Fastify
- PostgreSQL
- Prisma
- ESP32 firmware
- DHT22 sensor
- Next.js dashboard integration

## Repository guide

| Directory | Responsibility |
|---|---|
| `src/config/` | Environment configuration and validation |
| `src/mqtt/` | Broker connection, subscriptions, and publishing |
| `src/utils/` | Shared utilities such as logging |
| `src/types/` | Shared type definitions |
| `src/devices/` | DHT22 payload parsing and LED command creation |
| `src/services/` | Application coordination and latest-reading state |
| `src/api/` | Fastify server and HTTP routes |
| `src/database/` | Prisma client and database operations |
| `src/agent/` | AI agent tools and data-grounded responses |
| `firmware/` | ESP32 firmware |
| `scripts/` | Development utilities, including the mock device |
| `ai/` | Exploratory AI and analytics work |

See the README files in individual directories for more detailed component documentation.

## Development progress

The repository is developed incrementally, with supporting guides in `docs/`.

| Stage | Guide | Outcome |
|---|---|---|
| 0 | [Prerequisites](docs/00-prerequisites.md) | Development tools and broker access |
| 1 | [Project setup](docs/01-project-setup.md) | TypeScript project setup |
| 2 | [Connect to the broker](docs/02-connect-to-broker.md) | Receive MQTT messages |
| 3 | [Types](docs/03-types.md) | Shared message types |
| 4 | [Parse the DHT22 payload](docs/04-parse-dht22.md) | Convert raw messages to typed readings |
| 5 | [In-memory store](docs/05-in-memory-store.md) | Keep the latest reading |
| 6 | [HTTP API](docs/06-http-api.md) | Expose telemetry over HTTP |
| 7 | [Publish commands](docs/07-publish-commands.md) | Send LED commands to the device |
| 8 | [PostgreSQL and Prisma](docs/08-postgres-prisma.md) | Persist telemetry history |
| 9 | Hardening | Validation, error handling, and structured logging |
| 10 | [AI agent](docs/10-ai-agent.md) | Ask questions about IoT data |

The repository's development guide identifies the end of Stage 8 as the current baseline. Check the latest commits and documentation for any subsequent progress.

## Getting started

### Prerequisites

- Node.js and npm
- Access to an MQTT broker
- PostgreSQL database
- ESP32 and DHT22 hardware for physical-device testing (optional when using the mock device)

### Install dependencies

```bash
npm install
```

Create a local environment file and add the broker and database configuration expected by the application:

```bash
cp .env.example .env
```

Keep broker credentials and database secrets private. Do not commit `.env`.

### Run the server

```bash
npm run dev
```

The API is configured for local development at `http://localhost:3000`.

### Run without hardware

In a second terminal, use the repository's mock-device script:

```bash
npm run mock
```

The mock publisher sends messages using the same MQTT topic and payload contract as the ESP32.

## Further documentation

- [Development guides](docs/)
- [Firmware documentation](firmware/README.md)
- [AI documentation](ai/README.md)
- [Improvement proposals](docs/improvements/README.md)

## Project status

This repository is an incremental IoT engineering project. The core example focuses on one sensor and one output device; additional devices, production hardening, and expanded AI capabilities are separate extensions.
