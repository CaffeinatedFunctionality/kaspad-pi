
# Kaspa Node on Raspberry Pi

This guide provides steps to set up a Kaspa node on a Raspberry Pi using Rust or you could just use the image created that already has this done for you. 
For 1 BPS an SD Card will work just fine, however you will want a Solid State Drive(SSD) when we move to 10 BPS 

## Prerequisites

- Raspberry Pi 3 or 4
- Raspberry Pi OS (latest version)
- Internet connection

## Step 1: Set Up Your Raspberry Pi

1. **Install Raspberry Pi OS**
   Download and install the latest version of Raspberry Pi OS from the [Raspberry Pi website](https://www.raspberrypi.org/software/operating-systems/).

2. **Update Your System**
   ```bash
   sudo apt-get update
   sudo apt-get upgrade
   ```

## Step 2: Install Rust

1. **Install Rust**
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```
   Follow the on-screen instructions to complete the installation.

2. **Configure Rust**
   ```bash
   source $HOME/.cargo/env
   ```

## Step 3: Install Dependencies

1. Install general prerequisites

    ```bash
    sudo apt install curl git build-essential libssl-dev pkg-config 
    ```

2. Install Protobuf (required for gRPC)
  
    ```bash
    sudo apt install protobuf-compiler libprotobuf-dev #Required for gRPC
    ```
3. Install the clang toolchain (required for RocksDB and WASM secp256k1 builds)

    ```bash
    sudo apt-get install clang-format clang-tidy \
    clang-tools clang clangd libc++-dev \
    libc++1 libc++abi-dev libc++abi1 \
    libclang-dev libclang1 liblldb-dev \
    libllvm-ocaml-dev libomp-dev libomp5 \
    lld lldb llvm-dev llvm-runtime \
    llvm python3-clang
    ```
4. Install wasm-pack
    ```bash
    cargo install wasm-pack
    ```
5. Install wasm32 target
    ```bash
    rustup target add wasm32-unknown-unknown
    ```

6. Install pm2
    ```bash
    sudo apt-get update
    sudo apt-get install -y nodejs npm
    sudo npm install pm2 -g
    ```

## Step 4: Clone the Kaspa Node Repository

1. **Clone the Repository**
   ```bash
   git clone https://github.com/kaspanet/rusty-kaspa.git
   ```

## Step 5: Create a startup file

1. **Create the script that starts kaspad**
   ```bash
   nano ~/startup_script.sh
   ```

2. **Add the following content to the script**
  ```bash
  #!/bin/bash
  cd ~/rusty-kaspa
  cargo run --release --bin kaspad
  ```

3. **Make the script executable**
  ```bash
  chmod +x ~/startup_script.sh
  ```

## Step 6: Configure pm2

1. **Start the script with pm2**
   ```bash
   pm2 start ~/startup_script.sh --name kaspad
   ```

2. **Setup pm2 to run on startup**
   ```bash
   pm2 startup
   ```
   The command will output something like the following
   ```bash
   [PM2] Init System found: systemd
   [PM2] To setup the Startup Script, copy/paste the following command:
   sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u pi --hp /home/pi
   ```

3. **Copy and run the command provided by pm2**

4. **Save the pm2 process list**
   ```bash
   pm2 save
   ```

5. **Reboot the raspberry pi**
   ```bash
   sudo reboot
   ```

## Step 7: Monitor and Maintain

1. **Check the pm2 process**
   ```bash
   pm2 status
   ```
   You should see the 'kaspad' process running

2. **View the logs**
   ```bash
   pm2 logs kaspad
   ```
   This should show you that it's running properly and you should see

## Optional: Create an update script(unnecessary, however ideal so you don't miss a step with pm2) 

1. **Create the script that updates kaspad**
   ```bash
   nano ~/update_kaspad.sh
   ```

2. **Add the following content to the script**
  ```bash
  #!/bin/bash
  pm2 stop kaspad
  cd ~/rusty-kaspa
  git pull
  pm2 start kaspad
  ```

3. **Make the script executable**
  ```bash
  chmod +x ~/update_kaspad.sh
  ```

3. **Test the script**
  ```bash
  cd ~/
  ./update_kaspad.sh
  ```
  It should show pm2 stopping, the repo updating, and then the pm2 starting again

## Additional Tips

- **Optimize Performance**: Monitor your Raspberry Pi’s performance, get a case with a fan, preferrably one that will allow you to use a SSD in the future as that is what will be needed for 10 blocks per second.
- **Join the Community**: Engage with the Kaspa community for support, updates, and best practices.

By following these steps, you should have a fully functional Kaspa node running on your Raspberry Pi. If you encounter any issues, refer to the project documentation or community forums for assistance.
