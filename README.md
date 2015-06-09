# doorstate
Is my door locked?

You will need:

1. raspberry pi or other computer you can just leave around
2. cheap webcam
3. the `phashion` library.

I came into this project with a vague concept and no knowledge of image processing or daemonization.  All I had was an annoying habit of not remembering locking my door(I did generally lock it, just didn't remember) and a pi sitting around.

Problem Domain: I want to know whether my doorknob is locked or not without having to go back and check.  I mounted a webcam about five feet away from the knob, at the same height.  The basic difference is that when the door is locked, the knob is vertical, when it's unlocked, it's horizontal.  Complications arise from the varied lighting conditions and the low resolution of the webcam.

Overall Design:  I have one file named "locked.jpg" which represents the state of the door when it's locked.  Every second, a daemon will compare it to a picture newly scraped from the camera, and post the state of the door to a private webpage.  Whenever I leave my apartment wifi, my phone will ping that site and send me a notification of the state of my door.  I'll probably set that last bit up with ifttt.

Step one: find some way to do image comparisons.  I find image manipulation boring, so I decided to look for a library to do the comparison for me.  This is what I ended up with: https://www.amberbit.com/blog/2013/12/20/similar-images-detection-in-ruby-with-phash/. Seemed simple enough.  I had initially considered edge detection, but found this first.

Step two: find some way to programmatically take a picture from the webcam.  This is where I failed bigtime.  I was hoping to do this whole thing in pure ruby, but no dice - couldn't find a way to take a picture in ruby using the webcam without doing an `exec()`.  I ended up with a nice program called fswebcam that lets you specify all sorts of things, like resolution, cropping, and so on.

Step three: Tie the two together.  If I couldn't do it the ruby way, I wanted to do it the unix way - `fswebcam | compare_image.rb`.  Sadly, `phashion` only allows you to specify filenames, not actual image files.  So on my friend's recommendation, I used named pipes instead.  `mkfifo curr_state.jpg`.  I was worried that if `fswebcam` somehow ran more often than `compare_image`, the pipe would get clogged, but it does limit itself to one write per read and anyway it sends an EOF.  So now the command is `fswebcam --no-banner -r 640x480 --crop 50x50 - >curr_state.jpg & ./compare_image.rb`.
