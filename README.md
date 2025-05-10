# Horizon XI Linux Setup Notes

These notes cover optional steps to improve graphics and visual effects for Horizon XI on Linux, specifically focusing on using **dgVoodoo2** and **ReShade**.

**Important:** This guide assumes you have already installed Horizon XI on Linux using the method described at [https://github.com/MattyGWS/HorizonXI-Linux-Installation](https://github.com/MattyGWS/HorizonXI-Linux-Installation) and are running the game via Steam.

## Prerequisites

To use the tools mentioned in this document, you will need:

* **Wine:** Required if you want to use the configuration tool (`dgVoodooCpl.exe`) for dgVoodoo2 and if you want to use ReShade.
* **Winetricks:** Required for ReShade to install necessary components.

*Note: This document does not cover how to install `wine` or `winetricks` on your specific Linux distribution or Steam Deck.*

## dgVoodoo2: Improve Graphics Performance

**What it does:** dgVoodoo2 helps older games like Final Fantasy XI work better with modern graphics cards by translating their old graphics commands into ones modern GPUs understand. This usually results in smoother performance and higher frame rates.

**How to set it up:**

1. **Download dgVoodoo2:** Get the latest version from the official website: [https://dege.freeweb.hu/dgVoodoo2/dgVoodoo2](https://dege.freeweb.hu/dgVoodoo2/dgVoodoo2)
2. **Extract the files:** From the downloaded archive, you'll need these three files:
    * `dgVoodoo.conf` (the configuration file)
    * `D3D8.dll` (a graphics library file)
    * `dgVoodooCpl.exe` (the configuration tool)
3. **Copy files to the game folder:** Copy these three files into the `bootloader` directory of your Horizon XI installation.
    * Find your game installation directory (let's call it `$YOUR_INSTALL_DIR`).
    * The target directory is `$YOUR_INSTALL_DIR/HorizonXI/Game/bootloader`.
    * *Example path:* If you installed in your home folder, this might look like `/home/yourusername/HorizonXI/Game/bootloader`.
4. **Copy the provided configuration:** Get the `dgVoodoo.conf` file from this repo and copy it over the `dgVoodoo.conf` you just extracted. This file is pre-configured for a single graphics card setup running FFXI in a window.
5. **Configure Steam Launch Options:** Open Steam, right-click on your Horizon XI game in the Library, select "Properties", and then go to the "Shortcut" section. Add the following to the "Launch Options" box:

    ```sh
    WINEDLLOVERRIDES="d3d8=n,b" %command%
    ```

    *What this does:* This tells Wine (the compatibility layer Steam uses) to load the `D3D8.dll` from the game's directory (`n`) first, and if that doesn't work, fall back to its built-in version (`b`). This ensures dgVoodoo2 is used.

6. **Verify it's working:** Launch Horizon XI from Steam. You should see a "dgVoodoo" logo in the bottom-right corner of the screen very early on (around the "Rules of Conduct" screen). If you see the logo, dgVoodoo2 is active!
7. **Hide the logo** To remove the logo, open the `dgVoodoo.conf` file in a text editor and change the line `dgVoodooWatermark = true` to `dgVoodooWatermark = false`.

If you *do not* see the logo, then you'll likely need to configure your video card using the configuration tool in the next section.

### Running the dgVoodoo Configuration Tool (`dgVoodooCpl.exe`)

If you need to change settings for dgVoodoo2 using its graphical tool (`dgVoodooCpl.exe`), follow these steps:

1. **Make sure Wine is installed.**
2. **Open a terminal** and navigate to the `bootloader` directory where you copied the dgVoodoo files (`$YOUR_INSTALL_DIR/HorizonXI/Game/bootloader`).
3. **Run the tool using Wine:** Execute the following command:

    ```sh
    wine dgVoodooCpl.exe
    ```

4. Make any desired changes in the window that appears and click "OK".

#### Troubleshooting: Video card not showing up

If your graphics card doesn't appear in the dropdown menu within `dgVoodooCpl.exe`, it likely means you're running the tool with a different Wine setup than the one Steam uses for the game, and the necessary components aren't available there.

To fix this, run `dgVoodooCpl.exe` using the *exact same Wine environment* that Steam uses for Horizon XI. You'll need the unique number Steam assigned to the game's compatibility data ("compatdata").

1. **Find the game's compatdata number:** Open a terminal and run this command:

    ```sh
    find ~/.steam/steam/steamapps/compatdata -type d -name HorizonXI-Launcher | grep -oP '/compatdata/\K\d+(?=/pfx/)'
    ```

    This command searches your Steam installation for the Horizon XI launcher's compatibility folder and prints the number associated with it. Copy this number.
2. **Run the tool with the correct prefix:** Replace `HORIZON_NUMBER_HERE` with the number from the above command, and run it from the `bootloader` directory:

    ```sh
    WINEPREFIX=$HOME/.steam/steam/steamapps/compatdata/HORIZON_NUMBER_HERE/pfx wine dgVoodooCpl.exe
    ```

    *What this does:* `WINEPREFIX` tells Wine to use a specific directory (`$HOME/.steam/steam/steamapps/compatdata/HORIZON_NUMBER_HERE/pfx`) as its environment, which is where Steam keeps the game's Wine setup.

## ReShade: Add Post-Processing Visual Effects

**What it does:** ReShade is a tool that applies visual filters and effects (like improved colors, depth of field, ambient occlusion) to games as they are drawn to your screen. This allows you to customize the game's look significantly.

*Note: Using many effects in ReShade can impact performance, especially on less powerful hardware like the Steam Deck. Setup is also much easier with a mouse and keyboard.*

Follow these steps to install and configure ReShade:

### 1. Install ReShade

1. **Download ReShade:** Go to the ReShade website ([https://reshade.me/](https://reshade.me/)) and download the version **with full add-on support**. The download link is usually near the bottom of the page.
2. **Copy the installer:** Copy the downloaded `.exe` file (e.g., `ReShade_Setup_VERSION_Addon.exe`) into your game's `bootloader` directory (`$YOUR_INSTALL_DIR/HorizonXI/Game/bootloader`).
3. **Open a terminal** and navigate to the `bootloader` directory.
4. **Run the installer using Wine:** Execute the following command, replacing `VERSION` with the version number you downloaded:

    ```sh
    wine ReShade_Setup_VERSION_Addon.exe
    ```

5. **Select the game:** In the ReShade installer window, click the "Browse" button.
6. **Navigate to the game executable:** Use the file browser within the installer to go to: `My Computer` -> `Z:` -> `$YOUR_INSTALL_DIR/HorizonXI/Game/bootloader`. Select the file `horizon-loader.exe` and click "Open".
    *What is Z:?* In Wine's virtual file system, `Z:` usually maps to your computer's root directory (`/`), allowing you to access files outside the virtual Wine drive.
7. **Choose the rendering API:** Select `Microsoft DirectX 10/11/12`.
8. **Select effects:** Choose which shader effect collections you want to download.
    * You can click "Browse..." to load a preset file (often shared by other players) which will automatically select the required effects.
    * Alternatively, you can click "Uncheck all" and then "Check all" to download *all* available effects (this downloads more data but gives you everything to experiment with).
9. **Download effects:** Click "Next" and wait for the shaders to download.
10. **Optional Addons:** If prompted, you can optionally select additional addons (none are typically needed for basic FFXI setup). Click "Next" and then "Finish".

*ReShade is installed, but might not work yet. You need the next step.*

### 2. Install Dependencies (Shader Compiler)

ReShade needs a special component called a "shader compiler" to make the visual effects work. Steam Deck and some Linux setups have one, but it might not be the right version or available for all shaders.

Use `winetricks` to install the necessary compiler (`d3dcompiler_47`). You'll need the game's compatdata number again (see the troubleshooting section for dgVoodoo2 if you don't have it).

Open a terminal and run the following command, replacing `HORIZON_NUMBER_HERE` with your game's compatdata number:

```sh
WINEPREFIX=$HOME/.steam/steam/steamapps/compatdata/HORIZON_NUMBER_HERE/pfx winetricks d3dcompiler_47
```

*What this does:* This command runs `winetricks` using the game's specific Wine environment (`WINEPREFIX`) and tells it to install the `d3dcompiler_47` component.

### 3. Configure Launcher for ReShade and dgVoodoo2

Finally, you need to tell Steam's Wine environment to load the necessary libraries for both dgVoodoo2 and ReShade.

Go back to the game's "Launch Options" in Steam (`Right click game in Library -> Properties -> Shortcut`) and change the command to this:

```sh
WINEDLLOVERRIDES="d3d8=n,b;dxgi=n,b;d3dcompiler_47=n,b" %command%
```

*What's changed:*

* `d3d8=n,b`: Still needed for dgVoodoo2.
* `;`: Separates multiple overrides.
* `dxgi=n,b`: Needed for ReShade to function correctly with DirectX 10/11/12 games.
* `d3dcompiler_47=n,b`: Ensures the shader compiler you installed is used.

Now, when you launch the game via Steam, both dgVoodoo2 and ReShade should be active! ReShade usually shows a small banner at the top of the screen when the game starts, indicating you can press the `Home` key (or `Fn + Left Arrow` on Steam Deck) to open its configuration overlay.
