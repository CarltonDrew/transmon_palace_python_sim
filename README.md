# Transmon Qubit Simulation Suite: CAD to Microwave Control

A complete, end-to-end simulation framework for superconducting quantum circuits, bridging 3D CAD modeling, finite element method (FEM) electrostatics via AWS Palace, and open quantum systems dynamics.

---

## Repository Structure

transmon_palace_python_sim/
├── README.md                          # Project documentation
├── .gitignore                         # Ignored files/folders
├── requirements.txt                   # Python package dependencies
│
├── cad/
│   └── qubit_heterostructure.FCStd    # FreeCAD 3D model source
│
├── mesh/
│   └── qubit_scaled.msh               # Gmsh tetrahedral mesh file
│
├── config/
│   └── simulation.json                # AWS Palace electrostatic configuration
│
└── notebooks/
    ├── transmon_spectrum.ipynb           # scqubits energy spectrum & charge noise analysis
    └── microwave_control.ipynb           # QuTiP Rabi oscillations & T1 relaxation script

## Workflow Overview

    1. CAD Modeling (cad/): Designed in FreeCAD with spatial length scaling (L0 = 1.0e-3) to handle multi-scale geometries (from millimeter pads down to nanometer-scale junction barriers).

    2. Mesh Generation (mesh/): Tetrahedral mesh generated via Gmsh, mapping physical surface tags for MFEM compatibility. 
    
    3. Electrostatic FEA (config/): Solves the Maxwell capacitance matrix using AWS Palace.
    
    4. Quantum Analysis & Control (notebooks/): (transmon_spectrum.ipynb) Uses scqubits to analyze energy levels, charging energy (E_C), Josephson energy (E_J), and offset charge noise insensitivity. (microwave_control_and_relaxation.ipynb) Uses QuTiP to simulate coherent microwave control pulses and Lindblad open-system energy relaxation (T_1).

## Installation & Requirements

    1. Install the required Python dependencies using pip:

    pip install -r requirements.txt

## Josephson Junction Modeling

    1. I highly recommend using FreeCAD and gmsh to create your own junction geometry. The included results are for a 2D heterostructure Josephson junction that can be considered NbSe2/h-BN (1 nm)/NbSe2. 

## Running the Simulations

    1. Run the AWS Palace Electrostatics solver

    mpirun -np 8 palace config/simulation.json

    2. Input AWS capacitances into quantum scripts              (TODO: Automate this step)

    # Step 1. Load Maxwell Capacitance Matrix from Palace output
        C_ij = [[C11, C12], [C21, C22]] in Farads
    
    # Step 2. Electrostatics to E_c
        C_m = abs(C_12)
        C_g1 = C_11 - C_m
        C_g2 = C_22 - C_m
        C_sigma = C_m + (C_g1 * C_g2) / (C_g1 + C_g2)
        e = 1.602176634e-19
        h = 6.62607015e-34

        E_c_joules = (e**2) / (2 * C_sigma)
        E_c_hz = E_c_joules / h

    # Step 3. Target f_01 -> Ej
        f_01 = 5.0e9  # 5 GHz                                   (I am told this is standard)
        E_j_hz = ((f_01 + E_c_hz)**2) / (8 * E_c_hz)

    3. Explore the results in the notebooks

    jupyter lab

    Navigate to the notebooks/ folder to explore transmon_spectrum.ipynb and microwave_control.ipynb.

