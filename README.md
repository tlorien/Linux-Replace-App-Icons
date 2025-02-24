# Linux-Replace-App-Icons
This bash script converts an SVG file to PNG icons for various resolutions and places them in the appropriate directories for system-wide or user-specific icons.

## Requirements

- **Inkscape**: The script uses Inkscape to convert the SVG file to PNG format. You need to have Inkscape installed to run this script.
  
  To install Inkscape on a Debian-based system, use:
  ```bash
  sudo apt install inkscape
  ```
  The script needs root access to modify files in system directories like /usr/share/icons/hicolor.
  
## Usage
### Command Syntax:

```
sudo ./this_script.sh /path/to/new_icon.svg
```

### Parameters:

**/path/to/input.svg**: Path to the SVG file that you want to replace you app icon with.

**Note:** The .SVG file MUST have the same name as the icon you wish to replace.

### Script Workflow

    Initial Setup:
        The script checks if it is run as root.
        It verifies whether Inkscape is installed on the system.

    Directory Locations:
        The script will check for icons in two directories:
            User-specific icons: $HOME/.icons/hicolor
            System-wide icons: /usr/share/icons/hicolor

    Resolution Directories:
        It searches for folders with resolutions like 16x16, 32x32, 48x48, etc., inside the 'apps' folder within these directories.
        For each resolution that contains an icon who's name matches the input .SVG, the script will replace the existing PNG icon. If not match is found in a folder then that folder will be skipped.

    Changes Confirmation:
        The script outputs a list of all changes it plans to make.
        You will be asked to confirm whether you want to proceed with these changes.

    Icon Conversion:
        If you confirm, Inkscape will be used to convert the input SVG to PNG images for each resolution folder.

## Example

To convert a my_icon.svg file into PNG icons for multiple resolutions:

```
sudo ./this_script.sh /path/to/my_icon.svg
```

You will be prompted with the changes that will be made, like:

> Found existing /usr/share/icons/hicolor/16x16/apps/icon.png. It will be replaced.
> 
> Found existing /usr/share/icons/hicolor/32x32/apps/icon.png. It will be replaced.
> 
> Found no existing /home/user/.icons/hicolor/48x48/apps/icon.png. It will be skipped.

Once confirmed, the script will proceed to convert and place the PNG icons in the respective directories.

## Script
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
                echo "No existing PNG found for $base_name in $resolution_dir. Skipping creation."
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

# Execute the changes (replace the existing PNGs)
for resolution_dir in "$USER_BASE_DIR"/*x* "$SYSTEM_BASE_DIR"/*x*; do
    if [ -d "$resolution_dir/apps" ]; then
        resolution=$(basename "$resolution_dir")
        base_name=$(basename "$INPUT_SVG" .svg)
        target_png="$resolution_dir/apps/$base_name.png"

        # Replace the PNG file if it exists
        if [ -f "$target_png" ]; then
            inkscape "$INPUT_SVG" --export-type=png --export-filename="$target_png" --export-width="${resolution%x*}" --export-height="${resolution#*x}"
            
            # Confirm replacement
            if [ -f "$target_png" ]; then
                echo "Successfully replaced $target_png"
            else
                echo "Failed to replace $target_png"
            fi
        fi
    fi
done

echo "Icon update process complete!"
```

## Troubleshooting
```
Inkscape Not Installed: If Inkscape is not found, install it using sudo apt install inkscape.
Permission Issues: Make sure the script is run with root privileges when modifying system directories.
```

## License

This script is licensed under the MIT License. Feel free to modify and use it as needed!
