I use digitalocean for my video streaming servers, https://m.do.co/c/23f41a57675c  1st signup and create a ubuntu 64 bit 14.04 droplet. The droplet the $5/mo
You will need putty (How-to) and to learn a bit about Linux after you successful ssh to your server then this will get you up and running.
**Note cellular service may affect the streaming since the go app uses a fixed rate you have to have good strong coverage since the go app streams a a fixed bitrate!(I'll work on a local converter using a raspberry pi to maybe be a adaptive bitrate for lower qualilty streaming sorta the man in the middle concept. ((DJI SDK Team if you could support linux on the P3 that would be great!))

Next:
Installing Nginx and Nginx-RTMP

Install the tools required to compile Nginx and Nginx-RTMP from source.

sudo apt-get install build-essential libpcre3 libpcre3-dev libssl-dev
Make a working directory and switch to it.

mkdir ~/working
cd ~/working
Download the Nginx and Nginx-RTMP source.

wget http://nginx.org/download/nginx-1.7.5.tar.gz
wget https://github.com/arut/nginx-rtmp-module/archive/master.zip
Install the Unzip package.

sudo apt-get install unzip
Extract the Nginx and Nginx-RTMP source.

tar -zxvf nginx-1.7.5.tar.gz
unzip master.zip
Switch to the Nginx directory.

cd nginx-1.7.5
Add modules that Nginx will be compiled with. Nginx-RTMP is included.

./configure --with-http_ssl_module --add-module=../nginx-rtmp-module-master
Compile and install Nginx with Nginx-RTMP.

make
sudo make install
Install the Nginx init scripts.

sudo wget https://raw.github.com/JasonGiedymin/nginx-init-ubuntu/master/nginx -O /etc/init.d/nginx
sudo chmod +x /etc/init.d/nginx
sudo update-rc.d nginx defaults
Start and stop Nginx to generate configuration files.

sudo service nginx start
sudo service nginx stop
Installing FFmpeg

Add the FFmpeg PPA.

sudo apt-get install software-properties-common
sudo add-apt-repository ppa:kirillshkrogalev/ffmpeg-next
Update the package lists.

sudo apt-get update
Install FFmpeg.

sudo apt-get install ffmpeg
Note: The apt-add-repository command may not be installed in some cases. To install it run sudo apt-get install software-properties-common.
Configuring Nginx-RTMP and FFmpeg

Open the Nginx configuration file.

sudo nano /usr/local/nginx/conf/nginx.conf
Append the following.

rtmp {
    server {
            listen 1935;
            chunk_size 8192;

            application live {
                    live on;
                    record off;
                    exec ffmpeg -i rtmp://localhost/live/$name -threads 1 -c:v libx264 -profile:v baseline -b:v 6000K -s 1920x1200 -f flv -c:a aac -ac 1 -strict -2 -b:a 56k rtmp://localhost/live360p/$name;
            }
            application live360p {
                    live on;
                    record off;
        }
    }
}

After you've added the above, you can customize settings such a video bitrate, audio bitrate and resolution. These changes will only be applied to the lower quality stream. To add more qualities, copy and paste the exec ffmpeg line and change the settings. You'll also need to create a new application. You can do this by copying and pasting the live360 example that has been included. Don't forget to update the exec ffmpeg line with the address of the new application. You can do this by changing the final RTMP address in the exec ffmpeg line.

Note: Changing the value after -b:v will change the video bitrate. This is measured in kilobits per second. Changing the value after -b:a will change the audio bitrate. This is measured in kilobits per second. Changing the value after -s will change the resolution.
Save the file by pressing Control and X together. Restart Nginx.

sudo service nginx restart
Note: For best performance, each stream being converted should have its own CPU core. For example two qualities, 360P and 480P are being created from a 720P stream. A Vultr instance with at least two CPU cores should be used.
Security Note

If you're using a firewall, you'll need to make sure TCP 1935 is allowed.

The current configuration allows anyone to stream to your server. We can fix this by only allowing certain IP addresses the publish permission. So you might want to shut your server down when your not using it.


Finally in the app set your cutom rtmp stream to rtmp://yourdropletip/live/mydjidrone and it should be streaming you will need a custom player and a few more tweaks after that.
I've had really good luck on digital ocean. I run my storm chasing site and portal off ot them. 
