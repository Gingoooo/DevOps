# 使用 nvidia 作為基礎image
FROM nvidia/cuda:11.5.1-cudnn8-runtime-ubuntu20.04

# 定義變量
ARG PYTHON_VERSION=3.9.7
ARG UNAME=dsl_user
ARG UID=40002
ARG GID=40002


# 創建一個新的用戶組
RUN groupadd -g $GID -o $UNAME
# 創建一個新用戶，並將其添加到新的用戶組
RUN useradd -m -u $UID -g $GID -o -s /bin/bash $UNAME
# 更新系統並安裝 wget
RUN /usr/bin/apt-get update && /usr/bin/apt-get install -y wget

# 設置環境變量
ENV CONDA_DIR=/opt/conda
ENV PATH="/opt/conda/bin:${PATH}"
ENV PYTHONPATH="./.local/lib/python3.9/site-packages"

# 安裝 Mambaforge（一個輕量級的 conda 替代品）
RUN miniforge_arch=$(uname -m) && \
    miniforge_installer="Mambaforge-Linux-${miniforge_arch}.sh" && \
    wget --quiet "https://github.com/conda-forge/miniforge/releases/latest/download/${miniforge_installer}" && \
    /bin/bash "${miniforge_installer}" -f -b -p "${CONDA_DIR}" && \
    /bin/rm -f "${miniforge_installer}" && \
    # 配置 conda 和 mamba
    ${CONDA_DIR}/bin/conda config --system --set auto_update_conda false && \
    ${CONDA_DIR}/bin/conda config --system --set show_channel_urls true && \
    ${CONDA_DIR}/bin/mamba install --quiet --yes python="${PYTHON_VERSION}"&& \
    ${CONDA_DIR}/bin/mamba list python | grep '^python ' | tr -s ' ' | cut -d ' ' -f 1,2 >> "${CONDA_DIR}/conda-meta/pinned" && \
    ${CONDA_DIR}/bin/conda update --all --quiet --yes && \
    # 清理 conda 缓存
    ${CONDA_DIR}/bin/conda clean --all -f -y

RUN /opt/conda/bin/pip3 install jupyterlab==1.2.21 --no-cache-dir
RUN /opt/conda/bin/pip3 install tensorflow-gpu --no-cache-dir
# 創建 Python 和 Jupyter 的符號鏈接
RUN ln -s /opt/conda/bin/python3 /bin/python3 && ln -s /opt/conda/bin/jupyter /bin/jupyter
# 創建 pip3 和 pip 的符號鏈接
RUN ln -s /opt/conda/bin/pip3 /bin/pip3 && ln -s /opt/conda/bin/pip /bin/pip

USER $UNAME

WORKDIR /home/$UNAME
CMD ["/opt/conda/bin/jupyter", "lab", "--ip" , "0.0.0.0", "--no-browser", "--NotebookApp.token=''"]