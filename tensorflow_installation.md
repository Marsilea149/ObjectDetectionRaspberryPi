# Installation with conda environment
I followed the official link to install tensorflow: https://www.tensorflow.org/install/pip#step-by-step_instructions

I first tried to install only with pip, without conda environment, but it was not working. Because I have no installation of 'cudatoolkit'. To not mess up my ubuntu local installation, I prefered to create a conda environment and install everything inside. 

Cmd to follow: 
```
 cd ~/rpi_projects/
 conda create --name tf python=3.9
 conda deactivate
 conda activate tf
 nvidia-smi
 conda install -c conda-forge cudatoolkit=11.8.0
 pip install nvidia-cudnn-cu11==8.6.0.163
 CUDNN_PATH=$(dirname $(python -c "import nvidia.cudnn;print(nvidia.cudnn.__file__)"))
 export LD_LIBRARY_PATH=$CUDNN_PATH/lib:$CONDA_PREFIX/lib/:$LD_LIBRARY_PATH
 mkdir -p $CONDA_PREFIX/etc/conda/activate.d
 echo 'CUDNN_PATH=$(dirname $(python -c "import nvidia.cudnn;print(nvidia.cudnn.__file__)"))' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh
 echo 'export LD_LIBRARY_PATH=$CUDNN_PATH/lib:$CONDA_PREFIX/lib/:$LD_LIBRARY_PATH' >> $CONDA_PREFIX/etc/conda/activate.d/env_vars.sh
 pip install --upgrade pip
 pip install tensorflow==2.13.*
 ```
 Now, the conda environment has been installed, you can check your installation with: 
 ```
 #If a tensor is returned, you've installed TensorFlow successfully.
 python3 -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
 #If a list of GPU devices is returned, you've installed TensorFlow successfully.
 python3 -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
```



# Installation using Docker
Another to do this is using Docker, you can follow this link: https://www.tensorflow.org/install/docker 

I already did: 
```
 1997  docker pull tensorflow/tensorflow:latest-gpu-jupyter 
 1998  docker run -it --rm tensorflow/tensorflow    python -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
 1999  lspci | grep -i nvidia
 2000  docker run --gpus all --rm nvidia/cuda nvidia-smi
 2001  docker -v
```
But it seems that I don't have nvidia-docker, see more: https://github.com/NVIDIA/nvidia-docker