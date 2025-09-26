# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is a **Model Context Protocol (MCP) server** for the COTI blockchain, enabling AI applications to interact with COTI's privacy-focused blockchain through Multi-Party Computation (MPC) technology. The server exposes 37 tools across 5 categories: account management, private ERC20 tokens, private ERC721 NFTs, native token operations, and transaction management.

## Essential Commands

### Development
```bash
# Start development server with live reloading
npm run dev

# Install dependencies
npm install

# Compile TypeScript to JavaScript
npx tsx index.ts

# Build for distribution (manual compilation)
mkdir -p build && cp index.ts build/index.js
```

### Testing MCP Server
```bash
# Test server locally using Smithery CLI
npx @smithery/cli dev

# Install server globally via Smithery
npx -y @smithery/cli install @davibauer/coti-mcp --client claude
```

### Environment Setup
Required environment variables for operation:
```bash
export COTI_MCP_AES_KEY="your_aes_key"
export COTI_MCP_PRIVATE_KEY="your_private_key" 
export COTI_MCP_PUBLIC_KEY="your_public_key"
export COTI_MCP_CURRENT_PUBLIC_KEY="current_active_key"
export COTI_MCP_NETWORK="testnet" # or "mainnet"
```

## Architecture Overview

### Core Structure
- **`index.ts`**: Main MCP server entry point that registers all 37 tools
- **`tools/`**: Organized by blockchain operation categories
  - `account/`: Account creation, key management, signing (12 tools)
  - `erc20/`: Private ERC20 token operations (8 tools) 
  - `erc721/`: Private ERC721 NFT operations (11 tools)
  - `native/`: Native COTI token operations (2 tools)
  - `transaction/`: Smart contract interaction and monitoring (4 tools)
- **`tools/shared/`**: Common utilities for account management and network handling
- **`tools/constants/`**: Contract ABIs and blockchain constants

### Tool Pattern
Each tool follows a consistent structure:
1. **Tool Definition**: Exported `ToolAnnotations` object with schema
2. **Validation**: Type guard function (`isXxxArgs`)  
3. **Implementation**: Core business logic function (`performXxx`)
4. **Handler**: MCP-compatible wrapper function (`xxxHandler`)

### Key Dependencies
- **`@modelcontextprotocol/sdk`**: MCP protocol implementation
- **`@coti-io/coti-ethers`**: COTI-specific ethers.js wrapper
- **`@coti-io/coti-sdk-typescript`**: COTI SDK for blockchain interactions
- **`zod`**: Runtime type validation for tool inputs

### Multi-Account Support
The server supports managing multiple accounts through comma-separated environment variables. Account switching is handled through the `COTI_MCP_CURRENT_PUBLIC_KEY` variable.

### Privacy & Encryption
All token operations leverage COTI's MPC technology for privacy. Values are encrypted using account-specific AES keys before blockchain submission.

## Development Guidelines

### Adding New Tools
1. Create tool file in appropriate category folder under `tools/`
2. Follow the established pattern: annotations, validation, implementation, handler
3. Import and register the tool in `index.ts`
4. Use shared utilities from `tools/shared/account.ts` for common operations

### Working with Private Operations
- Always use `getCurrentAccountKeys()` to get current account credentials
- Encrypt sensitive values using the account's AES key before blockchain submission
- Handle both encrypted and plaintext responses appropriately

### Network Handling
- Default network is testnet; use `getNetwork()` helper for current network
- Provider instances should use `getDefaultProvider(getNetwork())`
- Network switching affects all subsequent operations globally

### Error Handling
- Wrap blockchain operations in try-catch blocks
- Provide descriptive error messages that preserve original error context
- Validate all user inputs before processing

### Testing Operations
Most tools can be tested directly through:
- Claude Desktop with proper MCP configuration
- Smithery CLI development environment
- Direct tool invocation with proper environment setup

## Key Files to Understand

- **`tools/shared/account.ts`**: Account management utilities and network configuration
- **`tools/constants/abis.ts`**: Smart contract ABIs for ERC20/ERC721 operations
- **`index.ts`**: Server initialization and tool registration
- **`smithery.yaml`**: Smithery CLI configuration for development
- **`tsconfig.json`**: TypeScript compilation settings for Node.js ES modules

## Blockchain Networks

- **Testnet**: Default development network
- **Mainnet**: Production network
- **Devnet**: Development network (available but not commonly used)

Network selection affects contract addresses, gas costs, and available features.