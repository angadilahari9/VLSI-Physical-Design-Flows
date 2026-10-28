# VLSI-Physical-Design-Flows
# ASIC Physical Design: Netlist-to-GDSII Implementation Flow & Automation

This repository houses a comprehensive documentation layout, structured command references, and architectural checklists detailing the ASIC Physical Design implementation flow. It serves as a portfolio highlighting core competencies in digital backend design, timing closure strategies, and EDA tool workflows utilizing industry-standard suites from Synopsys and Cadence.

---

## 🛠️ Technology & Tool Stack
*   **EDA Suites:** Synopsys IC Compiler II (ICC2), Design Compiler, PrimeTime | Cadence Innovus, Virtuoso
*   **Sign-Off Verification:** Layout Versus Schematic (LVS), Design Rule Checking (DRC)
*   **Automation & Scripting:** Tcl Scripting (Tool Command Language), Shell/Bash Scripting
*   **Operating Environment:** Linux RedHat / CentOS

---

## 📐 Featured Project: Hierarchical Physical Design of an L-Shaped Block
*Executed a complete netlist-to-GDSII physical implementation flow for a multi-million gate rectilinear L-shaped block using the Synopsys ICC2 environment.*

### Key Architectural & Floorplanning Challenges Managed:
*   **Non-Rectangular Floorplanning:** Configured complex core boundaries for the L-shaped block, optimizing block placement constraints to prevent routing congestion at the inside corner.
*   **Macro Placement Strategy:** Structured macro clusters and channel widths aligned with standard data flows, utilizing halos and placement blockages to keep the central channel completely clear for standard cells.
*   **Power Network Synthesis (PNS):** Designed non-uniform power distribution rings, straps, and rails (`VDD`/`VSS`) to prevent electromigration issues and safely handle dynamic IR drop across asymmetrical narrow core regions.
*   **Clock Tree Synthesis (CTS):** Handled asymmetrical clock distribution paths across the L-shaped geometry. Used multi-tap clock tree balancing strategies to meet rigorous global clock skew limits and minimize insertion delay.
*   **Timing Closure (PrimeTime):** Analyzed multi-corner multi-mode (MCMM) timing reports. Fixed setup and hold slack violations under explicit SDC constraints using cell resizing, driving-strength adjustments, and strategic buffer insertions.

---

## 🧭 Comprehensive Physical Design Flow

### 1. Design Setup & Inputs Handoff
Before launching the placement and routing (P&R) flow, the design database must be initialized with consistent logical and physical libraries.
*   **Logical Inputs:** Gate-level netlist (`.v`), Synopsys Design Constraints (`.sdc`), and timing models (`.lib` / `.db`).
*   **Physical Inputs:** Process Design Kit (PDK) files, LEF files (Technology & Cell LEFs), or Synopsys Workspace/NDM (New Data Model) libraries.
*   **Power Intent:** Unified Power Format (`.upf`) or Common Power Format (`.cpf`) definitions for multi-voltage designs.

### 2. Standard Cell Placement & Optimization
Assigning valid physical grid coordinates to the netlist components while optimizing for timing, power, and area (PPA).
*   **Pre-Placement Checks:** Verifying the design checks for unconstrained paths, missing libraries, or dangling nets.
*   **Global & Detail Placement:** Distributing cells uniformly to minimize total wirelength and cell density spikes.
*   **Congestion & Timing Check:** Analyzing global routing congestion maps. If high density or local hotspots occur, standard cell keep-out margins or bounds are applied.

### 3. Detailed Routing
Creating the actual physical metal connections for the signal nets following the structural rules defined by the technology file.
*   **Global Routing:** Defining coarse paths for the routing connections based on routing resource availability.
*   **Detail Routing:** Utilizing the tool's routing engine to physically lay down metal tracks and connect vias while satisfying manufacturing constraints.
*   **Optimization Iterations:** Running continuous search-and-repair functions to eliminate Shorts, Opens, and antenna effects.

### 4. Sign-Off Physical & Timing Verification
Validating that the completed physical GDSII layout accurately reflects the logical netlist definitions and meets all hardware fabrication rules.
*   **Static Timing Analysis (STA):** Using sign-off engines (like PrimeTime) to confirm complete Setup and Hold timing closure across all operating corners.
*   **Physical Sign-Off:** Running complete Design Rule Checking (DRC) to ensure metal spacing compliance and Layout Versus Schematic (LVS) to guarantee layout connectivity matches the gate-level netlist perfectly.

---

## 🎛️ Essential Tool Command Reference

### Synopsys Environment (ICC2 / PrimeTime)

| Design Phase | Command | Purpose |
| :--- | :--- | :--- |
| **Setup** | `open_lib`, `read_parasitic_tech` | Opens the design library and loads RC tech files. |
| **Floorplan** | `initialize_design`, `create_placement_blockage` | Sets core size/utilization and bounds cell areas. |
| **Placement** | `place_opt` | Executes global placement and coarse optimization. |
| **CTS** | `clock_opt` | Synthesizes the clock tree structures and balances skew. |
| **Routing** | `route_auto`, `route_opt` | Runs global/detail routing and repairs spacing errors. |
| **Timing** | `report_timing -delay_type max_min` | Generates comprehensive setup/hold timing paths. |

### Cadence Environment (Innovus)

| Design Phase | Command | Purpose |
| :--- | :--- | :--- |
| **Setup** | `init_design` | Reads the configuration files and initial definitions. |
| **Floorplan** | `floorplan`, `addRing`, `addStripe` | Initializes die boundaries and crafts power distribution meshes. |
| **Placement** | `place_opt_design` | Places standard cells and runs initial timing fixes. |
| **CTS** | `ccopt_design` | Invokes the concurrent clock optimization architecture. |
| **Routing** | `routeDesign` | Performs detailed routing of signal connections. |
| **Timing** | `timeDesign -postRoute` | Extracts parasitics and builds post-route timing matrices. |

---

## 📝 Scripting Logic & Automation Outlines
*Below are standard structural pseudocode flows for automating day-to-day backend tasks using Tool Command Language (Tcl).*

### Example 1: Log Parsing Script for WNS Extraction
```tcl
# Open the execution log generated by the P&R tools
set log_file [open "innovus_placement.log" r]
set wns "0.0"

while {[gets \$log_file line] >= 0} {
    # Search for lines indicating the Worst Negative Slack
    if {[regexp {WNS:\s+(-?[0-9.]+)} \$line match value]} {
        set wns \$value
    }
}
close \$log_file
puts "Extracted Worst Negative Slack for this run: \${wns} ns"
```

### Example 2: Automated Slack Violations Analyzer
```tcl
# Query all failing timing endpoints in the design database
set failing_paths [get_timing_paths -delay_type max -max_paths 100 -slack_less_than 0.0]

foreach_in_collection path \$failing_paths {
    set endpoint [get_object_name [get_property \$path endpoint]]
    set slack [get_property \$path slack]
    # Log information to categorize high-priority fixing paths
    puts "Failing Endpoint: endpoint | Slack: slack ns"
}
```
