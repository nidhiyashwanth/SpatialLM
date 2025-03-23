# Running SpatialLM on Colab with Tesla T4

This guide documents my journey setting up and running [SpatialLM](https://github.com/manycore-research/SpatialLM) on Google Colab using a Tesla T4 GPU. I share the step-by-step process—from cloning the repo and installing dependencies, to manually fixing the `torchsparse` build (thanks to CUDA version issues), running inference, and finally visualizing the output.

## Environment Details

- **GPU:** Tesla T4 (CUDA 12.4)
- **Python:** 3.11
- **PyTorch:** 2.4.1
- **Colab:** Headless environment (for visualization, I downloaded the output and ran it on my Windows machine)

## Step-by-Step Process

### 1. Check GPU and Clone SpatialLM Repository

First, I verified that Colab was running on a Tesla T4 with CUDA 12.4, then I cloned the SpatialLM repository:

```bash
!nvidia-smi
!git clone https://github.com/manycore-research/SpatialLM.git
%cd SpatialLM
```

### 2. Install Dependencies with Poetry

I installed Poetry and set up the project dependencies. This part worked smoothly:

```bash
!pip install poetry
!poetry config virtualenvs.create false --local
!poetry install
```

### 3. Installing `torchsparse`

The default command (`!poe install-torchsparse`) didn’t work for me due to CUDA version mismatches, so I had to build and install `torchsparse` manually.

#### a. Clone the `torchsparse` Repository

```bash
!git clone https://github.com/mit-han-lab/torchsparse.git
%cd torchsparse
```

#### b. Update CUDA Version in `setup.py`

I replaced references to CUDA 11.0 with CUDA 12.4 to match the Colab environment:

```bash
!sed -i 's/11\.0/12.4/g' setup.py
```

#### c. Install SparseHash Dependency

`torchsparse` requires the SparseHash library. I installed its development files:

```bash
!apt-get update && apt-get install -y libsparsehash-dev
```

#### d. Build and Install `torchsparse`

After that, I built the wheel and installed it:

```bash
!python setup.py bdist_wheel
!pip install dist/*.whl
```

Then I returned to the SpatialLM directory:

```bash
%cd ..
```

### 4. Download the Test Point Cloud

I installed the Hugging Face Hub CLI and downloaded the test point cloud:

```bash
!pip install huggingface_hub
!huggingface-cli download manycore-research/SpatialLM-Testset pcd/scene0000_00.ply --repo-type dataset --local-dir .
```

### 5. Run Inference

Next, I ran the inference script to process the point cloud. This generated a text file containing the predicted scene layout:

```bash
!python inference.py --point_cloud pcd/scene0000_00.ply --output scene0000_00.txt --model_path manycore-research/SpatialLM-Llama-1B
```

### 6. Generate the Visualization File (.rrd)

I then converted the output into an RRD file for visualization:

```bash
!python visualize.py --point_cloud pcd/scene0000_00.ply --layout scene0000_00.txt --save scene0000_00.rrd
```

### 7. Visualize the RRD File

Since Colab is a headless environment, I couldn’t open a GUI window directly. Instead, I packaged and downloaded the RRD file:

```bash
!zip -r scene0000_00_rrd.zip scene0000_00.rrd
```

After downloading and extracting the file on my Windows machine, I used [rerun](https://rerun.io) to visualize it:

```bash
rerun scene0000_00.rrd
```

This command opened a window showing the point cloud and the structured 3D layout visualization.

## Troubleshooting and Personal Notes

- **torchsparse Installation:**  
  I initially tried `!poe install-torchsparse`, but it failed because of CUDA version mismatches. Manually cloning `torchsparse`, updating the CUDA version in `setup.py`, and installing the `libsparsehash-dev` package fixed the issue.

- **Visualization in Colab:**  
  Since Colab doesn’t support GUI applications, I had to download the `.rrd` file and visualize it on a local machine. Using `xvfb-run` didn’t help for this visualization, so I resorted to running rerun on Windows.

This README should help anyone trying to set up SpatialLM on Colab with similar hardware and environment settings. Enjoy exploring SpatialLM!
