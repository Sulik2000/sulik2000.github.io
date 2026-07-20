# Problem with Alif TinyUSB

I got task to implement streaming by UVC USB protocol video from MT9914 camera connected to E7-DK. I found two examples:
1. Which implements TinyUSB on Alif board
2. Which implements reading frames from camera: https://github.com/alifsemi/alif_M55-viewfinder

In example TinyUSB never was declares adress of enabling USB clock, USB's control register and etc. 
But for some reason code of example was compiled and succesfully ran, but when I just copy-paste code of initialization, I got error that variable which will have value of register is *undefined*.
I faced with this problem. First of all I searched all folders: all project, CMSIS cache in *HOME* folder, but name of that variable never was used. I lost on this 2 hours, and almost surrendered, but during returning to home I designed my solution.

I'll compile example of TinyUSB, which is succesfully running, and by connecting JLink Debugger I'll inspect by *ni* instruction examine all instructions to know what is address which I need. I'll try tomorrow and hope this will work.