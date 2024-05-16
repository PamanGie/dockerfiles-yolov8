How to Run?

# Install xhost agar yolov8 jalan di docker
# buka terminal, jalankankan perintah ini

xhost +local:docker

# lalu gunakan perintah ini untuk menjalankan docker

docker run --gpus all --rm -it --env="DISPLAY" --env="QT_X11_NO_MITSHM=1" --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" my_yolov8_image
