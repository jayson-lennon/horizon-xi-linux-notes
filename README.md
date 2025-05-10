# Horizon XI Linux Notes

## Prereqs

- This has only been tested using Steam with the same configuration from the installation guide at [https://github.com/MattyGWS/HorizonXI-Linux-Installation](https://github.com/MattyGWS/HorizonXI-Linux-Installation)
- If you want to edit the `dgVoodoo2` config file using `dgVoodooCpl.exe`, then you'll need `wine` installed
- If you want to use [ReShade](https://reshade.me/), then you'll need `winetricks` installed

Installing `wine` and `winetricks` on Steamdeck/other systems is outside the scope of this document.

## dgVoodoo2

[dgVoodoo2](https://dege.freeweb.hu/dgVoodoo2/dgVoodoo2) wraps older graphics API calls into newer ones which lets old games (like FFXI) better utilize modern GPUs. You should get consistently higher FPS when using this.

Note that this has only been tested when running the game via Steam. It _should_ still work without Steam provided you set the correct environment variables.

`dgVoodoo2` comes with three files:

1. `dgVoodoo.conf`
2. `D3D8.dll`
3. `dgVoodooCpl.exe`

Copy these files to `$YOUR_INSTALL_DIR/HorizonXI/Game/bootloader` where `$YOUR_INSTALL_DIR` is wherever you chose to install the game.

Add this to your Steam "Launch Options" via `Right click game in Library -> Properties -> Shortcut`:
    `WINEDLLOVERRIDES="d3d8=n,b" %command%`

Copy the `dgVoodoo.conf` from this repo and overwrite the one from the `dgVoodoo2` package. The conf from this repo is configured for a single-GPU system running FFXI in windowed mode.

Run the HorizonXI launcher from Steam. Once you start the game, you should immediately see a "dgVoodoo" logo at the bottom-right corner of the screen (as early as the Accept/Decline "Rules of Conduct" disclaimer). If you _do_ see the logo, then that means it's running correctly. You can remove the logo by changing this line from `true` to `false` in the `dgVoodoo.conf` file:

```conf
dgVoodooWatermark = false
```

If you _do not_ see the logo (or if you want to change the `dgVoodoo` settings), then you will need to use the included control panel application. Getting this to work is detailed in the next section.

### Running `dgVoodooCpl.exe`

If you want/need to customize `dgVoodoo2` with the provided control panel application, then you'll need to take additional steps:

1. Make sure `wine` is installed
2. Navigate to `$YOUR_INSTALL_DIR/HorizonXI/Game/bootloader`
3. Run `wine dgVoodooCpl.exe`
4. Make any necessary changes
5. Click OK

#### Troubleshooting

**My video card doesn't show up in the `dgVoodooCpl` dropdown**

This likely means your default wine prefix is missing certain packages. I don't know the exact packages required to fix this, _but_ if you are using Steam then you can just use the prefix it provides (make sure you are in the `bootloader` directory):

```sh
WINEPREFIX=$HOME/.steam/steam/steamapps/compatdata/HORIZON_NUMBER_HERE/pfx wine dgVoodooCpl.exe
```

The `HORIZON_NUMBER_HERE` can be found by running:

```sh
find ~/.steam/steam/steamapps/compatdata -type d -name HorizonXI-Launcher | grep -oP '/compatdata/\K\d+(?=/pfx/)'
```

## ReShade

[ReShade](https://reshade.me/) adds post-processing shaders to games in order to provide them with a customized look.

Check out [https://ariel-logos.github.io/ElfyLab/2025/02/20/reshade-update.html](https://ariel-logos.github.io/ElfyLab/2025/02/20/reshade-update.html) for a great ReShade preset.

I've split this into multiple sections since there are a fair number of steps.

### Install ReShade

First let's get ReShade installed:

1. Download [ReShade](https://reshade.me/) _with full add-on support_. The download link is at the bottom of the ReShade page.
2. Copy the downloaded .exe to the `$YOUR_INSTALL_DIR/HorizonXI/Game/bootloader` directory
3. Navigate to `$YOUR_INSTALL_DIR/HorizonXI/Game/bootloader`
4. Run `wine ReShade_Setup_VERSION_Addon.exe`. Replace VERSION with whichever version you downloaded.
5. After it loads, click `Browse`
6. Navigate to `My Computer` then `Z:` then to `$YOUR_INSTALL_DIR/HorizonXI/Game/bootloader`
7. Select `horizon-loader.exe`
8. Click `Next`
9. Select `Microsoft DirectX 10/11/12`.
10. Select whichever effects you want to install.
    - Clicking `Browse...` will load a preset and only download the effects for that preset.
    - Click `Uncheck all` and then `Check all` if you just want to download everything
11. Click `Next`. Wait for download to complete
12. Optionally select additional addons (none are needed). Click `Next` then `Finish`

NOTE: ReShade will _probably not_ work yet! Continue to the next section.

### Install Dependencies

ReShade requires a shader compiler to work. Steamdeck comes packaged with a compiler in the default wine prefix, but it doesn't work for all the shaders. If you are running on a different distro, then there may not be a compiler at all.

Install this package to get the shaders to compile (you'll need your steamapps [compatdata](#troubleshooting) number again if using Steam):

```sh
WINEPREFIX=$HOME/.steam/steam/steamapps/compatdata/HORIZON_NUMBER_HERE/pfx winetricks d3dcompiler_47
```

After installing, proceed to the next section to configure the launcher.

### Launcher Configuration

Finally we need to set the launcher to use the correct DLLs. On Steam "Launch Options" (`Right click game in Library -> Properties -> Shortcut`), change it to this:

```
WINEDLLOVERRIDES="d3d8=n,b;dxgi=n,b;d3dcompiler_47=n,b" %command%
```

Note that we added `dxgi` and `d3dcompiler_47`. The `d3d8` is for `dgVoodoo2` and must also be present.
