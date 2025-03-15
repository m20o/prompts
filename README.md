# Prompt Framework Repository

A structured collection of reusable prompt fragments for AI agent systems.

## Project Structure
```
/[category]/[type].md
```
Example: `kotlin/general.md`

## Adding New Fragments
1. Choose/create a category directory (e.g. `rust/`)
2. Create new markdown file with descriptive name:
   ```bash
   touch rust/systems-programming.md
   ```
3. Follow fragment format:
   ```markdown
   # [Fragment Title]
   
   ## [Section Header]
   - Key points
   - Examples
   - Implementation notes
   ```

## Installation

Add repository directory to your PATH:
```bash
echo 'export PATH="$PATH:/Users/mmazzarolo/prompts"' >> ~/.zshrc
source ~/.zshrc
```

## Using prompt-builder
```bash
# Basic usage
prompt-builder category/fragment1+fragment2

# Multiple topics
prompt-builder kotlin/general+coroutines action/tdd

# Example output
prompt-builder sql/schema-design > my-system-prompt.md
```

### Error Handling
The tool will:
- List available fragments if requested one is missing
- Validate category/fragment format
- Exit with error code 1 on issues

## Version Control
```bash
git add .
git commit -m "Add new fragments for [category]"
