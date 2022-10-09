## first time set up docker container
`docker run --gpus all -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=$DISPLAY -v $PWD:/root/testground --name ds6devel -it nvcr.io/nvidia/deepstream:6.1.1-devel`

add to .bashrc:
export GST_PLUGIN_PATH=/usr/lib/x86_64-linux-gnu/gstreamer-1.0
export CUDA_VER=11.7

## non first time run
`docker start ds6devel`
`docker exec -it ds6devel bash`


## test plugin tutorial (with gst-template tool)
follow tuto of 
https://gitlab.freedesktop.org/gstreamer/gst-template
https://gstreamer.freedesktop.org/documentation/plugin-development/basics/boiler.html?gi-language=c

### build proj
```
cd gst-template
git checkout d80bfac  # after this commit requires gstream v1.19, we have v1.16
git switch -c v1d16
meson build
ninja -c build
```

### gen plugin skeleton
```
../tools/make_element gstmyfilter gstplugin
```


## test plugin tutorial (with gst-element-maker)
https://developer.ridgerun.com/wiki/index.php/Creating_a_New_GStreamer_Element_or_Application_Using_Templates

### setup helper tools
```
git clone https://github.com/GStreamer/gst-plugins-bad.git
cd gst-plugins-bad/
git checkout 1.16

NOCONFIGURE=1 ./autogen.sh
sudo cp common/gst-indent /usr/local/bin/

apt install indent
```

### create new project to hold the new element
```
cd gst-plugins-bad/tools/

./gst-project-maker myfilter  #creates the project gst-myfilter

# move gst-my_project to any prefered location
mv gst-myfilter ~/testground

#then
cd ~/testground/gst-myfilter
./autogen.sh
make
```

### create new element, move it to the created project
for less trouble, use the project name in previous step as name of your new element i.e.
```
./gst-element-maker myfilter  #creates the element gstmyfilter
mv gstmyfilter.* ~/testground/gst-myfilter/plugins/
```


### compile and install
```
cd ~/testground/gst-myfilter
make

# in the deepstream docker (ubuntu20.04), plugins are placed at /usr/lib/x86_64-linux-gnu/gstreamer-1.0
cp plugins/.libs/libgstmyfilter.so  /usr/lib/x86_64-linux-gnu/gstreamer-1.0
```