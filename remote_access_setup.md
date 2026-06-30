## Remote GPU Server Access Guide
  This guide provides the steps to connect to the remote machine learning environment. We use ZeroTier to create a secure, encrypted peer-to-peer network.
Do not skip any steps. If you do not get authorised on the ZeroTier network first, the SSH connection will simply time out and fail.
# Phase 1: Install and Join ZeroTier
  ZeroTier acts as a virtual network cable connecting your computer directly to the server.
  1. Download ZeroTier: Go to https://www.zerotier.com/download/ and install the version for your operating system.
  2. Start the Application:
     * Windows: Search for "ZeroTier One" in the Start menu and run it. It will appear in your system tray (bottom right).
     * Mac: It will appear in your top menu bar.
  3. Join the Network:
     * Right-click the ZeroTier icon in your system tray/menu bar.
     * Select Join New Network.
     * Enter this exact Network ID: `[NETWORK_ID]`
     * Check the boxes for "Allow Managed IPs" and "Allow Global IP" if prompted, then click Join.
  5. Send your Node Address for Authorisation:
     * Right-click the ZeroTier icon again and look at the top of the menu. You will see a 10-character alphanumeric string (e.g. My Address: 8a9b2c3d4e).
     * **STOP HERE**. Email or message this Node ID to the server administrator.
     * You cannot access the server until the administrator manually authorises your node on the network.
# Phase 2: Connect to the Remote Computer via SSH
  Once the administrator confirms you are authorised on the ZeroTier network, you can log into the remote computer.
  1. Open your computer's terminal:
     * Windows: Open PowerShell or Command Prompt.
     * Mac/Linux: Open the Terminal application.
  2. Run the following SSH command. Note the `-p 2201` is strictly required as the remote computer listens on port 2201, not the default port 22.

    ssh root@[HOST_IP] -p 2201

  3. When prompted about the authenticity of the host, type yes and press Enter.
  4. Enter the password when prompted: `[HOST_PASSWORD]` (Note: the terminal will not show any characters while you type the password).
You are now inside the Linux environment with full access to the GPU and the Python tools. The shared data directory is located at `/data`. 
# Phase 3: OPTIONAL -- Connect using Visual Studio Code
  Using a raw terminal to write Python code and view generated plots is a terrible workflow. It is highly recommended to use Visual Studio Code to attach directly to the remote computer.
  1. Download and install Visual Studio Code.
  2. Open VS Code, go to the Extensions tab (the squares icon on the left), and search for Remote - SSH (published by Microsoft). Install it.
  3. Press F1 to open the command palette and type `Remote-SSH: Connect to Host...`
  4. Select **Add New SSH Host...**
  5. Enter the exact same command you used in Phase 2: `ssh root@[HOST_IP] -p 2201`
  6. Select the default configuration file to save it.
  7. Click **Connect** in the bottom right popup, or press `F1` again, select `Remote-SSH: Connect to Host...`, and click the IP address you just saved.
  8. Enter the password when prompted.
  VS Code will now install a tiny server inside the remote computer. Once connected, click Open Folder in the explorer pane and type `/data`. You can now edit the Python scripts exactly as if they were on your local machine, and run the terminal directly inside VS Code, 
  
# Phase 4: Verify CPU and GPU Compute
  To ensure the environment is fully configured and the hardware is responding, create a test script to benchmark the system.
  1. Once connected (preferably via VS Code), create a new file in the `/data` folder called test_compute.py.
  2. Paste the following Python code into the file and save it:
```
import torch

print("--- Hardware Diagnostics ---")
print(f"PyTorch Version: {torch.__version__}")

if torch.cuda.is_available():
    print(f"GPU Detected: {torch.cuda.get_device_name(0)}")
else:
    print("WARNING: No GPU detected. Computations will run on CPU only.")

# 1. CPU Compute Test
print("\nRunning basic tensor operation on CPU...")
cpu_tensor = torch.randn(100, 100)
cpu_result = torch.matmul(cpu_tensor, cpu_tensor)
print(">> CPU Test: SUCCESS.")

# 2. GPU Compute Test
if torch.cuda.is_available():
    print("\nRunning basic tensor operation on GPU...")
    # Move tensors to the GPU memory
    gpu_tensor = torch.randn(100, 100).cuda()
    gpu_result = torch.matmul(gpu_tensor, gpu_tensor)
    print(">> GPU Test: SUCCESS.")
else:
    print("\nSkipping GPU test.")

print("\nEnvironment is fully configured and ready.")
```
  3. Open the terminal and run the script:

    python3 test_compute.py
    
  You should see both tests report SUCCESS, confirming the CUDA drivers and PyTorch environment are fully operational.
