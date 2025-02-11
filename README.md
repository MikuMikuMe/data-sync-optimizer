# data-sync-optimizer

Sure! Below is a Python program that implements an efficient data synchronization and optimization process between distributed systems. This program uses a simple client-server model to demonstrate data transfer, utilizes threading for concurrency, and includes error handling to ensure robustness.

```python
import socket
import threading
import sys
import hashlib
import os

BUFFER_SIZE = 1024  # Bytes
SERVER_HOST = '127.0.0.1'
SERVER_PORT = 5000

def calculate_checksum(file_path):
    """Calculate the checksum of a file for integrity verification."""
    hash_md5 = hashlib.md5()
    try:
        with open(file_path, "rb") as f:
            for chunk in iter(lambda: f.read(BUFFER_SIZE), b""):
                hash_md5.update(chunk)
    except Exception as e:
        print(f"Error calculating checksum: {e}")
        return None
    return hash_md5.hexdigest()

def send_file(file_path, server_address):
    """Send a file to the server."""
    try:
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as sock:
            sock.connect(server_address)
            with open(file_path, 'rb') as f:
                while True:
                    bytes_read = f.read(BUFFER_SIZE)
                    if not bytes_read:
                        # File transmitting is done
                        break
                    sock.sendall(bytes_read)
            checksum = calculate_checksum(file_path)
            sock.sendall(checksum.encode())
            print(f"File {file_path} sent successfully with checksum {checksum}.")
    except Exception as e:
        print(f"Error sending file {file_path}: {e}")
        return

def receive_file(connection, address, save_dir):
    """Receive a file from a client."""
    try:
        file_path = os.path.join(save_dir, f"received_from_{address[1]}.data")
        with open(file_path, 'wb') as f:
            while True:
                data = connection.recv(BUFFER_SIZE)
                if not data:
                    break
                f.write(data)
        print(f"File received and saved as {file_path}.")
    except Exception as e:
        print(f"Error receiving file from {address}: {e}")

def start_server():
    """Start the data synchronization server."""
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server_socket:
        server_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        server_socket.bind((SERVER_HOST, SERVER_PORT))
        server_socket.listen(5)
        print(f"Server listening on {SERVER_HOST}:{SERVER_PORT}")

        while True:
            try:
                client_socket, client_address = server_socket.accept()
                print(f"Connection from {client_address}")
                client_thread = threading.Thread(target=receive_file, args=(client_socket, client_address, '.'))
                client_thread.start()
            except KeyboardInterrupt:
                print("Server shutting down.")
                break
            except Exception as e:
                print(f"Error accepting connections: {e}")

def optimize_data_transfer(file_path, server_address):
    """Simulate data transfer optimization process."""
    print(f"Optimizing data transfer for {file_path}...")
    send_file_thread = threading.Thread(target=send_file, args=(file_path, server_address))
    send_file_thread.start()
    send_file_thread.join()

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: python data_sync_optimizer.py [server|client] [file_path (for client)]")
        sys.exit(1)

    mode = sys.argv[1].lower()

    if mode == 'server':
        start_server()
    elif mode == 'client':
        if len(sys.argv) != 3:
            print("Usage: python data_sync_optimizer.py client <file_path>")
            sys.exit(1)
        file_path = sys.argv[2]
        if not os.path.isfile(file_path):
            print(f"File {file_path} does not exist.")
            sys.exit(1)
        optimize_data_transfer(file_path, (SERVER_HOST, SERVER_PORT))
    else:
        print("Invalid mode. Use 'server' or 'client'.")
        sys.exit(1)
```

### Explanation:
1. **Checksum Calculation:** The `calculate_checksum()` function computes the MD5 checksum for file integrity verification.

2. **Sending and Receiving Files:** Different functions handle sending (`send_file`) and receiving (`receive_file`) files, ensuring threading for concurrency and providing error messages for any exceptions.

3. **Server and Client Modes:** The script can be run as a server or as a client by providing the appropriate arguments (`server` or `client`).

4. **Concurrency:** The program uses threading to handle multiple client connections and data sending in parallel.

5. **Error Handling:** The code includes error handling for various possible issues, such as file access errors, network errors, and argument parsing issues.

To run this script, you will need to execute it twice in different terminals (or machines), one as the server and one as the client. Ensure you have Python installed and that any network configurations are correctly set up.