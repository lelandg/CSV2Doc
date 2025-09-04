# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
CSV2Doc is a Streamlit web application that converts CSV files to multiple document formats (CSV, HTML, DOCX) with advanced data manipulation features. The application is deployed at https://csv2doc.streamlit.app/.

## Development Commands

### Running the Application
```bash
# Run the Streamlit app locally
streamlit run app.py

# Run with specific port
streamlit run app.py --server.port 8501
```

### Dependency Management
```bash
# Install dependencies
pip install -r requirements.txt

# Create/activate virtual environment (recommended)
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

## Code Architecture

### Single-File Application Structure
The entire application is contained in `app.py` (1,022 lines) with the following key sections:

1. **User Management (lines ~50-150)**: UUID-based session tracking with query parameter persistence
2. **Data Caching (lines ~200-300)**: File-based CSV caching using hash identifiers in `data/csv_cache/`
3. **File Processing (lines ~400-600)**: Core CSV manipulation logic (grouping, sorting, filtering)
4. **Export Functions (lines ~700-900)**: Document generation for CSV, HTML, and DOCX formats
5. **UI Components (lines ~900-1022)**: Streamlit interface with sidebar controls and main display

### Key Functions
- `get_user_id()`: Manages user session persistence
- `cache_uploaded_file()`: Handles file caching with hash-based naming
- `generate_grouped_tables()`: Core data grouping logic
- `generate_html()`, `generate_docx()`: Format-specific export functions
- `display_grouped_tables()`: Main UI rendering function

### Data Flow
1. User uploads CSV → cached in `data/csv_cache/` with hash filename
2. Processing options saved to `data/csv_cache/{hash}_options.json`
3. Data manipulated using pandas based on user selections
4. Export generated on-demand without additional caching
5. User history tracked in `data/users/{user_id}/history.json`

## Directory Structure
```
data/
├── csv_cache/         # Cached CSV files and processing options
├── users/            # User-specific data and history
│   └── {user_id}/
│       ├── history.json
│       └── options/
└── css_styles/       # Reserved for future CSS customization
```

## Important Development Notes

### Streamlit-Specific Patterns
- Use `st.session_state` for maintaining state across reruns
- File uploads trigger full page rerun - cache operations accordingly
- Query parameters (`st.query_params`) used for user session persistence
- Sidebar used for all control inputs, main area for data display

### Data Processing Approach
- All CSV manipulation done with pandas DataFrames
- Grouping creates separate tables, not aggregated data
- Sorting/filtering applied after grouping operations
- Large files may cause memory issues (no streaming implementation)

### Export Formats
- **CSV**: Direct pandas to_csv() export
- **HTML**: Custom table generation with inline styling
- **DOCX**: python-docx library for Word document creation

### Known Limitations
- No test suite currently exists
- File-based storage not suitable for multi-instance deployment
- No database integration for persistent user data
- All processing happens in memory (no streaming for large files)

## Testing Approach
Currently no tests exist. When implementing tests:
- Unit test individual functions (grouping, sorting, filtering logic)
- Integration test the full upload-process-export flow
- Use pytest with fixtures for sample CSV data
- Mock Streamlit components for UI testing

## Deployment
Application is deployed on Streamlit Cloud. For deployment updates:
- Push changes to main branch
- Streamlit Cloud auto-deploys from GitHub repository
- Check logs at https://share.streamlit.io/ dashboard