### Svetlana-Singularity Container VUB-HPC

```markdown
# Svetlana Singularity Container Recipe

This repository provides the Singularity (Apptainer) definition file for the Svetlana environment. The container is designed to run Napari (v0.4.17 with all extras) together with the napari_svetlana plugin (v1.6.1) and additional dependencies needed for GPU-accelerated processing. The container includes:

- **Conda Environment:** `svetlana_env` with Python 3.9 created via Miniforge.
- **Napari:** Installed with all extras (v0.4.17).
- **CUDA Toolkit:** cudatoolkit 11.8 from conda-forge.
- **PyTorch:** Version 2.3.0+cu118 built for CUDA 11.8, along with torchvision and torchaudio via pip.
- **Cupy:** Version 13.4.* (from conda-forge) for CUDA 11.8.
- **Additional Dependencies:** pydantic (1.10.21), NumPy (pinned to 1.24.3), scikit‑learn (1.6.1), ttach, and grad-cam (provides the `pytorch_grad_cam` module).

## Singularity Definition File (`svetlana.def`)

```singularity
Bootstrap: docker
From: abcdesktopio/oc.template.ubuntu.22.04:3.2

%files
    # No extra files to include

%environment
    # Define the conda binary path and add it to the PATH
    CONDA_BIN_PATH="/opt/conda/bin"
    export PATH="$CONDA_BIN_PATH:$PATH"
    # Add the svetlana_env libraries to LD_LIBRARY_PATH
    export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/opt/conda/envs/svetlana_env/lib/"
    export DEBIAN_FRONTEND=noninteractive
    export TZ=Europe/Berlin
    export QT_XCB_NO_MITSHM=1

%post
    # Update system packages and install prerequisites
    apt-get update -y && \
    apt-get install -y -q --no-install-recommends \
         gcc \
         wget \
         tzdata \
         qtcreator \
         python3-dev \
         python3-pip \
         python3-wheel \
         libblas-dev \
         liblapack-dev \
         libgl1 \
         mesa-utils \
         libxcb-cursor0 \
         libgl1-mesa-glx \
         libxcb-xinerama0 \
         libatlas-base-dev \
         gfortran \
         apt-utils \
         bzip2 \
         ca-certificates \
         curl \
         locales \
         libarchive-dev \
         cmake \
         libxcb-cursor-dev \
         unzip && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

    # Enable en_US.UTF-8 locale
    sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && locale-gen

    # Install Miniforge (Conda) into /opt/conda
    cd /tmp && \
    wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh && \
    bash Miniforge3-Linux-x86_64.sh -b -p /opt/conda && \
    rm -f Miniforge3-Linux-x86_64.sh

    # Create a new conda environment "svetlana_env" with Python 3.9 from conda-forge
    /opt/conda/bin/conda create -y --name svetlana_env python=3.9 -c conda-forge

    # Ensure pip is installed in the environment
    /opt/conda/bin/conda run --name svetlana_env conda install -y pip

    # Install Napari (with all extras) version 0.4.17 via pip
    /opt/conda/bin/conda run --name svetlana_env python -m pip install "napari[all]"==0.4.17Below is a complete, finalized Markdown file you can publish on GitHub. It contains the full Singularity (Apptainer) definition file for the Svetlana environment along with build and run instructions that enable GPU support.

---

    # Install cudatoolkit 11.8 via conda-forge (official command)
    /opt/conda/bin/conda install -y --name svetlana_env conda-forge::cudatoolkit=11.8

    # Install the napari_svetlana plugin version 1.6.1 via pip
    /opt/conda/bin/conda run --name svetlana_env python -m pip install napari_svetlana==1.6.1

    # Install additional Python dependencies
    /opt/conda/bin/conda run --name svetlana_env python -m pip install pydantic==1.10.21
    /opt/conda/bin/conda install -y --name svetlana_env -c conda-forge "numpy==1.24.3"
    /opt/conda/bin/conda install -y --name svetlana_env -c conda-forge scikit-learn==1.6.1

    # Install PyTorch, torchvision, and torchaudio for CUDA 11.8 via pip
    /opt/conda/bin/conda run --name svetlana_env python -m pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

    # Install Cupy 13.4.* for CUDA 11.8 from conda-forge
    /opt/conda/bin/conda install -y --name svetlana_env -c conda-forge "cupy=13.4.*"

    # Install ttach (dependency for napari_svetlana)
    /opt/conda/bin/conda run --name svetlana_env python -m pip install ttach

    # Install grad-cam to supply the pytorch_grad_cam module dependency
    /opt/conda/bin/conda run --name svetlana_env python -m pip install grad-cam

%runscript
    # Launch Napari from the svetlana_env environment
    exec /opt/conda/envs/svetlana_env/bin/napari "$@"
```

## Building the Container

To build the container, run (with root privileges):

```bash
sudo singularity build svetlana.sif svetlana.def
```

Or, if your system supports fakeroot:

```bash
singularity build --fakeroot svetlana.sif svetlana.def
```

## Running the Container with GPU Support

To run the container with GPU support, use the `--nv` flag:

```bash
singularity run --nv svetlana.sif
```

This will bind the host’s NVIDIA drivers and libraries to the container, enabling GPU acceleration for PyTorch and Cupy.

---

Enjoy your GPU-enabled Napari environment with the Svetlana plugin!

If you have any questions or issues, please feel free to open an issue or contact me.
```
