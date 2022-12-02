# Blockers for Group Project

## Not able to use the camer inside `docker`

- Possible Problems: Docker not recognizing camera as it has no access to hardware.

- Possible solutions: 
1. While running Docker image attach the camera module to it.

```
docker run -d -p 5000:5000 --privileged -v /dev/video0:/dev/video0 image-name
```

This didn't worked for us.

2. Another solution

Inside python file:
```
import cv2


WINDOW_NAME = "Opencv Webcam"

def run():

    cv2.namedWindow(WINDOW_NAME)

    video_capture = cv2.VideoCapture(0)

    while True:
        ret, frame = video_capture.read()
        print(ret, frame.shape)

        cv2.imshow(WINDOW_NAME, frame)
        cv2.waitKey(1)

if __name__ == "__main__":
    run()
```
- `Dockerfile`
```
ENV TERM=xterm
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
    libopencv-dev \
    python3-opencv \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR /app
ADD . /app

ENV PYTHONUNBUFFERED=1
ENV PYTHONPATH=/app
```
- Run Command
```
docker build -t opencv-webcam .
docker run -it -v $PWD:/app/ --device=/dev/video0:/dev/video0 -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=$DISPLAY opencv-webcam bash
```
### The above solution doesn't seem to work. Tried using `javascript` for camera

- HTML Code
```
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <link rel="stylesheet" href="style.css" />
  <title>Camera Page</title>
</head>

<body>
  <button id="start-camera">Start Camera</button>
<video id="video" width="320" height="240" autoplay></video>
<button id="start-record">Start Recording</button>
<button id="stop-record">Stop Recording</button>
<a id="download-video" download="test.webm">Download Video</a>
</body>

</html>
```
- JavaScript Code
```
let camera_button = document.querySelector("#start-camera");
let video = document.querySelector("#video");
let start_button = document.querySelector("#start-record");
let stop_button = document.querySelector("#stop-record");
let download_link = document.querySelector("#download-video");

let camera_stream = null;
let media_recorder = null;
let blobs_recorded = [];

camera_button.addEventListener('click', async function() {
   	camera_stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
	video.srcObject = camera_stream;
});

start_button.addEventListener('click', function() {
    // set MIME type of recording as video/webm
    media_recorder = new MediaRecorder(camera_stream, { mimeType: 'video/webm' });

    // event : new recorded video blob available 
    media_recorder.addEventListener('dataavailable', function(e) {
		blobs_recorded.push(e.data);
    });

    // event : recording stopped & all blobs sent
    media_recorder.addEventListener('stop', function() {
    	// create local object URL from the recorded video blobs
    	let video_local = URL.createObjectURL(new Blob(blobs_recorded, { type: 'video/webm' }));
    	download_link.href = video_local;
    });

    // start recording with each recorded blob having 1 second video
    media_recorder.start(1000);
});

stop_button.addEventListener('click', function() {
	media_recorder.stop(); 
});
```
### Blocker
- With this approach we were able to store the video file as a `blob` in a variable. But we need to send that variable to python file.

- Possible solution, Not implemented yet.

- JavaScript file
```
$.ajax({
    type: "POST",
    url: "{{ url_for("get_post_json") }}",
    contentType: "application/json",
    data: JSON.stringify({hello: "world"}), // replace this with our variable
    dataType: "json",
    success: function(response) {
        console.log(response);
    },
    error: function(err) {
        console.log(err);
    }
});
```
- Python File
```
@app.route('/_get_post_json/', methods=['POST'])
def get_post_json():    
    data = request.get_json()

    return jsonify(status="success", data=data)
```
