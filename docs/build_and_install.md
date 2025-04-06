## Clone repository
This project uses [TextTCP](https://github.com/IsmailBiswas/TextTCP_CTS.git) as submodules so when
cloning this repository, use the `--recurse-submodules` flag:

```sh
git clone --recurse-submodules https://github.com/IsmailBiswas/CTSync.git
```
Otherwise, the server build will fail.

# Build Server

### Using Docker
Make sure both `Docker` and `Docker Compose` are installed. Then, navigate to the `/backend/compose` directory and build the Docker image by running:

```bash
docker compose build
```

This will create an image named `ctsync_server`.

Next, generate the required server files by executing the `gen_server_files.sh` script:

```bash
./gen_server_files.sh
```

This script will create a `server_files` directory containing:

- A private key  
- A self-signed certificate  
- An example server join invite key file  

Once generated, start the server using:

```bash
docker run -d --name ctsync_server --restart=always -v ./server_files:/app/server_files -p 4343:4343 ctsync_server:latest
```

To use a different port, set the `CTSYNC_PORT` environment variable. Make sure the exposed port matches:

```sh
docker run -d --name ctsync_server --restart=always -v ./server_files:/app/server_files -e CTSYNC_PORT=5757 -p 5757:5757 ctsync_server:latest
```

When you first time run the server it will automatically create an SQLite database inside the `server_files` directory.

**Use Prebuilt Image**  

If you don't want to build the image locally, you can pull the latest image from the GitHub Container Registry:  

```sh
docker pull ghcr.io/ismailbiswas/ctsync_server:latest
```  

After that, generate the `server_files` and start the container as described earlier.  


### On Host

The server is a single-threaded C program that uses CMake as its build system.

**Build Requirements:**  
=== "Debian"
    ``` bash
    sudo apt update
    sudo apt install build-essential \
    cmake \
    libssl-dev \
    uuid-dev \
    libsqlite3-dev
    libcjson-dev
    ```

=== "Arch"
    ``` bash
    sudo pacman -Syu
    sudo pacman -S --needed \
    base-devel \
    cmake \
    openssl \
    sqlite \
    util-linux-libs
    cjson
    ```

**Building the Server**  
After installing dependencies, navigate to the `/backend` directory and run:

```bash
mkdir -p build && cd build
cmake .. && make
```
This will generate the server binary inside the `build` directory.

### Server Required Files (Run Server)
The server needs **three files** to work properly. All three files must be present inside a directory named `server_files` at the current working directory. 

1. **Server Join Key File:**  
   This file must be named `invite_keys.txt`. Each line in this file should contain a alphanumeric string, followed by a space and an ISO 8601 format timestamp indicating the key's expiration time.  
   &nbsp;  
   When a new device wants to join the server, it must provide one of the keys specified in this file.  
   &nbsp;  
   Example invite_keys.txt file:
   ```
   invitekey123 2025-09-07T01:32:00Z #test invite key 1
   invitekey456 2025-07-07T01:32:00Z #test invite key 2
   ```
 

1. **Private Key:** This is the TLS private key file in PEM format. It must named `pkey.pem`
1. **Self Signed Certificate:** The self signed certificate file in PEM format. It must be named `self_signed_cert.pem`  

You can create all three of these files inside a `server_files` directory by running the `/backend/compose/gen_server_files.sh` script. 

After that execute the server binary, running the binary will automatically create a SQLite DB in the working directory.


# Build Client

The client is a [Tauri](https://tauri.app/) application developed with [Vue.js](https://vuejs.org/) and the [shadcn-vue](https://www.shadcn-vue.com/) component library. Thanks to Tauri’s cross-platform framework, the client can be compiled for Linux, Windows, macOS, iOS, and Android.

### Prerequisites

Being a Tauri application, it requires Tauri’s specific [prerequisites](https://v2.tauri.app/start/prerequisites/) to be installed. While I've outlined them below, for the most accurate and current details, please refer to the [official Tauri documentation](https://v2.tauri.app/start/prerequisites/).


- [Install Rust](https://www.rust-lang.org/tools/install).  
- [Install Node.js](https://nodejs.org/en/download). 

Then install these system prerequistes.

=== "Debian"

    ``` bash
    sudo apt update
    sudo apt install libwebkit2gtk-4.1-dev \
    build-essential \
    curl \
    wget \
    file \
    libxdo-dev \
    libssl-dev \
    libayatana-appindicator3-dev \
    librsvg2-dev
    ```

=== "Arch"

    ``` bash
    sudo pacman -Syu
    sudo pacman -S --needed \
    webkit2gtk-4.1 \
    base-devel \
    curl \
    wget \
    file \
    openssl \
    appmenu-gtk-module \
    libappindicator-gtk3 \
    librsvg
    ```

=== "Windows"

    1. Download the [Microsoft C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) installer and open it to begin installation. During installation check the “Desktop development with C++” option.
    - [Install WebView2](https://v2.tauri.app/start/prerequisites/#webview2). (not required on Window10 and onward)


### Run Development Version
First, navigate to the **/frontend** directory and install npm packages.
```sh
npm install
```
Then to start the development version run:

```sh
npm run tauri dev
```

### Build Production Version
To build production version of the application run:

```sh
npm run tauri build
  
```

- On Linux, this generates .deb, .rpm, and an AppImage.
- On Windows, it creates .msi and .exe installers.


???+ tip "Arch build problem!"
    If you encounter a binary stripping problem when building the AppImage, a temporary fix is to set the `NO_STRIP=true` environment variable. This is an upstream problem.

## Build Client APK On Linux

Ensure that the Android SDK and NDK are installed, and set the following environment variables:  

```sh
export ANDROID_HOME=/path/to/android/sdk  
export NDK_HOME=/path/to/android/ndk
```
The simplest way to get the SDK and NDK is to install [Android Studio](https://developer.android.com/studio) and use it to manage both.

After that, go to the `/frontend` directory and initialize an Android project by running:

```sh
npm run tauri android init
```

If you want, you can configure the icons to be used by running:

```sh
npm run tauri icon /path/to/icon.png

```

#### Android code signing
Next, you need to set up Android code signing; otherwise, you won't be able to
install the release version of the APK on a physical device.
The process involves first creating a `Keystore` file and then configuring
`gradle` to automatically use that `Keystore` file to sign the code whenever
an Android build is created. Please visit [this page↗️](https://tauri.app/distribute/sign/android/#configure-the-signing-key)
in the official Tauri documentation to set up Android code signing.  


#### Build and install
After setting up Android code signing, you can simply run:  

```sh
npm run tauri android build  
```
to build a signed APK.

If everything goes accordingly, at the end of the build, the APK file path will be shown. You can then use
```sh
adb install /path/to/apk/file
```
 to install it on a device.
