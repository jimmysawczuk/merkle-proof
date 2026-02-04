# Merkle Proof Generator

A web tool for generating Merkle trees and proofs from CSV data, designed for Solidity/Ethereum allowlists.

## Features

- Parse CSV data and assign Solidity types to columns
- Generate Merkle roots using `solidityPackedKeccak256`
- Export leaves and legend data as JSON
- Optional address case normalization

## Setup

```bash
bun install
```

## Development

```bash
bun run dev
```

## Build

```bash
bun run build
```

## Usage

1. Paste CSV data into the text area (e.g., `address,amount`)
2. Select the Solidity type for each column (`address`, `uint256`, `uint64`, or `IGNORE`)
3. Optionally set a legend key to index the output by a column
4. Copy the generated Merkle root, leaves, or legend JSON

## Tech Stack

- Svelte 5
- Vite 7
- ethers.js v6
- merkletreejs
- Tailwind CSS v4
