# Install Snort 2 on Windows 11 (with Npcap)

This guide documents how to install Snort on Windows 11 using the last Windows installer available (Snort 2.9.20) and Npcap 1.83. It also covers adding Snort to your system PATH so you can run it from any terminal, and verifying the installation.

Notes:
- Snort 3 currently does not provide an official Windows installer. This guide therefore uses Snort 2.9.20 for Windows.
- All steps below were performed on Windows 11 with Administrator rights.

Contents:
- Prerequisites
- 1) Download the Snort 2.9.20 Windows installer
- 2) Install Snort 2.9.20
- 3) Install Npcap 1.83
- 4) Add Snort to the PATH environment variable
- 5) Verify the installation
- Troubleshooting

Screenshots:
Each screenshot referenced below should include a visible red rectangle or highlight over the exact UI element being discussed (links, buttons, checkboxes). Save the images in `assets/snort-windows/images/` using the file names shown with each figure.

## Prerequisites

- Windows 11 with Administrator access
- Internet connectivity to download installers
- Willingness to install a kernel packet capture driver (Npcap)

## 1) Download the Snort 2.9.20 Windows installer

1. Visit https://www.snort.org/downloads#additional_downloads
2. Under the “Snort 2” section, locate “Binaries”.
3. Click “Snort_2_9_20_Installer.x64.exe”.

Figure 1 (save as `snort-downloads-binaries.png`):
![Snort downloads page — the “Binaries” link with “Snort_2_9_20_Installer.x64.exe” highlighted](assets/snort-windows/images/snort-downloads-binaries.png)

Why Snort 2? Snort 3 currently has no official Windows installer; as such, the Windows path uses Snort 2.9.20. You can still capture traffic and test rules with Snort 2 on Windows.

## 2) Install Snort 2.9.20

1. Launch `Snort_2_9_20_Installer.x64.exe` as Administrator.
2. Accept the license agreement.
3. Leave default components selected (Snort, Dynamic Modules, Documentation).
4. Choose an install directory. In this guide we use `E:\liamh\Snort`.
5. Complete the wizard.
6. At the end, the installer reminds you that Npcap is required — we’ll install it in the next section.

Figures:
- Figure 2 (save as `snort-setup-license.png`):  
  ![Snort Setup — License Agreement page with “I Agree” highlighted](assets/snort-windows/images/snort-setup-license.png)
- Figure 3 (save as `snort-setup-components.png`):  
  ![Snort Setup — Choose Components page with all components checked/highlighted](assets/snort-windows/images/snort-setup-components.png)
- Figure 4 (save as `snort-setup-destination.png`):  
  ![Snort Setup — Destination Folder page with E:\liamh\Snort highlighted](assets/snort-windows/images/snort-setup-destination.png)
- Figure 5 (save as `snort-setup-complete.png`):  
  ![Snort Setup — Completed page with Close highlighted](assets/snort-windows/images/snort-setup-complete.png)
- Figure 6 (save as `snort-setup-npcap-required.png`):  
  ![Snort Setup — Message indicating Npcap is required highlighted](assets/snort-windows/images/snort-setup-npcap-required.png)

## 3) Install Npcap 1.83

1. Go to https://npcap.com/#download
2. Download “Npcap 1.83 installer”.
3. Run the installer as Administrator.
4. Recommended options:
   - Leave “Install Npcap in WinPcap API-compatible Mode” checked (helps older tools).
   - Optionally enable “Restrict Npcap driver’s access to Administrators only” for tighter security in lab/production.
   - Leave others at defaults unless you specifically need 802.11 raw capture.
5. Complete the install.

Figures:
- Figure 7 (save as `npcap-download-page.png`):  
  ![Npcap download page — “Npcap 1.83 installer” link highlighted](assets/snort-windows/images/npcap-download-page.png)
- Figure 8 (save as `npcap-setup-license.png`):  
  ![Npcap Setup — License Agreement with “I Agree” highlighted](assets/snort-windows/images/npcap-setup-license.png)
- Figure 9 (save as `npcap-setup-options.png`):  
  ![Npcap Setup — Options with “Install Npcap in WinPcap API-compatible Mode” highlighted](assets/snort-windows/images/npcap-setup-options.png)
- Figure 10 (save as `npcap-setup-finished.png`):  
  ![Npcap Setup — Finished page with Finish highlighted](assets/snort-windows/images/npcap-setup-finished.png)

## 4) Add Snort to the PATH environment variable

Adding Snort’s `bin` directory to PATH lets you run `snort` from any terminal.

Target path used in this guide:
- `E:\liamh\Snort\bin`

Steps:

1. Open the Start menu, search “View advanced system settings”, and open it.
2. In the System Properties window, go to the “Advanced” tab and click “Environment Variables…”.
3. Under “System variables”, select “Path” and click “Edit…”.
4. Click “New” and add `E:\liamh\Snort\bin`.
5. Click OK to close all dialogs.

Figures:
- Figure 11 (save as `system-properties-advanced.png`):  
  ![System Properties — Advanced tab with “Environment Variables…” highlighted](assets/snort-windows/images/system-properties-advanced.png)
- Figure 12 (save as `environment-variables.png`):  
  ![Environment Variables dialog — “Path” (System variables) highlighted](assets/snort-windows/images/environment-variables.png)
- Figure 13 (save as `edit-path-add-snort-bin.png`):  
  ![Edit environment variable — new entry E:\liamh\Snort\bin highlighted](assets/snort-windows/images/edit-path-add-snort-bin.png)

Optional variables (helpful for tooling/scripts):
- `SNORT_HOME` = `E:\liamh\Snort`
- `SNORT_CONF` = `E:\liamh\Snort\etc\snort.conf`

## 5) Verify the installation

Open a new Command Prompt window (PATH changes require a new shell) and run:

```
snort -V
```

Expected output should show the Snort version and linked libraries, for example:

```
--> Snort! <--
Version 2.9.20-WIN64 GRE (Build 82)
By Martin Roesch & The Snort Team: http://www.snort.org/contact#team
Copyright (C) 2014-2022 Cisco and/or its affiliates. All rights reserved.
Copyright (C) 1998-2013 Sourcefire, Inc., et al.
Using PCRE version: 8.10 2010-06-25
Using ZLIB version: 1.2.11
```

Figure 14 (save as `cmd-snort-version.png`):
![Command Prompt — snort -V output highlighted](assets/snort-windows/images/cmd-snort-version.png)

If the command is not found, confirm the PATH entry points to `E:\liamh\Snort\bin` and that you opened a fresh terminal after editing PATH.

## Troubleshooting

- Open a new terminal after editing PATH. Old windows won’t have the updated environment.
- Run Command Prompt as Administrator for operations that require elevated permissions.
- Ensure Npcap service is installed and running:
  - Services name is typically “Npcap Packet Driver (NPCAP)”.
- If Snort can’t capture traffic:
  - Verify Npcap was installed with WinPcap API-compatible mode enabled.
  - Reinstall Npcap 1.83 and reboot.
- If you installed Snort to a different drive/folder, update PATH accordingly.
- Confirm the `bin` directory actually contains `snort.exe` (e.g., `E:\liamh\Snort\bin\snort.exe`).

## File/Folder Summary

- Snort install dir (example): `E:\liamh\Snort\`
- Snort binaries: `E:\liamh\Snort\bin\`
- Images for this guide (create this folder in your repo):  
  `assets/snort-windows/images/`
  - `snort-downloads-binaries.png`
  - `snort-setup-license.png`
  - `snort-setup-components.png`
  - `snort-setup-destination.png`
  - `snort-setup-complete.png`
  - `snort-setup-npcap-required.png`
  - `npcap-download-page.png`
  - `npcap-setup-license.png`
  - `npcap-setup-options.png`
  - `npcap-setup-finished.png`
  - `system-properties-advanced.png`
  - `environment-variables.png`
  - `edit-path-add-snort-bin.png`
  - `cmd-snort-version.png`

Ensure each screenshot includes a red rectangle or highlight around the UI element referenced in its caption before committing to GitHub.

— End —
