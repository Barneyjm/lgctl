# Contributing to lgctl

This document provides instructions for developing and testing the lgctl package.

## Development Setup

### 1. Create a Virtual Environment

```bash
cd lgctl
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS/Linux
source .venv/bin/activate
```

### 2. Install Dependencies

```bash
# Install in development mode with dev dependencies
pip install -e ".[dev]"

# To also work on MCP server features, install the mcp extra
pip install -e ".[dev,mcp]"

# Or install all optional dependencies
pip install -e ".[all,dev]"
```

## Running Tests

### Run All Tests

```bash
pytest tests/ -v
```

### Run Specific Test Files

```bash
# Test formatters
pytest tests/test_formatters.py -v

# Test CLI
pytest tests/test_cli.py -v

# Test REPL
pytest tests/test_repl.py -v

# Test MCP server (requires mcp package)
pytest tests/test_mcp_server.py -v
```

### Run Tests with Coverage

```bash
pip install pytest-cov
pytest tests/ --cov=lgctl --cov-report=term-missing
```

## Test Structure

```
tests/
├── __init__.py
├── conftest.py           # Shared fixtures and mock clients
├── test_formatters.py    # TableFormatter, JsonFormatter, RawFormatter
├── test_client.py        # LGMemClient initialization
├── test_commands_store.py      # Store commands (ls, get, put, rm, etc.)
├── test_commands_threads.py    # Thread commands
├── test_commands_runs.py       # Run commands
├── test_commands_assistants.py # Assistant commands
├── test_commands_crons.py      # Cron commands
├── test_commands_ops.py        # Memory operations (grep, find, export, etc.)
├── test_cli.py           # CLI argument parsing and dispatch
├── test_repl.py          # Interactive REPL
└── test_mcp_server.py    # MCP server tools (requires mcp package)
```

## Optional Dependencies

The package has optional dependencies that enable additional features:

| Extra | Package                    | Features                            |
| ----- | -------------------------- | ----------------------------------- |
| `mcp` | `mcp>=1.0.0`               | MCP server for AI agent integration |
| `dev` | `pytest`, `pytest-asyncio` | Development and testing             |
| `all` | All optional packages      | Everything                          |

### MCP Server Tests

The MCP server tests require the `mcp` package. If not installed, these tests are automatically skipped:

```bash
# install via extras
pip install -e ".[mcp]"
```

## Mock Clients

Tests use mock clients defined in `conftest.py` that simulate LangGraph SDK behavior:

- `MockStoreClient` - Simulates store operations
- `MockThreadsClient` - Simulates thread operations
- `MockRunsClient` - Simulates run operations
- `MockAssistantsClient` - Simulates assistant operations
- `MockCronsClient` - Simulates cron operations
- `MockLGMemClient` - Complete mock client with all sub-clients

These mocks allow testing without a live LangGraph server.

## Writing Tests

### Async Tests

Most lgctl operations are async. Use `@pytest.mark.asyncio` for async tests:

```python
import pytest

class TestMyFeature:
    @pytest.mark.asyncio
    async def test_async_operation(self, mock_client):
        result = await mock_client.store.get_item(("user", "123"), "key")
        assert result is not None
```

Note: The `conftest.py` includes a hook that automatically adds the `@pytest.mark.asyncio` marker to async test functions, so explicit marking is optional.

### Using Fixtures

Common fixtures are available from `conftest.py`:

```python
class TestExample:
    def test_with_mock_client(self, mock_client):
        """mock_client provides a MockLGMemClient instance."""
        assert mock_client.url == "http://localhost:8123"

    def test_with_formatter(self, table_formatter, json_formatter):
        """Formatter fixtures are available."""
        pass

    def test_with_sample_data(self, sample_items, sample_threads):
        """Sample data fixtures for testing."""
        assert len(sample_items) > 0
```

## Code Style

- Follow PEP 8 guidelines
- Use type hints where appropriate
- Write docstrings for public functions and classes
- Keep test names descriptive: `test_<function>_<scenario>_<expected_result>`

## Pull Request Process

1. Create a feature branch from `main`
2. Write tests for new functionality
3. Ensure all tests pass: `pytest tests/ -v`
4. Update documentation if needed
5. Submit a pull request

## Troubleshooting

### "async def functions are not natively supported"

Ensure `pytest-asyncio` is installed and configured:

```bash
pip install pytest-asyncio
```

The `pyproject.toml` includes the necessary pytest-asyncio configuration.

### MCP tests skipped

Install the mcp package:

```bash
pip install mcp
```

### Import errors

Ensure the package is installed in development mode:

```bash
pip install -e ".[dev]"
```
