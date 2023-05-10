# cudaversionmanager

```bash
# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color

# Function to check if a directory exists
function check_dir() {
    if [ ! -d "$1" ]; then
        echo -e "${RED}$2${NC}"
        return 1
    fi
    return 0
}

# Function to check if a command outputs a specific string
function check_cmd() {
    if [[ ! $1 == *"$2"* ]]; then
        echo -e "${RED}$3${NC}"
        return 1
    fi
    return 0
}

# Main function to set CUDA and cuDNN versions
function SetCUDA() {
    # Check if a parameter was passed
    if [ -z "$1" ]; then
        echo -e "${RED}Please specify a CUDA version.${NC}"
        return
    fi

    # Check CUDA hardware and drivers
    nvidia_smi=$(nvidia-smi)
    smi_error="No CUDA hardware detected or NVIDIA driver is not installed."
    check_cmd "$nvidia_smi" "NVIDIA-SMI" "$smi_error" || return

    # Default cuDNN versions for each CUDA version
    declare -A cudnn_versions
    cudnn_versions=(["11"]="8.0.5" ["12"]="8.1.0")

    cuda_dir="/usr/local/cuda-$1"
    cuda_error="CUDA version $1 is not installed."

    # If the specific CUDA version directory does not exist, try the general /usr/local/cuda directory
    if ! check_dir "$cuda_dir" "$cuda_error"; then
        cuda_dir="/usr/local/cuda"
        if ! check_dir "$cuda_dir" "CUDA is not installed."; then
            return
        fi
    fi

    export CUDA_HOME=$cuda_dir
    export PATH=$CUDA_HOME/bin:${PATH}
    export LD_LIBRARY_PATH=$CUDA_HOME/lib64:${LD_LIBRARY_PATH}

    nvcc_version=$(nvcc --version)
    nvcc_error="There was a problem setting the CUDA version. Please ensure that CUDA $1 is installed correctly."
    check_cmd "$nvcc_version" "$1" "$nvcc_error" || return

    cuda_major_version=${1%%.*}
    cudnn_version=${cudnn_versions[$cuda_major_version]}
    cudnn_dir="/usr/local/cudnn-$cudnn_version"
    cudnn_error="cuDNN version $cudnn_version is not installed."
    check_dir "$cudnn_dir" "$cudnn_error" || return

    export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$CUDA_HOME/extras/CUPTI/lib64:$cudnn_dir/lib64:${LD_LIBRARY_PATH}
    echo -e "${GREEN}Set cuDNN version to $cudnn_version (relies on CUDA version)${NC}"
    echo -e "${GREEN}Switched to CUDA version $1${NC}"
}

```
