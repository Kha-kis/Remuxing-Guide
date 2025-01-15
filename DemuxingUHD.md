# How to Demux a UHD Blu-ray Disk

## Overview

Demuxing a UHD Blu-ray disc extracts individual streams (video, audio, subtitles, chapters, etc.) from the disc structure. Given the complexity of UHD Blu-rays and their frequent use of HEVC (H.265) video, specialized tools are essential. While eac3to is a robust tool for standard Blu-rays, DGDemux is better suited for UHD Blu-rays due to its precise handling of Dolby Vision metadata and seamless branching.

---

## Why DGDemux?

DGDemux is optimized for UHD Blu-rays, especially those containing Dolby Vision. Compared to other tools, it offers:

- Accurate extraction of Base Layer (BL) and Enhancement Layer (EL).
- Seamless integration with dovi_tool for handling Dolby Vision metadata.
- Proper processing of seamless branching playlists.
- Superior error detection for flawed discs.

---

## Setting Up DGDemux

### Installation

1. **Download DGDemux**:

   - Visit the [DGDemux website](https://rationalqm.us/dgdemux/dgdemux.html) to download the latest version.

2. **Install dovi_tool**:

   - Download dovi_tool from its [GitHub repository](https://github.com/quietvoid/dovi_tool).
   - Extract and ensure the executable is in your system path.

### Adding to System Path

- Add the DGDemux directory to your system path to make it accessible from any command prompt location.
- Search online for instructions on how to add directories to the system path for your operating system.

---

## Demuxing UHD Blu-rays

### Step 1: Identify the Playlist

UHD Blu-rays use playlists (.MPLS files) to organize their streams. To identify the correct playlist:

1. **Analyze the Disc Structure**:
   - Use a tool like [BDInfo](https://www.videohelp.com/software/BDInfo) to scan the disc and identify playlists.
2. **Find the Main Feature**:
   - Look for the playlist with the longest runtime and largest size. These typically represent the main feature.
   - Check for seamless branching, where multiple `.M2TS` files are combined into a single playlist.

### Step 2: Demux Using DGDemux

Run DGDemux to extract streams from the identified playlist:

1. **Extract All Streams**:

   ```
   DGDemux.exe "<BDMVFolderPath>" --mpls <PlaylistNumber> --output "<OutputFolderPath>"
   ```

   Example:

   ```
   DGDemux.exe "D:\discs\MovieTitle\BDMV" --mpls 1 --output "D:\output\MovieTitle"
   ```

2. **Demux Specific Streams**:
   To extract only the video and audio tracks:

   ```
   DGDemux.exe "<BDMVFolderPath>" --mpls <PlaylistNumber> --output "<OutputFolderPath>" --video --audio
   ```

---

## Verifying Dolby Vision UHD Blu-ray Mastering Using DGDemux

The purpose of this section is to ensure that the UHD Blu-ray disc is mastered correctly. A properly mastered disc will have matching RPU and scene-cut data when the Metadata Enhancement Layer (MEL) is merged with the Base Layer (BL) compared to the untouched EL.

### Step 1: Extract the Enhancement Layer (EL) RPU

Use DGDemux to extract the Enhancement Layer (EL) and generate the RPU:

```
dovi_tool extract-rpu --input EL.hevc --output RPU_EL.bin
```

### Step 2: Export Scene-Cut Data from EL

Generate the scene-cut data from the RPU extracted from the EL:

```
dovi_tool export -i RPU_EL.bin -d scenes
```

### Step 3: Compare RPU and Scene-Cut Data After BL and EL Merge

Merge the BL and EL to verify mastering consistency:

- Extract the RPU from the merged BL and EL:

```
dovi_tool extract-rpu --input BL_MERGED_WITH_EL.hevc --output RPU_MERGED.bin
```

- Generate scene-cut data from the merged BL and EL RPU:

```
dovi_tool export -i RPU_MERGED.bin -d scenes
```

- Compare the original EL scene-cut data and RPU with the merged BL and EL:

```
diff scene-cuts_RPU_EL.txt scene-cuts_RPU_MERGED.txt
diff RPU_EL.bin RPU_MERGED.bin
```

If the scene-cut data and RPU files match, the disc is mastered correctly, and the RPU can be injected into the Base Layer (BL) during the remuxing process.

If discrepancies are detected:

- Revisit the extraction or merging process to ensure no errors occurred.
- If issues persist, the disc may be poorly mastered. Consider obtaining a corrected release from another region or a replacement disc.

With MEL, converting the RPU from the EL to Profile 8 can resolve some issues caused by incorrect Profile 7 RPUs. Use the following command to create the Profile 8 RPU:

```
dovi_tool -m 2 extract-rpu EL.hevc -o RPU_EL_P8.bin
```

After creating the Profile 8 RPU, you can plot the RPU to verify its consistency and ensure correctness. Use the following command to plot the RPU L1 metadata into a graph:

```
dovi_tool plot RPU_EL_P8.bin -t "Dolby Vision L1 plot (Profile 8)" -o L1_plot.png
```

We will want to compare this plot to the plot of the Profile 7 RPU to ensure consistency and detect any discrepancies. To plot the Profile 7 RPU, use the following command:

```
dovi_tool plot RPU_EL.bin -t "Dolby Vision L1 plot (Profile 7)" -o L1_plot_P7.png
```

The output will be PNG images that visually represent the RPU's L1 metadata. This allows you to confirm the number of frames plotted aligns with the total number of frames in the movie. Visualization tools or image viewers can be used to inspect the plot.

Ensure to document the results of the comparison for future reference or troubleshooting. Proper verification ensures that any REMUX made from the disc will maintain Dolby Vision compatibility without introducing artifacts or inconsistencies.

### Step 4: Inject the RPU into the Base Layer

Once the RPU has been validated and any corrections have been made (if necessary), it must be injected into the Base Layer (BL) to finalize the remux.

Use the following command to inject the RPU into the BL:

For the original RPU:

```
dovi_tool inject-rpu -i BL.hevc --rpu-in RPU_EL.bin -o BL_with_RPU.hevc
```

For the corrected Profile 8 RPU:

```
dovi_tool inject-rpu -i BL.hevc --rpu-in RPU_EL_P8.bin -o BL_with_RPU.hevc
```

This step ensures the Dolby Vision metadata is properly embedded into the video stream, maintaining compatibility with Dolby Vision-enabled playback devices.

---

## Special Considerations for UHD Discs

### Seamless Branching

UHD Blu-rays often use seamless branching. Ensure the selected playlist references all parts of the main feature. DGDemux handles this automatically.

### Forced Subtitles

If forced subtitles are present, extract them using [BDSup2Sub](https://www.videohelp.com/software/BDSup2Sub):

1. Import the subtitle file.
2. Set the desired FPS.
3. Export the forced subtitles.

---

## Conclusion

DGDemux is the go-to tool for demuxing UHD Blu-ray discs, offering unmatched accuracy and reliability for Dolby Vision content and seamless branching. By following this guide, you can confidently extract high-quality streams ready for further processing or remuxing. For advanced Dolby Vision handling, integrate dovi_tool into your workflow to ensure flawless metadata management.