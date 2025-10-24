Remove-Item -Recurse -Force build -ErrorAction SilentlyContinue
cmake -S . -B build
cmake --build build









# Markdown Document Builder with CMake

This project uses **CMake** and **Pandoc** to automatically convert Markdown files to PDF and DOCX formats. It automatically detects all `.md` files in the directory and creates build targets for each one.

## Prerequisites

- **CMake** (version 3.10 or higher)
- **Pandoc** (with LaTeX installation for PDF generation)
- **LaTeX** (for PDF generation - xelatex recommended)

## Installation

1. Ensure you have CMake and Pandoc installed on your system
2. LaTeX distribution (like MiKTeX or TeX Live) for PDF generation
3. Place your markdown files and `cite.bib` (if using citations) in the project directory

## Project Structure

```
your-project/
├── CMakeLists.txt          # This build configuration
├── cite.bib                # Bibliography file (optional)
├── lab1.md                 # Your markdown files
├── lab2.md
├── report.md
└── pandoc/                 # Custom CSL styles (optional)
    └── csl/
        └── gost-r-7-0-5-2008-numeric.csl
```

## Usage

### Initial Setup
```bash
# Configure the project (run once)
cmake -S . -B build
```

### Building Documents

**Build all documents (PDF + DOCX):**
```bash
cmake --build build
# or
cmake --build build --target build_all
```

**Build only PDF files:**
```bash
cmake --build build --target build_pdf
```

**Build only DOCX files:**
```bash
cmake --build build --target build_docx
```

**Build specific files:**
```bash
# For a file named "lab2.md"
cmake --build build --target lab2_pdf    # Build only lab2.pdf
cmake --build build --target lab2_docx   # Build only lab2.docx
```

### Cleaning Generated Files
```bash
cmake --build build --target clean_all
```

## Features

- **Automatic File Detection**: Finds all `.md` files in the directory
- **Dual Output**: Generates both PDF and DOCX for each markdown file
- **Citation Support**: Automatically processes citations from `cite.bib`
- **Numbered Sections**: Includes section numbering in outputs
- **Individual Targets**: Build specific files or formats as needed
- **Cross-Platform**: Works on Windows, Linux, and macOS

## Available Targets

After running `cmake -S . -B build`, you'll see a list of available targets like:
```
Available targets:
  cmake --build build --target build_all    # Build all documents
  cmake --build build --target build_pdf    # Build all PDFs
  cmake --build build --target build_docx   # Build all DOCXs
  cmake --build build --target lab2_pdf     # Build lab2.pdf
  cmake --build build --target lab2_docx    # Build lab2.docx
  cmake --build build --target clean_all    # Clean all generated files
```

## Markdown File Requirements

Your markdown files should include YAML metadata for proper processing:

```yaml
---
title: "Your Document Title"
author: "Your Name"
date: "2024-01-01"
bibliography: cite.bib
csl: pandoc/csl/gost-r-7-0-5-2008-numeric.csl  # Optional
---
```

## Customization

### Changing PDF Engine
Edit the `CMakeLists.txt` and modify the `--pdf-engine` option:
- `xelatex` (default) - Good Unicode support
- `lualatex` - Better Unicode handling
- `pdflatex` - Standard LaTeX engine
- `wkhtmltopdf` - HTML-based PDF generation

### Adding Custom Filters
You can add pandoc filters by modifying the commands in `CMakeLists.txt`:
```cmake
COMMAND ${PANDOC_EXECUTABLE} 
        ${MD_FILE}
        --filter pandoc-crossref
        --filter pandoc-citeproc
        --pdf-engine=xelatex
        -o ${BASE_NAME}.pdf
```

## Troubleshooting

### Common Issues

1. **LaTeX Errors**: Check for raw LaTeX commands in markdown body
2. **Missing Citations**: Ensure `cite.bib` exists and citations are correct
3. **Unicode Issues**: Switch to `lualatex` PDF engine for better Unicode support
4. **Path Issues**: All files should be in the same directory or use absolute paths

### Debugging
To see what's being processed:
```bash
# List detected markdown files
cmake -S . -B build
```

## Dependencies

- **CMake**: Build system generator
- **Pandoc**: Document converter
- **LaTeX**: PDF typesetting (xelatex/lualatex recommended)
- **Bibliography**: cite.bib for reference management

This setup provides a professional, automated workflow for converting technical documentation and reports from Markdown to multiple output formats.