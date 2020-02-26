###########################
# 
# Silva/ml: 1.0
# 
# 2/25/2020
#
#

ARG BASE_CONTAINER=jupyter/all-spark-notebook
FROM $BASE_CONTAINER


USER root

#### Some custom code

RUN apt-get update && \
    apt-get install -y htop && \
    apt-get install -y pciutils && \ 
    apt-get install -y lm-sensors && \
    rm -rf /var/lib/apt/lists/*
#
#
##End custom code


USER $NB_UID

# Install tensorflow, pytorch, cudatoolkit, keras, and other goodies for the jupyter notebook

RUN conda install -c anaconda 'tensorflow-gpu' 'keras-gpu' 'pillow' --quiet --yes && \
    conda install pytorch torchvision cudatoolkit=10.1 -c pytorch --quiet --yes && \
    conda install -c conda-forge jupyter_contrib_nbextensions --quiet --yes     && \
    conda install -c conda-forge 'jupyterlab-nvdashboard' 'pynvml' --quiet --yes && \
    conda clean --all -f -y  && \
    pip install torch-scatter==latest+cu101 torch-sparse==latest+cu101 -f https://s3.eu-central-1.amazonaws.com/pytorch-geometric.com/whl/torch-1.4.0.html && \
    pip install torch-cluster && \
    pip install torch-spline-conv && \
    pip install torch-geometric && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER && \
    jupyter labextension install jupyterlab-nvdashboard --no-build && \
    jupyter labextension install @krassowski/jupyterlab_go_to_definition@^0.7.1 --no-build && \
    jupyter labextension install @jupyterlab/toc@1.0.1 --no-build && \
    jupyter lab build && \
    npm cache clean --force && \
    rm -rf $CONDA_DIR/share/jupyter/lab/staging && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    rm -rf /home/$NB_USER/.node-gyp && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER


######## Cuda (nvidia/cuda):10.1 cudnn:7.6.5.32


##### Base

USER root

RUN apt-get update && apt-get install -y --no-install-recommends \
gnupg2 curl ca-certificates && \
    curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub | apt-key add - && \
    echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list && \
    echo "deb https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/nvidia-ml.list && \
    apt-get purge --autoremove -y curl && \
rm -rf /var/lib/apt/lists/*

ENV CUDA_VERSION 10.1.243

ENV CUDA_PKG_VERSION 10-1=$CUDA_VERSION-1

# For libraries in the cuda-compat-* package: https://docs.nvidia.com/cuda/eula/index.html#attachment-a
RUN apt-get update && apt-get install -y --no-install-recommends \
        cuda-cudart-$CUDA_PKG_VERSION \
cuda-compat-10-1 && \
ln -s cuda-10.1 /usr/local/cuda && \
    rm -rf /var/lib/apt/lists/*

# Required for nvidia-docker v1
RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf && \
    echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf

ENV PATH /usr/local/nvidia/bin:/usr/local/cuda/bin:${PATH}
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64

# nvidia-container-runtime
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
ENV NVIDIA_REQUIRE_CUDA "cuda>=10.1 brand=tesla,driver>=384,driver<385 brand=tesla,driver>=396,driver<397 brand=tesla,driver>=410,driver<411"

##### Runtime

ENV NCCL_VERSION 2.4.8

RUN apt-get update && apt-get install -y --no-install-recommends \
    cuda-libraries-$CUDA_PKG_VERSION \
cuda-nvtx-$CUDA_PKG_VERSION \
libcublas10=10.2.1.243-1 \
libnccl2=$NCCL_VERSION-1+cuda10.1 && \
    apt-mark hold libnccl2 && \
    rm -rf /var/lib/apt/lists/*

##### Devel

RUN apt-get update && apt-get install -y --no-install-recommends \
        cuda-nvml-dev-$CUDA_PKG_VERSION \
        cuda-command-line-tools-$CUDA_PKG_VERSION \
cuda-libraries-dev-$CUDA_PKG_VERSION \
        cuda-minimal-build-$CUDA_PKG_VERSION \
        libnccl-dev=$NCCL_VERSION-1+cuda10.1 \
libcublas-dev=10.2.1.243-1 \
&& \
    rm -rf /var/lib/apt/lists/*

ENV LIBRARY_PATH /usr/local/cuda/lib64/stubs

##### CDNN 7
 
ENV CUDNN_VERSION 7.6.5.32
LABEL com.nvidia.cudnn.version="${CUDNN_VERSION}"

RUN apt-get update && apt-get install -y --no-install-recommends \
    libcudnn7=$CUDNN_VERSION-1+cuda10.1 \
libcudnn7-dev=$CUDNN_VERSION-1+cuda10.1 \
&& \
    apt-mark hold libcudnn7 && \
    rm -rf /var/lib/apt/lists/*


## End of cuda installation

## Set paths for cuda
ENV PATH=/usr/local/cuda/bin:$PATH
ENV CPATH=/usr/local/cuda/include:$CPATH
ENV LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID
