###########################
# 
# Silva/Dynamical: 1.1
# 
# 3/5/2020
#
#

ARG BASE_CONTAINER=jupyter/scipy-notebook
FROM $BASE_CONTAINER


USER root

#### Some custom code and install requirements.

RUN apt-get update && \
    apt-get install -y htop && \
    apt-get install -y pciutils && \ 
    apt-get install -y lm-sensors && \
    rm -rf /var/lib/apt/lists/*

#
#
##End custom code


# Install some goodies


USER $NB_UID

RUN conda install -c conda-forge jupyter_contrib_nbextensions --quiet --yes     && \
    conda install -c conda-forge 'jupyterlab-nvdashboard' 'pynvml' 'ipympl=0.5.3' --quiet --yes && \
    conda clean --all -f -y  && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER && \
    jupyter labextension install jupyterlab-nvdashboard --no-build && \
    jupyter labextension install @krassowski/jupyterlab_go_to_definition@^0.7.1 --no-build && \
    jupyter labextension install @jupyterlab/toc@1.0.1 --no-build && \
    jupyter labextension install jupyter-matplotlib --no-build && \
    jupyter labextension update --all && \
    jupyter lab build && \
    npm cache clean --force && \
    rm -rf $CONDA_DIR/share/jupyter/lab/staging && \
    rm -rf /home/$NB_USER/.cache/yarn && \
    rm -rf /home/$NB_USER/.node-gyp && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER


# Install Dynamical model environment

USER root

COPY environment.yml /opt/app/environment.yml
WORKDIR /opt/app
RUN conda env update --file environment.yml
COPY . /opt/app
WORKDIR /home/jovyan

# Switch back to jovyan to avoid accidental container runs as root
USER $NB_UID
