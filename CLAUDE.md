# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

eyecite is a Python library for extracting legal citations from text. It's used to process millions of legal documents for CourtListener and Harvard's Caselaw Access Project. The library recognizes various types of citations including full case citations, references, short cases, statutory citations, law journals, supra, and id. citations.

## Development Commands

### Setup and Installation
```bash
# Install with uv (preferred)
uv sync --frozen

# Install with pip
pip install -e .

# Install with dev dependencies using uv
uv sync --frozen --dev

# Install optional hyperscan dependency for faster tokenization
pip install hyperscan
```

### Running Tests
```bash
# Run all tests
python -m unittest discover -s tests -p 'test_*.py'

# Run specific test file
python -m unittest tests.test_FindTest

# Run specific test method
python -m unittest tests.test_FindTest.FindTest.test_find_citations
```

### Linting and Type Checking
```bash
# Run ruff linter
ruff check .
ruff format .

# Run mypy type checker
mypy .

# Run pre-commit hooks
pre-commit run --all-files
```

### Documentation
```bash
# Generate API documentation (requires pdoc3)
cd api_documentation
./generate_documentation.sh
```

## Architecture Overview

### Core Components

1. **Citation Extraction Pipeline**:
   - `tokenizers.py`: Converts text to tokens using regex patterns. Two implementations:
     - `AhocorasickTokenizer` (default): Uses pyahocorasick for filtering regexes
     - `HyperscanTokenizer`: Faster alternative using hyperscan (optional dependency)
   - `find.py`: Main entry point via `get_citations()` - extracts citations from tokens
   - `models.py`: Defines citation types (FullCaseCitation, IdCitation, SupraCitation, etc.)

2. **Post-Processing**:
   - `resolve.py`: Groups citations by common references (resolves id/supra to full citations)
   - `annotate.py`: Inserts markup around citations in original text using diffing algorithm
   - `clean.py`: Pre-processes text (removes HTML, normalizes whitespace, etc.)

3. **Data Sources**:
   - Uses `reporters-db` for reporter abbreviations and patterns
   - Uses `courts-db` for court information

### Key Design Patterns

1. **Token-Based Processing**: Text is first tokenized into CitationToken, IdToken, StopWordToken, etc. before citation extraction
2. **Span Tracking**: Each citation tracks its position in source text via `span()` method for accurate annotation
3. **Metadata Extraction**: Citations include metadata like year, court, pin cites, parentheticals
4. **Flexible Resolution**: Custom resolution functions can be provided to link citations to external databases

### Testing Approach

- Comprehensive test suite with real-world citation examples
- Mock factories in `test_factories.py` for creating test citations
- Tests organized by functionality: FindTest, ResolveTest, AnnotateTest, etc.
- Benchmark tests for performance tracking

## Important Implementation Notes

1. **Text Cleaning**: When using `clean_text()`, the same cleaned text must be used for both extraction and annotation to maintain correct spans
2. **HTML Handling**: The `annotate_citations()` function handles unbalanced HTML tags with `unbalanced_tags` parameter
3. **Reference Citations**: Two-step process - first extract full citations, then use `extract_reference_citations()` with resolved case names
4. **Performance**: HyperscanTokenizer is significantly faster but requires compilation and optional dependency

## Version Management

Version is managed in `pyproject.toml` using semantic versioning. Update with:
```bash
uv version --bump [major|minor|patch]
```

## CI/CD

- Tests run on Python 3.10, 3.11, 3.12 via GitHub Actions
- Linting via ruff and mypy
- Pre-commit hooks enforce code quality
- Automated deployment to PyPI on version tags (vX.Y.Z)
- Benchmark tests compare PR performance against main branch