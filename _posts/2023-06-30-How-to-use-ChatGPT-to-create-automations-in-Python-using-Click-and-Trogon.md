---
layout: post
title: How to use ChatGPT to create automations in Python using Click and Trogon
date: 2023-06-30
categories: [Python, Click, Trogon, ChatGPT]
---
I've been using ChatGPT to create automations in Python using the Click and Trogon. For example, I might say:

"I would like a Python Click script that combines all of the files in a target directory into a single file."

When I ask ChatGPT exactly this, I get:

```python
@click.command()
@click.argument('directory', type=click.Path(exists=True))
@click.argument('output_file', type=click.Path())
def combine_files(directory, output_file):
    # Get a list of all files in the target directory
    file_list = os.listdir(directory)
    
    # Sort the files alphabetically to ensure consistent order
    file_list.sort()
    
    # Open the output file in write mode
    with open(output_file, 'w') as output:
        # Iterate over each file in the directory
        for filename in file_list:
            # Construct the full file path
            file_path = os.path.join(directory, filename)
            
            # Check if the current item is a file (not a subdirectory)
            if os.path.isfile(file_path):
                # Read the contents of the file
                with open(file_path, 'r') as file:
                    file_contents = file.read()
                
                # Write the contents to the output file
                output.write(file_contents)
    
    click.echo(f"All files in '{directory}' have been combined into '{output_file}'.")

if __name__ == '__main__':
    combine_files()
```

Then I copy the output from ChatGPT into a script that have called automations.py where I'm beginning to grow a collection of these little automations. In automations.py I have a Click group that looks like:

```python
@tui()
@click.group()
def automation():
    pass
```

The `@tui` decorator comes from the Trogon library that analyzes the Click decorators to build a Terminal User Interface. Click is a Python library used for creating CLI's (Command Line Interfaces) and Trogon is built by the folks at Textualize and uses their Textual library to build the TUI.

Now I have to change the decorator `@click.command()` to `@automation.command()` so that the Python function is nested under my automation group. I've also changed:

```python
if __name__ == '__main__':
    automation()
```

The final output of the script looks like:

```python
import os
import click

from trogon import tui

@tui()
@click.group()
def automation():
    pass


@automation.command()
@click.argument('directory', type=click.Path(exists=True))
@click.argument('output_file', type=click.Path())
def combine_files(directory, output_file):
    # Get a list of all files in the target directory
    file_list = os.listdir(directory)
    
    # Sort the files alphabetically to ensure consistent order
    file_list.sort()
    
    # Open the output file in write mode
    with open(output_file, 'w') as output:
        # Iterate over each file in the directory
        for filename in file_list:
            # Construct the full file path
            file_path = os.path.join(directory, filename)
            
            # Check if the current item is a file (not a subdirectory)
            if os.path.isfile(file_path):
                # Read the contents of the file
                with open(file_path, 'r') as file:
                    file_contents = file.read()
                
                # Write the contents to the output file
                output.write(file_contents)
    
    click.echo(f"All files in '{directory}' have been combined into '{output_file}'.")


if __name__ == '__main__':
    automation()
```

Now I can run:
`python3 automations.py tui`
You can also use the help flag:
```bash
python3 automations.py --help
Usage: automations.py [OPTIONS] COMMAND [ARGS]...

Options:
  --help  Show this message and exit.

Commands:
  combine-files
  tui            Open Textual TUI.
```

To make things easier for myself, I added the following couple of lines to my .zshrc file:

```bash
export PATH="/path/to/automations.py:$PATH"
alias automations="automations.py"
```

Now I can run
`automations tui`
from anywhere.
