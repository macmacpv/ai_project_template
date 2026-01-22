# Project: ai_app_template; File: /README.md

# AI Agent Based App Template

**[INSTRUCTION FOR USER]:**
To initialize your AI Teammate, **COPY** the entire content of this file (and any other project files you have created) and **PASTE** it into your AI Chat.

---

## 1. Multi-Agent System Protocol

This project utilizes a multi-agent approach to development. Depending on the current need, the AI will assume one of the following specific roles.

**[CRITICAL INSTRUCTION FOR AI]:**
After ingesting the project context, you must **IMMEDIATELY ASK** the user which agent role you should assume for the session. Once a role is assigned, you **MUST STICK TO IT** completely until instructed to switch.

### 🧠 Main Developer Agent (MDA)
*   **Focus:** Architecture, Core Logic, System Consistency, Integration.
*   **Personality:** Senior Software-Architect. Careful, thoughtful, focused on long-term maintainability and patterns.
*   **Responsibilities:**
    *   Designing file structures and API interfaces.
    *   Writing complex core logic.
    *   Ensuring new features don't break existing systems.
    *   Maintaining the "Big Picture".

### 💡 Feature Consultant Agent (FCA)
*   **Focus:** Brainstorming, specialized knowledge, optimization, user experience.
*   **Personality:** Creative Technologist & Domain Expert. Innovative, suggests best practices, looks for "wow" factors.
*   **Responsibilities:**
    *   Proposing new features or improvements.
    *   Advising on specific libraries or algorithms.
    *   Refining requirements from vague ideas.
    *   Solving specific, isolated conceptual problems.

### 🛠️ Task Execution Agent (TEA)
*   **Focus:** Script running, tedious tasks, refactoring, testing, documentation.
*   **Personality:** Efficient DevOps Engineer. Precise, fast, pragmatic. "Get it done."
*   **Responsibilities:**
    *   Writing unit tests.
    *   Refactoring code for readability/PEP8 compliance.
    *   Generating documentation strings.
    *   Running repetitive terminal commands or file manipulations.

---

## 2. Project Lifecycle Phases

The AI (regardless of role) should guide the project through these phases.

### 🟡 Phase 1: Initialization
*   **Goal:** Establish the project identity and technical foundation.
*   **Steps:**
    1.  **Ingest Context:** Read `tools/` and existing docs.
    2.  **Define Profile:** User fills out `app_docs/meta/project.md`.
    3.  **Setup Environment:** Create virtual environment (`tools/setup_venv.py`).
    4.  **Skeleton:** Create the basic folder structure based on `project.md` requirements.

### 🟢 Phase 2: Core Development (MVP)
*   **Goal:** Build the Minimum Viable Product defined in the Project Vision.
*   **Steps:**
    1.  **Iterative Coding:** MDA builds features one by one.
    2.  **Verification:** TEA runs tests and checks consistency.
    3.  **Completion:** All "Key Features" from `project.md` are implemented and working.

### 🟣 Phase 3: Infinite Expansion
*   **Goal:** Continuous improvement, scaling, and polishing.
*   **Duration:** **Indefinite.** This phase never ends automatically.
*   **Protocol:**
    *   The project is "live".
    *   We continuously cycle through: **Review -> Plan -> Implement -> Verify**.
    *   FCA suggests optimizations or new tech.
    *   MDA integrates them.
    *   TEA maintains quality.
    *   **Loop:** Continue adding features and refining until the User explicitly says "PROJECT ENDS HERE".

---

## 3. Toolset & Standards

*   **Context Management:** Use `tools/list_project_tree.py` and `tools/serializer.py` to allow the AI to read the codebase.
*   **File Creation:** Use `tools/deserializer.py` to write AI-generated code to disk.
*   **Locations:**
    *   `app/`: Source code.
    *   `app_docs/`: Documentation and plans.

---

## 4. Operational Rules (CRITICAL)

### 🛡️ Rule 1: Tool Immutability
The `tools/` directory is **STRICTLY READ-ONLY** for all agents.
*   **Do NOT** modify `deserializer.py`, `serializer.py`, etc.
*   **Reason:** These are the "Means of Production". Modifying them risks breaking the agent's ability to communicate or function.

### 📝 Rule 2: Mandatory Logging
Every agent must maintain a work log.
*   **Location:** `app_docs/ai_logs/`
*   **Filename:** `<AGENT_PREFIX>_work_log.md` (e.g., `MDA_work_log.md`, `TEA_work_log.md`).
*   **Content:**
    *   Append a new entry for every session/iteration.
    *   Record: Date, Goal, Actions Taken, Issues Encountered, and Next Steps.

### 📦 Rule 3: Output Serialization
**MDA** and **FCA** (and TEA if generating code) **MUST** use the following format when presenting file content. This allows the user to automatically save your work using `tools/deserializer.py`.

**The Standard Serializable Format:**
```text
# Project: [Name]; File: [Relative_Path_From_Root]
[Content]
---[END OF FILE: [Relative_Path_From_Root]]---
```
*   **Exceptions:** CLI commands, explanations, or terminal logs do not need this format. Only code that needs to be saved to a file.


---[END OF FILE: /README.md]---

# Project: ai_app_template; File: /app_docs/meta/project.md

# Project Profile

## 1. Project Owner
*   **Name:** [Your Name]
*   **Role:** Product Owner / Lead Developer
*   **Technical Background:**
    *   [Briefly describe your experience, preferred stack, and any constraints]

## 2. Project Vision
*   **Project Name:** [Project Name]
*   **Type:** [e.g., AI Agent System, Web Application, Automation Tool, CLI Utility]
*   **Description:**
    *   [Detailed description of what you are building. What problem does it solve? Who is it for?]
    *   [Key Goals and Objectives]

## 3. Technical Requirements
*   **Core Stack:** [e.g., Python, Next.js, OpenAI API, LangChain]
*   **Key Features (MVP):**
    *   [Feature 1]
    *   [Feature 2]
    *   [Feature 3]
*   **Future Scope:**
    *   [Ideas for later expansion]

## 4. AI Interaction Guidelines
*   **Preferred Coding Style:** [e.g., Functional, OOP, specific linters]
*   **Documentation Standards:** [e.g., Google Style Docstrings, Markdown files]


---[END OF FILE: /app_docs/meta/project.md]---

# Project: ai_app_template; File: /tools/deserializer.py

import os
import re
import argparse

"""
The Context Deserializer

Purpose:
This script allows the AI to write files to your hard drive.
When the AI generates a code block containing multiple files (in the 
Standard Serializable Format), you paste that block into a text file,
and this script splits it up and saves the individual files.

The Format it expects:
# Project: Name; File: /path/to/file.txt
[File Content]
---[END OF FILE: /path/to/file.txt]---

Usage:
python tools/deserializer.py SERIALIZED_CONTEXT.txt
"""

def strip_markdown_artifacts(content):
    """
    Removes Markdown code block wrappers (``` or ```text) if the user
    accidentally copied them along with the content.
    """
    content = content.strip()
    
    if content.startswith("```"):
        content = re.sub(r"^```[^\n]*\n", "", content)
        
        if content.endswith("```"):
            content = content[:-3].strip()
            
    return content

def parse_serialized_context(input_file_path):
    """
    Reads the big text file and uses Regex to find the start and end
    of each file block.
    """
    files_data = []
    try:
        with open(input_file_path, 'r', encoding='utf-8') as f:
            raw_content = f.read()

        clean_content = strip_markdown_artifacts(raw_content)

        header_pattern = re.compile(r"^(?:\*\*)?#\s*Project:.*;\s*File:\s*(.*?)(?:\*\*)?\s*$", re.MULTILINE)
        
        matches = list(header_pattern.finditer(clean_content))
        
        for i, match in enumerate(matches):
            file_path = match.group(1).strip()
            start_index = match.end()
            
            footer_string = f"---[END OF FILE: {file_path}]---"
            
            footer_pos = clean_content.find(footer_string, start_index)
            
            if footer_pos != -1:
                file_content = clean_content[start_index:footer_pos].strip()
                
                if file_path:
                    files_data.append((file_path, file_content))
            else:
                print(f"WARNING: Found header for {file_path} but no matching footer. Skipping.")

        return files_data

    except Exception as e:
        print(f"ERROR: Failed to parse input file: {e}")
        return []

def write_files_to_workspace(files_data, repo_root):
    """
    Takes the parsed data and actually writes the files to disk.
    It will create directories if they don't exist.
    """
    if not files_data:
        print("No valid file blocks found to write.")
        return

    print(f"Writing files to: {repo_root}")

    for file_path, content in files_data:
        relative_path = file_path.lstrip('/').lstrip('\\')
        absolute_path = os.path.join(repo_root, relative_path)

        try:
            os.makedirs(os.path.dirname(absolute_path), exist_ok=True)

            with open(absolute_path, 'w', encoding='utf-8') as f:
                f.write(content)

            print(f"Updated: {relative_path}")

        except Exception as e:
            print(f"ERROR writing {relative_path}: {e}")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Splits a serialized text file into actual files.")
    parser.add_argument("input_file", help="Path to the text file containing the code blocks (e.g., SERIALIZED_CONTEXT.txt)")
    args = parser.parse_args()
    REPO_ROOT = os.getcwd()
    
    input_file_abs_path = os.path.abspath(args.input_file)
    
    parsed_data = parse_serialized_context(input_file_abs_path)
    
    write_files_to_workspace(parsed_data, REPO_ROOT)
    

---[END OF FILE: /tools/deserializer.py]---

# Project: ai_app_template; File: /tools/list_project_tree.py

import os
import sys

"""
Project File Finder (The "Map Maker")

Purpose:
This script scans your project directory and creates a clean list of all 
relevant files. This list is used by the 'ReConstructor' to build the 
AI's context.

How it works:
1. It walks through every folder in your project.
2. It ignores 'junk' folders (like .git, __pycache__, node_modules).
3. It ignores binary files (images, zips) that the AI can't read.
4. It prints the clean paths to the console (standard output).

Usage:
python tools/list_project_tree.py > reconstructor-file-list.txt
"""

def list_project_files(startpath):
    """
    Scans the project directory and prints a clean list of file paths.
    Intended to be piped into 'reconstructor-file-list.txt'.
    """

    ignore_dirs = {
        '.git', '.venv', '__pycache__', '.vscode', 'node_modules', 
        'build', 'dist', 'X_Intels', 'X_Repos'
    }
    
    ignore_files = {
        '.DS_Store', 'Thumbs.db', 'requirements.txt',
        'SERIALIZED_CONTEXT.txt', 'RECONSTRUCTOR_CONTEXT.md', 'START_CONTEXT.md',
        'reconstructor-file-list.txt', '_SESSION_ADDENDUM_BUFFER.md',
        'package-lock.json', 'yarn.lock', 'pnpm-lock.yaml', 'yarn-error.log', 
        'pnpm-error.log', 'yarn-error.log', '.gitignore', '.gitkeep'
    }
    
    ignore_extensions = {
        '.png', '.jpg', '.jpeg', '.gif', '.ico', 
        '.pdf', '.zip', '.7z', '.exe', '.dll', 
        '.pyc', '.blend', '.fbx', '.obj'
    }

    for root, dirs, files in os.walk(startpath, topdown=True):
        dirs[:] = [d for d in dirs if d not in ignore_dirs]

        for name in files:
            if name in ignore_files:
                continue
            
            _, ext = os.path.splitext(name)
            if ext.lower() in ignore_extensions:
                continue
                
            full_path = os.path.join(root, name)
            
            relative_path = os.path.relpath(full_path, startpath)
            
            clean_path = relative_path.replace(os.sep, '/')
            
            print(clean_path)

if __name__ == "__main__":
    project_root = "."
    
    if len(sys.argv) > 1:
        project_root = sys.argv[1]
        
    list_project_files(project_root)
    

---[END OF FILE: /tools/list_project_tree.py]---

# Project: ai_app_template; File: /tools/serializer.py

import os
import sys

"""
The Context Consolidator (The "ReConstructor")

Purpose:
This script builds the "Boot Disk" for your AI. It reads the list of files
generated by `list_project_tree.py` and combines their contents into a single
Markdown file (`RECONSTRUCTOR_CONTEXT.md`).

Why do we need this?
AI Context Windows are limited. Pasting files one by one is slow and error-prone.
This script formats everything into a standardized block that the AI can ingest
instantly to understand the entire project state.

Usage:
python tools/serializer.py
"""

def get_project_name_from_repo_root(repo_root_path):
    """
    Attempts to guess the project name from the folder name.
    Used for the file headers.
    """
    try:
        project_name = os.path.basename(repo_root_path)
        if project_name:
            return project_name
    except Exception:
        pass
    return "AI-Assisted-Student-Project"

def create_consolidated_context(file_list_path, output_file_path):
    """
    Reads a list of files and consolidates them into a single Markdown file
    with headers/footers for AI ingestion.
    """
    print(f"Starting ReConstructor context consolidation...")
    print(f"Reading file list from: {file_list_path}")

    file_paths = []
    encodings_to_try = ['utf-8', 'utf-16', 'cp1252']
    
    file_list_read_success = False
    for encoding in encodings_to_try:
        try:
            with open(file_list_path, 'r', encoding=encoding) as f:
                file_paths = [line.strip() for line in f if line.strip() and not line.strip().startswith('#')]
            file_list_read_success = True
            break
        except UnicodeDecodeError:
            continue
        except FileNotFoundError:
            print(f"CRITICAL ERROR: Could not find file list '{file_list_path}'")
            print("Did you run 'list_project_tree.py' first?")
            return

    if not file_list_read_success:
        print(f"ERROR: Could not decode {file_list_path} with any of {encodings_to_try}.")
        return

    if not file_paths:
        print("ERROR: No file paths found in the list.")
        return

    repo_root = os.getcwd()
    project_name = get_project_name_from_repo_root(repo_root)

    try:
        with open(output_file_path, 'w', encoding='utf-8') as outfile:
            for file_path in file_paths:
                full_file_path = os.path.join(repo_root, file_path)
                
                header_path = file_path.replace(os.sep, '/')
                if not header_path.startswith('/'):
                    header_path = '/' + header_path

                try:
                    # Try reading file content with multiple encodings
                    content = ""
                    read_success = False
                    for enc in ['utf-8', 'utf-16', 'cp1252']:
                        try:
                            with open(full_file_path, 'r', encoding=enc) as infile:
                                content = infile.read()
                            read_success = True
                            break
                        except UnicodeDecodeError:
                            continue
                    
                    if not read_success:
                         print(f"WARNING: Skipping {file_path} (Encoding error)")
                         continue

                    outfile.write(f"# Project: {project_name}; File: {header_path}\n\n")
                    outfile.write(content)
                    outfile.write(f"\n\n---[END OF FILE: {header_path}]---\n\n")
                    
                    print(f"Processed: {file_path}")

                except FileNotFoundError:
                    print(f"WARNING: File listed in text file but not found on disk: {file_path}")
                except Exception as e:
                    print(f"ERROR processing file {file_path}: {e}")

        print(f"\nSuccess! Context created at: {output_file_path}")
        print("You can now copy the content of this file and paste it to your AI.")

    except Exception as e:
        print(f"ERROR writing output file: {e}")

if __name__ == "__main__":
    FILE_LIST = 'reconstructor-file-list.txt'
    OUTPUT_FILE = 'RECONSTRUCTOR_CONTEXT.md'
    
    if len(sys.argv) > 1:
        FILE_LIST = sys.argv[1]

    create_consolidated_context(FILE_LIST, OUTPUT_FILE)
    

---[END OF FILE: /tools/serializer.py]---

# Project: ai_app_template; File: /tools/setup_venv.py

import os
import sys
import subprocess
import platform

"""
Virtual Environment Setup Tool

Purpose:
Creates a focused Python environment (.venv) for this project.
This ensures your project tools run correctly without interfering
with other programs on your computer.

Usage:
python tools/setup_venv.py
"""

def create_venv():
    venv_dir = os.path.join(os.getcwd(), ".venv")
    
    if os.path.exists(venv_dir):
        print(f"✅ Virtual environment already exists at: {venv_dir}")
    else:
        print(f"⚙️  Creating virtual environment at: {venv_dir}...")
        try:
            subprocess.check_call([sys.executable, "-m", "venv", ".venv"])
            print("✅ Virtual environment created successfully.")
        except subprocess.CalledProcessError:
            print("❌ Error: Failed to create virtual environment.")
            return

    req_file = "requirements.txt"
    if os.path.exists(req_file):
        print(f"📦 Installing dependencies from {req_file}...")
        
        if platform.system() == "Windows":
            pip_exe = os.path.join(venv_dir, "Scripts", "pip")
        else:
            pip_exe = os.path.join(venv_dir, "bin", "pip")
            
        try:
            subprocess.check_call([pip_exe, "install", "-r", req_file])
            print("✅ Dependencies installed.")
        except Exception as e:
            print(f"⚠️  Warning: Could not install dependencies. Error: {e}")
            print("You may need to install them manually.")
    
    print("\n" + "="*50)
    print("🎉 SETUP COMPLETE!")
    print("="*50)
    print("To make VS Code use this environment:")
    print("1. Press 'Ctrl + Shift + P' (Cmd+Shift+P on Mac)")
    print("2. Type: 'Python: Select Interpreter'")
    print("3. Choose the one marked ('.venv': venv)")
    
    print("\nTerminal activation command:")
    if platform.system() == "Windows":
        print(r"  .venv\Scripts\activate")
    else:
        print("  source .venv/bin/activate")
    print("="*50)

if __name__ == "__main__":
    create_venv()
    

---[END OF FILE: /tools/setup_venv.py]---

