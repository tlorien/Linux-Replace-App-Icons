# Linux-Replace-App-Icons
A simple bash script to customize desktop icons in Linux (Debian-based systems).

```
#!/bin/bash

# Define the base directories where icons are located
USER_BASE_DIR="$HOME/.icons/hicolor"
SYSTEM_BASE_DIR="/usr/share/icons/hicolor"

# Ensure the script is run as root to modify files in /usr/share/icons/hicolor
if [ "$(id -u)" -ne 0 ]; then
    echo "This script must be run with root privileges to access the icons folder."
    exit 1
fi

# Check if Inkscape is installed
if ! command -v inkscape &> /dev/null; then
    echo "Inkscape is not installed. Please install it and try again. (sudo apt install inkscape)"
    exit 1
fi

# Input SVG file filepath
INPUT_SVG="$1"
if [ -z "$INPUT_SVG" ]; then
    echo "Usage: $0 /path/to/input.svg"
    exit 1
fi

# Validate that the input SVG file exists
if [ ! -f "$INPUT_SVG" ]; then
    echo "Input SVG file does not exist."
    exit 1
fi

# Array to hold changes for confirmation
declare -a changes

# Function to process resolution directories
process_resolutions() {
    local base_dir="$1"
    
    for resolution_dir in "$base_dir"/*x*; do
        # Check if the resolution folder contains an 'apps' subfolder
        if [ -d "$resolution_dir/apps" ]; then
            echo "Found apps folder in $resolution_dir"

            # Extract the resolution size (e.g., 16x16)
            resolution=$(basename "$resolution_dir")

            # Get the name of the input SVG file (without the .svg extension)
            base_name=$(basename "$INPUT_SVG" .svg)

            # Define the target PNG file path in the current resolution folder
            target_png="$resolution_dir/apps/$base_name.png"

            # Check if the resolution folder already contains a PNG file to replace
            if [ -f "$target_png" ]; then
                echo "Found existing $target_png. It will be replaced."
                changes+=("Replace: $target_png")
            else
                echo "No existing $target_png found. It will be created."
                changes+=("Create: $target_png")
            fi
        else
            echo "No 'apps' folder found in $resolution_dir. Skipping."
        fi
    done
}

# Process both user and system directories
process_resolutions "$USER_BASE_DIR"
process_resolutions "$SYSTEM_BASE_DIR"

# Output the synopsis of changes
echo ""
echo "Synopsis of changes:"
for change in "${changes[@]}"; do
    echo "$change"
done

# Ask for confirmation before proceeding
read -p "Do you want to proceed with these changes? (y/n): " confirm
if [[ ! "$confirm" =~ ^[Yy]$ ]]; then
    echo "Operation cancelled."
    exit 0
fi

# Execute the changes
for resolution_dir in "$USER_BASE_DIR"/*x* "$SYSTEM_BASE_DIR"/*x*; do
    if [ -d "$resolution_dir/apps" ]; then
        resolution=$(basename "$resolution_dir")
        base_name=$(basename "$INPUT_SVG" .svg)
        target_png="$resolution_dir/apps/$base_name.png"

        # Replace or create the PNG file
        inkscape "$INPUT_SVG" --export-type=png --export-filename="$target_png" --export-width="${resolution%x*}" --export-height="${resolution#*x}"
        
        # Confirm creation
        if [ -f "$target_png" ]; then
            echo "Successfully created $target_png"
        else
            echo "Failed to create $target_png"
        fi
    fi
done

echo "Icon update process complete!"
```
