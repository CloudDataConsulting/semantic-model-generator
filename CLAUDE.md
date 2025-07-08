# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Setup and Installation
- **Poetry setup**: `make setup` or `poetry install`
- **Streamlit app setup**: `make setup_admin_app` or `pip install .`
- **Looker integration**: `pip install -e ".[looker]"`

### Running the Application
- **Local Streamlit**: `make run_admin_app` or `python -m streamlit run app.py`
- **Requirements**: Python 3.9-3.11 (excluding 3.9.7)

### Testing
- **Run all tests**: `make test` or `python -m pytest -vvs semantic_model_generator`
- **Test framework**: pytest with verbose output

### Code Quality
- **Format code**: `make fmt_lint` (runs black, isort, and flake8)
- **Black formatting**: `make run_black`
- **Import sorting**: `make run_isort`
- **Type checking**: `make run_mypy`
- **Linting**: `make run_flake8`

### Build and Release
- **Build wheel**: `make build`
- **Version bump**: `poetry version [patch|minor|major]`
- **Release**: `make release` (creates git tag and pushes)

## Architecture Overview

### Core Components

1. **Streamlit Application** (`app.py`)
   - Main entry point for the semantic model generator UI
   - Supports both local deployment and Streamlit in Snowflake (SiS)

2. **Semantic Model Generation** (`semantic_model_generator/`)
   - `generate_model.py`: Core logic for generating Cortex Analyst semantic models
   - `validate_model.py`: Validation of generated semantic models
   - Protocol buffers for data structures (`protos/`)

3. **Partner Integrations** (`partner/`)
   - DBT semantic model translation
   - Looker explore translation
   - Cortex integration utilities

4. **Snowflake Integration**
   - Connection handling via `snowflake_utils/`
   - Supports multiple authentication methods (password, MFA, SSO)
   - Callable stored procedure: `CORTEX_ANALYST_SEMANTICS.SEMANTIC_MODEL_GENERATOR.GENERATE_SEMANTIC_FILE`

### Key Design Patterns

1. **Multi-table Support**: Generates semantic models for multiple tables with join relationships
2. **Auto-description Generation**: Uses Cortex LLM for generating column descriptions when missing
3. **Partner Metadata Translation**: Extracts and merges metadata from DBT and Looker models
4. **Token Limit Management**: Enforces <30,980 token limit for semantic models

## Snowflake Connection Configuration

The app supports two connection methods:

1. **connections.toml**: Preferred method using Snowflake's standard connection file
2. **Environment Variables**: 
   - Required: `SNOWFLAKE_ROLE`, `SNOWFLAKE_WAREHOUSE`, `SNOWFLAKE_USER`, `SNOWFLAKE_ACCOUNT_LOCATOR`, `SNOWFLAKE_HOST`
   - Authentication options: password, MFA, SSO via `SNOWFLAKE_AUTHENTICATOR`

## Important Constraints

- **Token Limit**: Generated semantic models must be <30,980 tokens (~123,920 characters)
- **Python Version**: 3.9+ required (excluding 3.9.7), up to 3.11
- **BCR Requirement**: Streamlit in Snowflake deployment requires 2024_08 behavior change bundle
- **Fill-out Tags**: Generated YAML files contain `# <FILL-OUT>` tags that must be completed

## Testing Approach

- Unit tests in `semantic_model_generator/tests/`
- Test data and samples in `tests/samples/`
- Validation includes schema validation and context length checks