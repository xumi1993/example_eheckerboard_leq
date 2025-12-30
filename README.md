# Checkerboard Test Example for Local Earthquake Adjoint Tomography with SpecFWAT

This project demonstrates a checkerboard resolution test for Local Earthquake (LEQ) tomography using `specfwat`. It sets up a synthetic experiment with a known checkerboard perturbation, simulates data, and attempts to recover the model using full waveform inversion.

## Project Structure

*   **`01_create_model_ckbd.ipynb`**: Jupyter notebook to generate the initial 1D model and the target 3D model with checkerboard perturbations.
*   **`02_setup_src_rec.ipynb`**: Jupyter notebook to randomly generate source positions and a grid of receivers, updating the `CMTSOLUTION` and `STATIONS` files.
*   **`03_forward.sh`**: Shell script to generate synthetic data based on the target model.
*   **`04_run_this_test.sh`**: The main shell script to run the iterative inversion process.
*   **`DATA/`**: Contains configuration files and data directories.
    *   `fwat_params.yml`: Main configuration file for `specfwat` parameters (filters, windows, etc.).
    *   `Par_file`: SPECFEM3D parameter file.
    *   `meshfem3D_files/`: Mesh generation parameters.
*   **`src_rec/`**: Directory containing source (`CMTSOLUTION`) and receiver (`STATIONS`) files for each event.
*   **`ak135.txt`**: 1D velocity model file.

## Prerequisites

*   **SpecFWAT**: The `specfwat` binaries (`xfwat_mesh_databases`, `xfwat_fwd_measure_adj`, `xfwat_post_proc`, `xfwat_optimize`) must be compiled and available.
*   **MPI**: An MPI implementation (e.g., OpenMPI) to run the parallel executables.
*   **Python**: Python 3 with the following packages:
    *   `numpy`
    *   `scipy`
    *   `h5py`
    *   `matplotlib`
    *   `pyfwat` (Python interface for SpecFWAT)

## Usage

### 1. Generate Models
Run the `01_create_model_ckbd.ipynb` notebook to create the initial and target models.
*   This will generate `initial_model.h5` (1D background) and `target_model.h5` (with checkerboard perturbation).

### 2. Setup Sources and Receivers
Run the `02_setup_src_rec.ipynb` notebook to configure the acquisition geometry.
*   This will generate random source locations and a regular receiver grid.
*   It updates the `CMTSOLUTION` and `STATIONS` files in the `src_rec/` directory.

### 3. Generate Synthetic Data
Execute the `03_forward.sh` script to generate the synthetic dataset using the target model.

```bash
./03_forward.sh
```

This script copies `target_model.h5` to the tomography directory and runs a forward simulation to create the "observed" data for the inversion.

### 4. Run Inversion
Execute the `04_run_this_test.sh` script to start the inversion loop.

```bash
./04_run_this_test.sh
```

This script performs the following steps for a specified number of iterations (default is 9):
1.  **Initialization**: Copies `initial_model.h5` to `DATA/tomo_files/tomography_model.h5` (only in the first iteration).
2.  **Meshing**: Runs `xfwat_mesh_databases`.
3.  **Forward/Adjoint**: Runs `xfwat_fwd_measure_adj` to simulate wave propagation, measure misfits, and calculate adjoint sources.
4.  **Post-processing**: Runs `xfwat_post_proc` to compute the gradient.
5.  **Optimization**: Runs `xfwat_optimize` to update the model.

## Configuration

You can adjust the inversion parameters in `DATA/fwat_params.yml`. Key parameters include:
*   `IMEAS`: Measurement type (e.g., 4 for exponentiated phase misfit).
*   `SHORT_P` / `LONG_P`: Period bands for filtering.
*   `WINDOW`: Parameters for the time window selection algorithm.
*   `SIGMA_H` / `SIGMA_V`: Smoothing lengths for the gradient.
