# GSH RaspiCam

Dies ist mein worklog für unsere mobilen GSH Cams mit den Raspberry Camera v2 Modulen.

## Kamera anschließen

Das kleine Kamera-Modul wird direkt an die CSI-Schnittstelle (Camera Serial Interface) des Raspberry Pi angeschlossen.  
Die CSI-Schnittstelle des Raspberry Pi befindet sich zwischen dem HDMI-Anschluss und Audioausgang.

![Angeschlossene Raspberry Camera v2](/assets/img/hardware/raspberry_pi/gsh_raspi_cam/raspi_cam_connected.jpg)  
*&copy; [Datenreise.de](https://www.datenreise.de/)*  
*Quelle: [Datenreise.de](https://www.datenreise.de/raspberry-pi-ueberwachungskamera-livestream/)*

!!! Info "Ausrichtung CIS Kabel"
	Die **blaue Markierung** wird in **Richtung Ethernet-Anschluss** eingesteckt!

## Kamera aktivieren

Öffne das Raspberry Konfigurationsmenü:  
```shell
sudo raspi-config
```

* 5 Interfacing Options
  * P1 Camera
    * Enable the camera

!!! warning
	Den Raspberry im Anschluss neu starten!

### Kamera testen

Wir nehmen zum test ein Bild auf:  
```shell
raspistill -o test.jpg
```

## V4L Kernel mods laden und persistieren

```shell
sudo modprobe v4l2_common && \
sudo modprobe bcm2835-v4l2 && \
echo "" && \
ls /dev/video* && \
echo "If there is NO '/dev/videoX' device hit CTRL-C to abort..." && \
read -n1 -s && \
echo "v4l2_common" | sudo tee -a /etc/modules && \
echo "bcm2835-v4l2" | sudo tee -a /etc/modules && \
echo "" && \
cat /etc/modules
```

## Installiere Docker

```shell
curl -sSL https://get.docker.com | sudo sh && \
sudo usermod pi -aG docker && \
sudo reboot
```

## Starte den Streaming Docker Container

??? Info "Baue einen streaming Server Docker Container"
	!!! Warning "Achtung"
		Es ist nicht notwendig den Docker Container zu bauen!  
		Der Container ist auf DockerHub verfügbar.  
		<https://hub.docker.com/r/shokinn/gsh-streaming/>

	```shell
	DOCKER_USER_ID="shokinn" && \
	DOCKER_IMG_NAME="gsh-streaming" && \
	DOCKER_TAG="2018-10-2" && \
	mkdir -p ~/workspace/streaming; \
	cd ~/workspace/streaming/ && \
	cat << EOF > ~/workspace/streaming/Dockerfile && \
	cat << EOF > ~/workspace/streaming/entry.sh && \
	chmod +x ~/workspace/streaming/entry.sh && \
	docker build -t $DOCKER_USER_ID/$DOCKER_IMG_NAME:$DOCKER_TAG . && \
	docker image rm alexellis2/streaming:07-05-2018 && \
	echo "Press CTRL+C to avoid uploading to Docker Hub..." && \
	read -n1 -s && \
	docker login && \
	docker push $DOCKER_USER_ID/$DOCKER_IMG_NAME; \
	docker logout
	FROM alexellis2/streaming:07-05-2018
	COPY entry.sh entry.sh
	EOF
	#!/bin/bash

	echo "Width: \$1"
	echo "Height: \$2"
	echo "raspivid fps: \$3"
	echo "ffmpeg fps: \$4"
	echo "KBit/s: \$5"
	echo "URL: \$6"

	width=\$1
	height=\$2
	rfps=\$3
	ffps=\$4
	bitrate=\$(expr \$5 \* 1000)
	url=\$6

	raspivid -o - -t 0 -w \$width -h \$height -fps \$rfps -b \$bitrate -g 40 | ffmpeg -re -ar 44100 -ac 2 -acodec pcm_s16le -f s16le -ac 2 -i /dev/zero -f h264 -i pipe:0 -c:v copy -c:a aac -ab 128k -g 40 -strict experimental -f flv -r \$ffps \$url
	EOF
	```
		

??? info "Debugging"
	```shell
	read -p "Stream width: " width && \
	read -p "Stream height: " height && \
	echo "For Camera specs see here: https://picamera.readthedocs.io/en/release-1.13/fov.html#sensor-modes"; \
	read -p "raspivid FPS (40-90@720p; 1/10-30fps@1080p): " rfps && \
	read -p "ffmpeg fps: " ffps && \
	read -p "Streaming bitrate (KBit/s): " bitrate && \
	read -p "Enter Streaming Endpoint: " url && \
	docker run \
	--privileged \
	--name streaming \
	-it \
	shokinn/gsh-streaming:2018-10-2 \
	$width $height $rfps $ffps $bitrate "$url"
	```

```shell
read -p "Stream width: " width && \
read -p "Stream height: " height && \
echo "For Camera specs see here: https://picamera.readthedocs.io/en/release-1.13/fov.html#sensor-modes"; \
read -p "raspivid FPS (40-90@720p; 1/10-30fps@1080p): " rfps && \
read -p "ffmpeg fps: " ffps && \
read -p "Streaming bitrate (KBit/s): " bitrate && \
read -p "Enter Streaming Endpoint: " url && \
docker run \
--privileged \
--name streaming \
-d \
--restart=always \
shokinn/gsh-streaming:2018-10-2 \
$width $height $rfps $ffps $bitrate "$url"
```