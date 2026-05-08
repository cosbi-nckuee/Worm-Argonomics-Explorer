# *Worm Argonomics Explorer*: Construction of a Database of Argonaute Target Transcripts in *C. elegans*

## Overview

**Worm Argonomics Explorer** provides a Docker-based analysis pipeline for processing Argonaute-related small RNA sequencing data in *Caenorhabditis elegans*.

The pipeline supports two analysis modes:

- **IP mode**: Argonaute immunoprecipitation/input analysis.
- **MUT mode**: Argonaute mutant/wild-type analysis.

The currently supported GEO datasets are:

| GEO accession | Supported analysis mode | Description |
|---|---|---|
| GSE208702 | IP and MUT | Argonaute IP/input data and Argonaute mutant/wild-type data |
| GSE212382 | IP | Argonaute IP/input data |

Users must prepare the required `.fastq` files before running the Docker pipeline. The pipeline checks whether each supported group is complete. Only complete groups will be processed.

---

## 1. Install Docker

We recommend using Docker to set up the required environment for the Worm Argonomics Explorer pipeline.

If Docker has already been installed, skip this step.

- For Microsoft Windows systems, follow the Docker Desktop installation guide:  
  https://docs.docker.com/desktop/install/windows-install/

- For macOS systems, follow the Docker Desktop installation guide:  
  https://docs.docker.com/desktop/install/mac-install/

After installation, start Docker Desktop before running the pipeline.

Do not stop Docker while the analysis is running.

---

## 2. Pull the Docker Image

Open a terminal and run:

```bash
docker pull cosbincku/wago:latest
```

---

## 3. Prepare FASTQ Input Files

Users need to prepare `.fastq` files from the supported GEO datasets.

The supported datasets can be searched from the NCBI GEO website:

```text
https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi
```

Users may download the required SRA/FASTQ files manually from NCBI GEO/SRA or use `fasterq-dump` from the SRA Toolkit.

---

## 4. Optional: Download FASTQ Files Using `fasterq-dump`

### 4.1 Download GSE208702 FASTQ Files

GSE208702 contains 147 SRR files:

```text
SRR20334616 to SRR20334762
```

Example command:

```bash
mkdir -p GSE208702
cd GSE208702

for i in $(seq 20334616 20334762); do
    fasterq-dump SRR${i}
done
```

This will generate files such as:

```text
SRR20334616.fastq
SRR20334617.fastq
...
SRR20334762.fastq
```

---

### 4.2 Download GSE212382 FASTQ Files

GSE212382 contains 22 SRR files:

```text
SRR21374053 to SRR21374074
```

Example command:

```bash
mkdir -p GSE212382
cd GSE212382

for i in $(seq 21374053 21374074); do
    fasterq-dump SRR${i}
done
```

This will generate files such as:

```text
SRR21374053.fastq
SRR21374054.fastq
...
SRR21374074.fastq
```

---

### 4.3 Required Merging for GSE212382

For GSE212382, two merged FASTQ files must be created before running the pipeline.

| Original FASTQ files | Required merged FASTQ file |
|---|---|
| `SRR21374061.fastq` + `SRR21374062.fastq` | `SRR2137406162.fastq` |
| `SRR21374073.fastq` + `SRR21374074.fastq` | `SRR2137407374.fastq` |

Run:

```bash
cat SRR21374061.fastq SRR21374062.fastq > SRR2137406162.fastq
cat SRR21374073.fastq SRR21374074.fastq > SRR2137407374.fastq
```

After merging, the original files may be removed if desired:

```bash
rm SRR21374061.fastq SRR21374062.fastq SRR21374073.fastq SRR21374074.fastq
```

---

## 5. Input Folder Structure

Prepare a local folder containing the `.fastq` files.

The FASTQ files must be placed directly inside the mounted input folder. Do not put them inside another subfolder.

Example:

```text
/your/local/folder/
├── SRR20334753.fastq
├── SRR20334715.fastq
├── SRR20334754.fastq
├── SRR20334716.fastq
├── SRR20334652.fastq
├── SRR20334639.fastq
├── SRR20334619.fastq
├── SRR20334678.fastq
├── SRR20334665.fastq
└── SRR20334660.fastq
```

The pipeline only accepts files ending with:

```text
.fastq
```

---

## 6. Run the Pipeline

### 6.1 Run IP Mode

Use IP mode for Argonaute IP/input analysis.

```bash
docker run --rm \
    --name wago \
    -v {/your/local/folder}:/WormArgonomicsExplorer/in_out_file \
    -it cosbincku/wago \
    bash run.sh IP
```

Replace:

```text
{/your/local/folder}
```

with the real path to the folder containing the `.fastq` files.

Example:

```bash
docker run --rm \
    --name wago \
    -v /home/user/wago_fastq:/WormArgonomicsExplorer/in_out_file \
    -it cosbincku/wago \
    bash run.sh IP
```

---

### 6.2 Run MUT Mode

Use MUT mode for Argonaute mutant/wild-type analysis.

```bash
docker run --rm \
    --name wago \
    -v {/your/local/folder}:/WormArgonomicsExplorer/in_out_file \
    -it cosbincku/wago \
    bash run.sh MUT
```

Example:

```bash
docker run --rm \
    --name wago \
    -v /home/user/wago_fastq:/WormArgonomicsExplorer/in_out_file \
    -it cosbincku/wago \
    bash run.sh MUT
```

---

## 7. Complete Group Requirement

The pipeline processes a group only when all required FASTQ files for that group are present.

For example, in IP mode, the following group belongs to `alg1` in GSE208702:

```text
SRR20334753 SRR20334715 SRR20334754 SRR20334716
```

This group will only be processed if all four files exist:

```text
SRR20334753.fastq
SRR20334715.fastq
SRR20334754.fastq
SRR20334716.fastq
```

If one or more files are missing, this group will be skipped.

Similarly, in MUT mode, the following group belongs to `alg1`:

```text
SRR20334678 SRR20334665 SRR20334660 SRR20334652 SRR20334639 SRR20334619
```

This group will only be processed if all six files exist.

---

## 8. Supported IP Groups

### 8.1 GSE208702 IP Groups

Each GSE208702 IP group requires four files:

```text
IP replicate 1, IP replicate 2, input replicate 1, input replicate 2
```

| Argonaute | Required SRR files |
|---|---|
| alg1 | SRR20334753, SRR20334715, SRR20334754, SRR20334716 |
| alg2 | SRR20334751, SRR20334713, SRR20334752, SRR20334714 |
| alg5 | SRR20334745, SRR20334707, SRR20334746, SRR20334708 |
| rde1 | SRR20334731, SRR20334693, SRR20334732, SRR20334694 |
| alg3 | SRR20334749, SRR20334711, SRR20334750, SRR20334712 |
| alg4 | SRR20334747, SRR20334709, SRR20334748, SRR20334710 |
| ergo1 | SRR20334741, SRR20334703, SRR20334742, SRR20334704 |
| prg1 | SRR20334733, SRR20334695, SRR20334734, SRR20334696 |
| csr1 | SRR20334743, SRR20334705, SRR20334744, SRR20334706 |
| vsra1 | SRR20334761, SRR20334723, SRR20334762, SRR20334724 |
| wago1 | SRR20334757, SRR20334719, SRR20334758, SRR20334720 |
| ppw2 | SRR20334735, SRR20334697, SRR20334736, SRR20334698 |
| wago4 | SRR20334725, SRR20334687, SRR20334726, SRR20334688 |
| ppw1 | SRR20334737, SRR20334699, SRR20334738, SRR20334700 |
| sago2 | SRR20334727, SRR20334689, SRR20334728, SRR20334690 |
| sago1 | SRR20334729, SRR20334691, SRR20334730, SRR20334692 |
| hrde1 | SRR20334739, SRR20334701, SRR20334740, SRR20334702 |
| wago10 | SRR20334755, SRR20334717, SRR20334756, SRR20334718 |
| nrde3 | SRR20334759, SRR20334721, SRR20334760, SRR20334722 |

---

### 8.2 GSE212382 IP Groups

Each GSE212382 IP group requires two files:

```text
IP sample, merged/common input sample
```

The common input file is:

```text
SRR2137407374.fastq
```

| Argonaute | Required SRR files |
|---|---|
| alg1 | SRR21374072, SRR2137407374 |
| alg2 | SRR21374071, SRR2137407374 |
| alg3 | SRR21374070, SRR2137407374 |
| alg4 | SRR21374069, SRR2137407374 |
| alg5 | SRR21374068, SRR2137407374 |
| vsra1 | SRR21374067, SRR2137407374 |
| csr1 | SRR21374066, SRR2137407374 |
| ergo1 | SRR21374065, SRR2137407374 |
| hrde1 | SRR21374064, SRR2137407374 |
| nrde3 | SRR21374063, SRR2137407374 |
| ppw1 | SRR2137406162, SRR2137407374 |
| ppw2 | SRR21374060, SRR2137407374 |
| prg1 | SRR21374059, SRR2137407374 |
| rde1 | SRR21374058, SRR2137407374 |
| sago1 | SRR21374057, SRR2137407374 |
| sago2 | SRR21374056, SRR2137407374 |
| wago1 | SRR21374055, SRR2137407374 |
| wago10 | SRR21374054, SRR2137407374 |
| wago4 | SRR21374053, SRR2137407374 |

---

## 9. Supported MUT Groups

MUT mode compares Argonaute mutant samples against wild-type samples.

The three common wild-type samples are required for every MUT group:

| Wild-type replicate | SRR ID |
|---|---|
| WT rep1 | SRR20334652 |
| WT rep2 | SRR20334639 |
| WT rep3 | SRR20334619 |

Therefore, each MUT group requires six FASTQ files:

```text
mut1, mut2, mut3, WT1, WT2, WT3
```

| Argonaute | Mutant SRR files | Required WT SRR files |
|---|---|---|
| alg1 | SRR20334678, SRR20334665, SRR20334660 | SRR20334652, SRR20334639, SRR20334619 |
| alg2 | SRR20334677, SRR20334664, SRR20334659 | SRR20334652, SRR20334639, SRR20334619 |
| alg5 | SRR20334674, SRR20334645, SRR20334656 | SRR20334652, SRR20334639, SRR20334619 |
| rde1 | SRR20334647, SRR20334634, SRR20334629 | SRR20334652, SRR20334639, SRR20334619 |
| alg3 | SRR20334676, SRR20334663, SRR20334658 | SRR20334652, SRR20334639, SRR20334619 |
| alg4 | SRR20334675, SRR20334646, SRR20334657 | SRR20334652, SRR20334639, SRR20334619 |
| ergo1 | SRR20334654, SRR20334641, SRR20334621 | SRR20334652, SRR20334639, SRR20334619 |
| prg1 | SRR20334648, SRR20334635, SRR20334630 | SRR20334652, SRR20334639, SRR20334619 |
| csr1 | SRR20334672, SRR20334643, SRR20334623 | SRR20334652, SRR20334639, SRR20334619 |
| vsra1 | SRR20334673, SRR20334644, SRR20334655 | SRR20334652, SRR20334639, SRR20334619 |
| wago1 | SRR20334668, SRR20334631, SRR20334626 | SRR20334652, SRR20334639, SRR20334619 |
| ppw2 | SRR20334649, SRR20334636, SRR20334616 | SRR20334652, SRR20334639, SRR20334619 |
| wago4 | SRR20334666, SRR20334661, SRR20334624 | SRR20334652, SRR20334639, SRR20334619 |
| ppw1 | SRR20334650, SRR20334637, SRR20334617 | SRR20334652, SRR20334639, SRR20334619 |
| sago2 | SRR20334669, SRR20334632, SRR20334627 | SRR20334652, SRR20334639, SRR20334619 |
| sago1 | SRR20334670, SRR20334633, SRR20334628 | SRR20334652, SRR20334639, SRR20334619 |
| hrde1 | SRR20334653, SRR20334640, SRR20334620 | SRR20334652, SRR20334639, SRR20334619 |
| wago10 | SRR20334667, SRR20334662, SRR20334625 | SRR20334652, SRR20334639, SRR20334619 |
| nrde3 | SRR20334651, SRR20334638, SRR20334618 | SRR20334652, SRR20334639, SRR20334619 |

---

## 10. Example Input Folder for IP Mode

To run IP mode for `alg1` from GSE208702, prepare:

```text
/your/local/folder/
├── SRR20334753.fastq
├── SRR20334715.fastq
├── SRR20334754.fastq
└── SRR20334716.fastq
```

Then run:

```bash
docker run --rm \
    --name wago \
    -v /your/local/folder:/WormArgonomicsExplorer/in_out_file \
    -it cosbincku/wago \
    bash run.sh IP
```

---

## 11. Example Input Folder for MUT Mode

To run MUT mode for `alg1`, prepare:

```text
/your/local/folder/
├── SRR20334678.fastq
├── SRR20334665.fastq
├── SRR20334660.fastq
├── SRR20334652.fastq
├── SRR20334639.fastq
└── SRR20334619.fastq
```

Then run:

```bash
docker run --rm \
    --name wago \
    -v /your/local/folder:/WormArgonomicsExplorer/in_out_file \
    -it cosbincku/wago \
    bash run.sh MUT
```

---

## 12. Output

After the analysis finishes, output files will be written to:

```text
{/your/local/folder}/output
```

The exact output contents depend on the selected mode and the complete groups detected in the input folder.

In general:

- **IP mode** produces Argonaute IP/input-based target analysis results.
- **MUT mode** produces mutant/wild-type differential analysis results and target lists.

---

## 13. Notes

1. Input files must end with `.fastq`.
2. Each supported Argonaute group is processed only when all required FASTQ files are present.
3. Missing or incomplete groups are skipped automatically.
4. For GSE212382, `SRR2137406162.fastq` and `SRR2137407374.fastq` must be generated by merging the required original FASTQ files before running the pipeline.
5. The mounted local folder should contain FASTQ files directly, not inside nested subfolders.
6. IP mode and MUT mode should be run separately.
