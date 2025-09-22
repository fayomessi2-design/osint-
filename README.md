import os
import hashlib
import time
import json
import sys

# Description: This is an isolated Python code that simulates a simple website system with 4 users and 4 files.
# It creates users and files, allows simulated user modifications to files, and detects changes by comparing hashes.
# When a change is detected, it shows the old hash, new hash, modification time, and the name of the user who made the change.
# The system uses file system for storage (no database or web server needed).
# Users are stored in 'users.json', files in 'files/' directory, hashes in 'file_hashes.json', and change logs in 'change_logs.json'.
# Run the script and follow the prompts to interact (e.g., modify files as a user).

# Part 1: Constants and Setup
USERS_FILE = 'users.json'  # File to store user data
FILES_DIR = 'files'  # Directory to store the 4 files
HASH_FILE = 'file_hashes.json'  # File to store initial and updated hashes
CHANGE_LOG_FILE = 'change_logs.json'  # File to log changes

# Create directories if not exist
os.makedirs(FILES_DIR, exist_ok=True)

# Part 2: Functions for Hash Calculation
def compute_hash(file_path):
    """Description: Calculates the SHA-256 hash of a file's content. This is used to detect changes by comparing old and new hashes."""
    hasher = hashlib.sha256()
    try:
        with open(file_path, 'rb') as f:
            while chunk := f.read(4096):
                hasher.update(chunk)
        return hasher.hexdigest()
    except Exception as e:
        print(f"Error computing hash for {file_path}: {e}")
        return None

# Part 3: Functions for User Management
def create_users():
    """Description: Creates 4 simulated users and saves them to 'users.json'. Each user has a name and password (for simulation)."""
    users = {
        "user1": {"name": "Alice", "password": "pass1"},
        "user2": {"name": "Bob", "password": "pass2"},
        "user3": {"name": "Charlie", "password": "pass3"},
        "user4": {"name": "David", "password": "pass4"}
    }
    with open(USERS_FILE, 'w') as f:
        json.dump(users, f)
    print("4 users created and saved to 'users.json'.")

def load_users():
    """Description: Loads users from 'users.json'. If file doesn't exist, returns empty dict."""
    if os.path.exists(USERS_FILE):
        with open(USERS_FILE, 'r') as f:
            return json.load(f)
    return {}

# Part 4: Functions for File Management
def create_files():
    """Description: Creates 4 files in 'files/' directory with initial content."""
    files = ["file1.txt", "file2.txt", "file3.txt", "file4.txt"]
    for file in files:
        file_path = os.path.join(FILES_DIR, file)
        with open(file_path, 'w') as f:
            f.write(f"Initial content for {file}.")
    print("4 files created in 'files/' directory.")

def load_hashes():
    """Description: Loads stored hashes from 'file_hashes.json'."""
    if os.path.exists(HASH_FILE):
        with open(HASH_FILE, 'r') as f:
            return json.load(f)
    return {}

def save_hashes(hashes):
    """Description: Saves hashes to 'file_hashes.json'."""
    with open(HASH_FILE, 'w') as f:
        json.dump(hashes, f)

def initialize_hashes():
    """Description: Computes initial hashes for the 4 files and saves them."""
    hashes = {}
    for filename in os.listdir(FILES_DIR):
        file_path = os.path.join(FILES_DIR, filename)
        if os.path.isfile(file_path):
            file_hash = compute_hash(file_path)
            if file_hash:
                hashes[filename] = file_hash
    save_hashes(hashes)
    print("Initial hashes computed and saved.")

# Part 5: Functions for Change Detection and Logging
def check_changes():
    """Description: Checks all files for changes, logs old/new hash, modification time, and user (if known)."""
    hashes = load_hashes()
    changes = load_changes()  # Load existing logs
    for filename in os.listdir(FILES_DIR):
        file_path = os.path.join(FILES_DIR, filename)
        if os.path.isfile(file_path):
            current_hash = compute_hash(file_path)
            old_hash = hashes.get(filename, None)
            if old_hash and old_hash != current_hash:
                mod_time = time.ctime(os.path.getmtime(file_path))  # Get modification time
                change = {
                    'file': filename,
                    'old_hash': old_hash,
                    'new_hash': current_hash,
                    'modification_time': mod_time,
                    'user': 'Unknown'  # Simulate user; in real system, track logged-in user
                }
                changes.append(change)
                print(f"Change detected in {filename}: Old Hash: {old_hash}, New Hash: {current_hash}, Mod Time: {mod_time}, User: {change['user']}")
                # Update hash to new one
                hashes[filename] = current_hash
    save_hashes(hashes)
    save_changes(changes)

def load_changes():
    """Description: Loads change logs from 'change_logs.json'."""
    if os.path.exists(CHANGE_LOG_FILE):
        with open(CHANGE_LOG_FILE, 'r') as f:
            return json.load(f)
    return []

def save_changes(changes):
    """Description: Saves change logs to 'change_logs.json'."""
    with open(CHANGE_LOG_FILE, 'w') as f:
        json.dump(changes, f)

# Part 6: Simulate User Modification
def simulate_user_modification():
    """Description: Simulates a user logging in and modifying a file, then detects the change."""
    users = load_users()
    if not users:
        print("No users found. Create users first.")
        return

    # List users
    print("Available users:")
    for username in users:
        print(username)

    # Simulate login
    username = input("Enter username to simulate: ").strip()
    if username not in users:
        print("Invalid user.")
        return

    # Choose file to modify
    files = [f for f in os.listdir(FILES_DIR) if os.path.isfile(os.path.join(FILES_DIR, f))]
    print("Available files:")
    for file in files:
        print(file)

    filename = input("Enter filename to modify: ").strip()
    if filename not in files:
        print("Invalid file.")
        return

    # Modify content
    new_content = input("Enter new content for the file: ").strip()
    file_path = os.path.join(FILES_DIR, filename)
    with open(file_path, 'w') as f:
        f.write(new_content)

    # Update log with user
    hashes = load_hashes()
    old_hash = hashes.get(filename, 'Unknown')
    new_hash = compute_hash(file_path)
    mod_time = time.ctime(os.path.getmtime(file_path))
    change = {
        'file': filename,
        'old_hash': old_hash,
        'new_hash': new_hash,
        'modification_time': mod_time,
        'user': users[username]['name']
    }
    changes = load_changes()
    changes.append(change)
    save_changes(changes)
    hashes[filename] = new_hash
    save_hashes(hashes)
    print(f"File {filename} modified by {change['user']}. Change logged.")

# Part 7: Main Function to Run the System
def main():
    """Description: Main entry point to create users/files and simulate modifications."""
    create_users()
    create_files()
    initialize_hashes()
    while True:
        print("\nOptions:")
        print("1. Simulate user modification")
        print("2. Check for changes")
        print("3. View change logs")
        print("4. Exit")
        choice = input("Enter choice: ").strip()
        if choice == '1':
            simulate_user_modification()
        elif choice == '2':
            check_changes()
        elif choice == '3':
            changes = load_changes()
            if changes:
                for change in changes:
                    print(change)
            else:
                print("No changes logged.")
        elif choice == '4':
            break
        else:
            print("Invalid choice.")

if __name__ == '__main__':
    main()
