	 	 	 	
Membuat Docker di Jetson untuk YOLOv8

(1)Buat Environment (Opsional)
Buka terminal dan masukkan perintah
sudo apt install python3-virtualenv
Tulis nama environment yang dinginkan misal jetson
virtualenv jetson
Aktifkan environment anda dengan perintah beriku
source jetson/bin/activate 
Jika ingin nonaktifkan virtual environment, ketik perintah
deactivate
(2)Instalasi depedensi YOLOv8 (Opsional)
Tahapan ini opsional, jika anda ingin menguji YOLOv8 anda di Jetson sebelum diisolasi, anda bisa menggunakan langkah ini. Namun jika anda yakin model YOLO anda sudah berjalan dengan baik, anda bisa langsung lompat ke tahap (3). 
Karena pada nantinya kita akan menggunakan instalasi depedensi dari dockerfiles yang sudah disiapkan di github ultralytics untuk jetson, kita hanya perlu memberikan sedikit konfigurasi kecil. 
Untuk Panduan lengkap instalasi lingkungan non-Docker YOLOv8 di Jetson, bisa anda baca disini:
https://docs.ultralytics.com/guides/nvidia-jetson/#install-onnxruntime-gpu 
Berikut cara yang saya lakukan agar lingkungan YOLOv8 berjalan optimal di Jetson.
Persiapan Pustaka, Package dan Pip
Update semua package dan pip
sudo apt update
sudo apt install python3-pip -y
pip install -U pip
Install pustaka ultralytics 
pip install ultralytics[export]
Reboot Jetson anda
sudo reboot
Install Pytorch dan Torchvision


Jika anda pernah menginstall pytorch dan torchvision menggunakan pip / pip3, melalui pustaka di pytorch website atau pypi, silahkan uninstall dulu dengan perintah.
pip uninstall torch torchvision
Install PyTorch 2.1.0 berdasarkan JP5.1.3 (masukkan perintah satu persatu)
sudo apt-get install -y libopenblas-base libopenmpi-dev
wget https://developer.download.nvidia.com/compute/redist/jp/v512/pytorch/torch-2.1.0a0+41361538.nv23.06-cp38-cp38-linux_aarch64.whl -O torch-2.1.0a0+41361538.nv23.06-cp38-cp38-linux_aarch64.whl
pip install torch-2.1.0a0+41361538.nv23.06-cp38-cp38-linux_aarch64.whl
Install Torchvision v0.16.2 berdasarkan PyTorch v2.1.0  (masukkan perintah satu persatu)
sudo apt install -y libjpeg-dev zlib1g-dev
git clone https://github.com/pytorch/vision torchvision
cd torchvision
git checkout v0.16.2
python3 setup.py install --user
Catatan:
Jika anda menginstal tensorflow, maka nanti akan ada warning incompatible numpy requirements. Jika anda tidak membutuhkan tensorflow anda bisa hapus saja dari environment anda, dan anda bisa membuat environment baru yang terpisah untuk tensorflow. 
Install onnxruntime-gpu (Opsional jika mau konversi ke Tensorrt)
wget https://nvidia.box.com/shared/static/zostg6agm00fb6t5uisw51qi6kpcuwzd.whl -O onnxruntime_gpu-1.17.0-cp38-cp38-linux_aarch64.whl
pip install onnxruntime_gpu-1.17.0-cp38-cp38-linux_aarch64.whl

(3)Testing YOLOv8 di Jetson
Selanjutnya anda bisa menguji model YOLOv8 anda di Jetson, dengan menggunakan perintah sebagai berikut:
python3 nama_model.py
jika saat menjalankan model, terjadi error seperti berikut:
error error: (-2:Unspecified error) The function is not implemented. Rebuild the library with Windows, GTK+ 2.x or Cocoa support
Silahkan jalankan perintah ini di terminal:
pip uninstall opencv-python-headless -y
pip install opencv-python --upgrade

Gambar 1. YOLO di Jetson

(4)Membuat Docker untuk YOLOv8 di Jetson
Pada tahapan ini saya sudah menganggap anda sudah memiliki model YOLOv8 yang sudah dioptimasi dan berjalan lancar di Jetson. Pastikan semua depedensi yang anda butuhkan sudah terinstall semua, karena ketika sudah diisolasi menjadi image, maka akan sulit untuk menambah depedensi lagi. 
Hapus Gnupg
Gnupg dihapus karena akan menyebabkan instalasi docker menjadi sangat kompleks, sehingga karena elemen ini tidak terlalu penting, maka akan kita pindahkan saja. 
Izinkan akses ke gnupg dengan perintah: chmod -R +r .gnupg
ketikkan perintah: nano .dockerignore
tambahkan baris ini: .gnupg
lalu tekan ctrl+0 untuk simpan
	Catatan:
	Jika belum ada text editor nano, silahkan install dulu dengan perintah
sudo apt update
sudo apt install nano
nano --version


B. Mengunduh dan mengkonfigurasi dockerfiles
B.1 Mempersiapkan Model YOLOv8 
Pada tahapan ini saya menganggap anda menyimpan model anda beserta dataset label, gambar, video, dan depedensi lainnya didalam folder khusus. saya anggap anda simpan model anda di folder /home/yolov8 
Tentunya anda bisa menggunakan nama folder lain, namun saya rekomendasikan disimpan didalam folder /home/ agar mudah diatur jalur salinannya di dockerfiles.
B.2 Mengunduh dockerfiles Jetson Nano 
Untuk mendapatkan dockerfiles yang sudah diisolasi oleh ultralytics untuk lingkungan jeton, kita bisa ambil dari repositori mereka di github, dengan perintah dibawah ini:
wget https://raw.githubusercontent.com/ultralytics/ultralytics/main/docker/Dockerfile-jetson
Selanjutnya anda bisa buka dockerfiles tersebut melalui folder /home dan dibuka dengan VSCode atau texeditor atau bisa juga dibuka di terminal dengan perintah nano dockerfiles
tambahkan baris dibawah ini di dockerfiles, diatas baris #usage examples
# Copy the yolov8 folder into the container
COPY yolov8 /usr/src/ultralytics/yolov8
# Usage Examples --------------------------

yolov8 adalah nama folder yang saya buat, anda bisa gunakan folder anda sendiri


Jika anda membutuhkan pip depedensi lain, misal pustaka speech recognition anda bisa tambahkan baris perintahnya disini. tulisa diatas # Set environment variables
# Install SpeechRecognition library
RUN pip install SpeechRecognition

Tentunya anda bisa menambahkan pustaka lain dengan tambahan RUN
Jika sudah dikonfigurasi silahkan disimpan dengan nama dockerfiles tanpa ekstensi apapun.
untuk contoh dockerfiles yang saya gunakan, anda bisa cek di github saya disini
https://github.com/PamanGie/dockerfiles-yolov8   
B.3 Membuat image docker
Selanjutnya kita akan membuat image dockernya, perintahnya adalah 
docker build -t nama_docker -f dockerfiles .
contohnya
docker build -t yolov8_docker -f dockerfiles .
jika sudah selesai maka akan ada tulisan:
=> => naming to docker.io/library/yolov8_docker 

Gambar 2. Image Exporting
B.4 Menjalankan Image dan Penanganan Error 
Untuk menjalankan image docker yang sudah kita buat, maka perintahnya adalah
docker run -u root -it yolov8_docker
Jika anda cek maka anda akan melihat folder anda tersalin ke image, contoh ada folder yolov8 yang berisi model saya. 

Gambar 3. Docker Environment
namun, masalah muncul ketika model dijalankan didalam docker, maka akan terjadi error karena GUI windows tidak terbuka atau menolak untuk dibuka.
FPS: 0.015099930570554488
Unable to init server: Could not connect: Connection refused
Traceback (most recent call last):
  File "video.py", line 114, in <module>
	cv2.imshow('ObjectDetection', frame)
cv2.error: OpenCV(4.5.0) /opt/opencv/modules/highgui/src/window_gtk.cpp:624: error: (-2:Unspecified error) Can't initialize GTK backend in function 'cvInitSystem'

Gambar 4. Error Model
Pesan kesalahan diatas karena ada masalah saat mencoba menginisialisasi backend GTK untuk tampilan gambar dengan OpenCV. Ini bisa terjadi karena lingkungan Docker tidak memiliki akses ke server X Window System atau tidak memiliki libgtk yang diperlukan untuk inisialisasi GUI.
untuk mengatasi ini keluar dari docker dengan mengetik  exit
lalu kita perlu mengikatkan display X11 host ke container untuk memastikan kita memiliki akses yang sesuai ke server X. gunakan perintah ini
xhost +local:docker
ketika sudah muncul pesan
non-network local connections being added to access control list
maka selanjutnya jalankan perintah ini
docker run --gpus all --rm -it --env="DISPLAY" --env="QT_X11_NO_MITSHM=1" --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" nama_image_docker_anda
contoh nama image saya adalah yolov8_docker
docker run --gpus all --rm -it --env="DISPLAY" --env="QT_X11_NO_MITSHM=1" --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" yolov8_docker
sekarang ketika sudah masuk image docker kembali, anda bisa cek lagi dengan masuk ke folder model anda, dan lihat seharusnya model anda sudah berjalan didalam image docker anda

Gambar 5. YOLOv8 on Docker

B.5 Konversi ke Image.tar
agar docker anda bisa dipindahkan ke Jetson lain, maka kita bisa jadikan .tar file yang nanti bisa diload di jetson yang lain. berikut perintahnya
docker save -o nama_docker_anda.tar nama_image_docker_anda
misal saya akan menyimpan dengan nama yolov8-docker .tar maka perintahnya adalah
docker save -o yolov8-docker.tar yolov8-docker
tunggu beberapa menit, sampai terminal kembali ke awal. 

Gambar 6. YOLOv8 tar Image
Jika sudah jadi image .tar, anda bisa bawa docker tersebut ke jetson lain. untuk melakukan load  image .tar tersebut di jetson lain  maka perintahnya adalah
docker load -i nama_docker.tar
contoh
docker load -i yolov8-docker.tar
lalu gunakan perintah ini kembali untuk menjalankan docker dan model.
xhost +local:docker
ketika sudah muncul pesan
non-network local connections being added to access control list
maka selanjutnya jalankan perintah ini
docker run --gpus all --rm -it --env="DISPLAY" --env="QT_X11_NO_MITSHM=1" --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" nama_image_docker_anda
contoh nama image saya adalah yolov8_docker
docker run --gpus all --rm -it --env="DISPLAY" --env="QT_X11_NO_MITSHM=1" --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" yolov8_docker

(5)Melihat list docker dan menghapusnya
melihat semua images
docker images


hapus docker
docker rmi -f yolov8-docker















