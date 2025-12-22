# Pin Protocol Proto Definitions

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository hosts the canonical `.proto` specifications for Pin Protocol, organized by layer to maintain clear architectural boundaries.

## Installation

```bash
# Clone the repository
git clone https://github.com/PIN-AI/pin-protocol-proto.git

# Or use as a submodule
git submodule add https://github.com/PIN-AI/pin-protocol-proto.git proto
```

## Directory Structure

```
proto/
├── common/             # Shared type definitions
│   └── types.proto            # Common types used across layers
│
├── rootlayer/          # RootLayer protocol definitions
│   ├── intent.proto           # Intent lifecycle (4-state model)
│   ├── assignment.proto       # Matcher-Agent assignments
│   ├── direct.proto           # Direct mode protocol
│   ├── error_reason.proto     # Error reason codes
│   ├── validation.proto       # Validation definitions
│   ├── service.proto          # Intent + assignment services
│   └── openapi.yaml           # OpenAPI specification
│
├── subnet/             # Subnet protocol definitions
│   ├── agent.proto            # Agent definitions
│   ├── bid.proto              # Agent bid payloads + acks
│   ├── matcher.proto          # Matcher snapshots + events
│   ├── matcher_service.proto  # Matcher gRPC control-plane
│   ├── execution_report.proto # Agent→Validator reports
│   ├── validation.proto       # Validation policies
│   ├── checkpoint.proto       # Checkpoint consensus
│   ├── validator.proto        # Validator management
│   ├── gossip.proto           # Validator gossip protocol
│   ├── report.proto           # Legacy evidence payloads
│   └── service.proto          # Subnet node service surface
│
└── service.proto       # Index file
```

## Architecture Principles

### Layer Separation
- **RootLayer** (`rootlayer/`): Intent management, bidding, and assignments
- **Subnet** (`subnet/`): Execution verification, validation, and checkpoints

### Key Design Decisions
1. **Matcher-local Bidding**: Each intent's bidding window is managed inside the subnet matcher, RootLayer only observes assignments.
2. **Simplified Status Models**: 4-state lifecycle for IntentStatus and AssignmentStatus
3. **Direct Reporting**: Agents submit ExecutionReports directly to Validators
4. **ECDSA Signatures**: Using ECDSA(secp256k1) with bitmap instead of BLS

## Guidelines

### Versioning
- RootLayer: `package rootlayer.v1`
- Subnet: `package subnet.v1`
- Use semantic versioning with proper deprecation

### Compatibility
- Use `bytes` for hashes/fingerprints
- Use `string` for IDs
- Mark deprecated fields, use `reserved` for removed fields

### Services
- `SubmitExecutionReport`: Agent→Validators (primary)
- `ReportIntent`: Legacy, marked deprecated

### Checkpoint
- ECDSA(secp256k1) single-sig + threshold
- Submit: `header + packed_signatures + signers_bitmap`
- Optional commitments: `assignments_root`, `validation_commitment`, `policy_root`

## Generating Go Bindings

### For Subnet
```bash
# From Subnet/ directory
make proto

# Or manually
protoc -I ../pin_protocol \
  --go_out=paths=source_relative:. \
  --go-grpc_out=paths=source_relative:. \
  ../pin_protocol/proto/subnet/*.proto
```

### For RootLayer
```bash
protoc -I ../pin_protocol \
  --go_out=paths=source_relative:. \
  --go-grpc_out=paths=source_relative:. \
  ../pin_protocol/proto/rootlayer/*.proto
```

## Migration Notes
- Subnet may temporarily keep local .proto files for compatibility
- Goal: Converge on this directory as single source of truth
- Generate code downstream from these canonical definitions
