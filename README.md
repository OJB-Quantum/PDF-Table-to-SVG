# PDF-Table-to-SVG
A tool that runs in the browser using Google Colab and a Graphics Processing Unit (GPU) to identify and convert located tables into Scalable Vector Graphics (SVG) outputs. Control knobs are included for easy parameter adjustment. The Colab code is open source, written by Onri Jay Benally in 2026.

---

You can use a free GPU in Colab as needed. 

## Click to use the PDF-Table-to-SVG converter in Google Colab: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/OJB-Quantum/PDF-Table-to-SVG/blob/main/PDF_Table_to_SVG_Converter.ipynb)

---

## Primer on Document Image Analysis and Mathematical Morphology

Preserving the exact visual fidelity of a table from a compiled document requires treating the page as a high-resolution image. This tool utilizes Document Image Analysis to rasterize pages and apply computer vision techniques to identify structural grids. 

To locate the tables, the algorithm relies on mathematical morphology. By passing specific geometric kernels over the image matrix, the algorithm isolates the long horizontal and vertical lines that form table borders. Because analyzing millions of pixels with large kernels is computationally demanding, this mathematical convolution is parallelized on a Graphics Processing Unit using CuPy. 

Finally, to guarantee absolute visual accuracy while providing the requested vector wrapper, the identified table regions are cropped as high-resolution rasters and embedded directly inside an SVG `<image>` container.

## Acronyms and Symbols

| Term  Symbol | Definition |
| :--- | :--- |
| **BGR/ RGB** | Blue-Green-Red/ Red-Green-Blue (Color channel spaces) |
| **CPU** | Central Processing Unit |
| **CUDA** | Compute Unified Device Architecture (Parallel computing platform) |
| **DOM** | Document Object Model |
| **DPI** | Dots Per Inch (Resolution metric) |
| **GPU** | Graphics Processing Unit |
| **HTML** | HyperText Markup Language |
| **PDF** | Portable Document Format |
| **SVG** | Scalable Vector Graphics |
| **URI** | Uniform Resource Identifier |
| $I$ | Input image matrix |
| $B$ | Structuring element (Kernel) |
| $\ominus$ | Morphological Erosion operator |
| $\oplus$ | Morphological Dilation operator |
| $\circ$ | Morphological Opening operator |

---

## Mathematical Foundation: Morphological Line Detection

To detect the structural grid of a table, the tool applies morphological opening. This operation isolates structures that fit a specific kernel $B$ (e.g., a long horizontal or vertical line) while suppressing surrounding text. The opening of an image $I$ by a structuring element $B$ is defined mathematically as an erosion followed by a dilation:

$$I \circ B = (I \ominus B) \oplus B$$

The script defines a horizontal kernel ($1 \times L$) and a vertical kernel ($L \times 1$) to isolate the respective table borders. These resulting matrices are combined to reveal the complete table grid.

---

## Pipeline Architecture

1.  **Image Ingestion:** The script prompts for a `.pdf` or `.docx` file. It rasterizes PDF pages into memory using Poppler or extracts embedded media directly from the DOCX archive.
2.  **GPU Acceleration:** The image matrix is transferred to the GPU VRAM via CuPy, converted to grayscale, and binarized.
3.  **Morphological Scanning:** The GPU applies the line detection kernels to find the table grids.
4.  **Contour Extraction:** The CPU calculates the bounding boxes of the detected grids and applies a padding margin.
5.  **Vector Embedding:** The cropped matrix is encoded into a Base64 URI and embedded inside an SVG Document Object Model.
6.  **Interactive Delivery:** Dynamic HTML download buttons are generated within the Colab output cell, ensuring zero intermediate files are written to the virtual disk.

---

## Control Knobs

The script features an upfront parameter block. You can adjust these top-level variables to tune the extraction sensitivity and output dimensions:

* **`SCAN_DPI`** (Default: `300`): The resolution used when rasterizing PDF pages. Higher values yield better clarity at the cost of memory. I prefer using 600 or 1000 DPI for better clarity.
* **`MORPH_KERNEL_LENGTH`** (Default: `50`): The length in pixels of the line-detection kernel. Increase this if the script detects text lines as tables; decrease it if small tables are being ignored.
* **`BINARIZATION_THRESH`** (Default: `200`): The pixel intensity threshold (0-255) for separating ink from the page background.
* **`PADDING_PX`** (Default: `15`): The physical pixel margin added around the detected table bounding box before cropping.
* **`TARGET_WIDTH_PX`** (Default: `2000`): The physical width of the generated SVG container. The height scales proportionally.

---

## Environment Setup and Dependencies

This project relies on system-level binaries for PDF rendering and `uv` for extremely fast Python package resolution within the Colab environment. The code adheres strictly to PEP 8 style guidelines and PEP 257 docstring conventions. 

Ensure your Colab runtime is set to utilize a GPU (e.g., T4, L4, G4, etc) before execution.

```bash
# 1. Update system package lists
!apt-get update

# 2. Install system-level binary dependencies required for PDF rasterization
!apt-get install -y poppler-utils

# 3. Bootstrap uv into the current Colab environment
!pip install uv

# 4. Use uv to rapidly install Python libraries into the system environment
!uv pip install --system pdf2image python-docx opencv-python-headless cupy-cuda12x
