# CryptoStego  
JS library for steganography with encryption - Hide text in an image with encryption and obfuscation. Support least significant bit mode and DCT mode.    
  
## Version  
v1.5  
  
## DEMO  
[http://stego.js.org](http://stego.js.org)  
**Note: Library needs HTML5 support!**  
   
## Download  
Download [cryptostego.min.js](https://github.com/zeruniverse/CryptoStego/releases/download/v1.5/cryptostego.min.js)  
**Note: This JS library needs HTML5 support!**  
  
## Features  
+ Obfuscation - Random initialization of invalid bits  
+ Non-linear bit-by-bit message storage
+ Valid bits and their order decided by SHA512-based hash function  
+ Password decides the parameter in hash function. Different passwords map message to different locations in the image  
+ No signal for password error. Wrong password results in wrong message
+ LSB (Least Significant Bit) mode
  + Use least significant bits of RGB channels of each pixel to store message  
  + Resulting image is visually identical to original one  
  + Can only be stored in non-compressed format such as PNG  
+ DCT (Discrete cosine transform) mode
  + Store information by slightly changing lowest frequency component of each block in frequency domain  
  + Robust to image compression but stores less data compared to LSB mode  
  + Resulting image looks different from original one  
  
## Usage  
This library provides 3 functions. Use `<script src="cryptostego.min.js"></script>` in your HTML to include this library.  
### `loadIMGtoCanvas(inputid, canvasid, callback, maxsize)`  
This function loads an image from file input to a dynamically generated canvas. After that, it will call `callback()` function to do some stuff, and then delete the generated canvas.
  + `inputid` is the id of the html5 file input.
    + You need to put a file input element like `<input type="file" id="file" accept="image/*" />` in your HTML and ask user to select an image here.  
  + `canvasid` is the id of the canvas that this function will generate. **`canvasid` will be generated by this function dynamically when called.** You will use this `canvasid` in `callback()` to locate the canvas.    
    + **You must use an id that not currently used in your HTML**, When this function is called, a canvas with id to be `canvasid` will be created. Then, the image selected by user will be loaded to this canvas.  
    + **Canvas created by this function (`canvasid`) will be deleted after finishing calling callback function.** So you might want to copy image in this canvas to another canvas in callback function or trig a download in callback function.  
  + `callback` is the callback function that will be called after image successfully loaded to canvas.
    + Use `canvasid` to locate the canvas in `callback()` function.  
    + `canvasid` will be deleted after calling `callback()`. Make sure you store all data needed in `callback` function.  
    + An example for callback -> download the result image after steganography:
    ``` JavaScript
    //callback function is writefunc()
    function writefunc(){
        if(writeMsgToCanvas('canvas',$("#msg").val(),$("#pass").val(),3)!=null){ 
        var myCanvas = document.getElementById("canvas"); //canvasid='canvas'  
        var image = myCanvas.toDataURL("image/jpeg",1.0);    
        var element = document.createElement('a');
        element.setAttribute('href', image);
        element.setAttribute('download', 'result.jpg');
        element.style.display = 'none';
        document.body.appendChild(element);
        element.click();
        document.body.removeChild(element);        
        }
    }
    //For convenience, jQuery is used here. But that's not necessary.
    loadIMGtoCanvas('file','canvas',writefunc,500);
    ```
  + `maxsize` is the max width or height for created canvas (default value is 0)  
    + If either image width or image height larger than `maxsize`, This function scale the image so that it fits the generated canvas with width and height not larger than `maxsize`  
    + If `maxsize`<=0, image will not be scaled (use original size). This is not recommended if your callback function is steganography function (write info to image). As super large image will make browser dead. A recommended value is 500.  
    + Make sure when callback function is reading info from image, your `maxsize` is at least the `maxsize` you use for steganography. As steganography algorithm is not robust to scale.    
  
### `writeMsgToCanvas(canvasid,msg,pass,mode)`  
This function writes your message into image in canvas `canvasid`. Before calling this function, make sure some image is loaded in `canvasid`. Usually, this function will be called in callback function of `loadIMGtoCanvas`  
+ `canvasid` specifies the id of the canvas to whose image the message will be written to.  
+ `msg` specifies the message.  
+ `pass` specifies the password for retrieving the message. (default value is '')  
+ `mode` an integer specifies the steganography mode. [0-5]   
  + `0` -> LSB mode (default), result image looks identical to original image.  
  + `1` - `5` -> DCT mode, higher value means better robustness to compression but the image looks more different from the original one.  
  + Generally, if you don't need image compression robustness, use mode `0`, otherwise, mode `2` and `3` are recommended.
  
+ Return value is either `null` or `1`, `null` stands for `fail` and `1` stands for `success`.  
  
### `readMsgFromCanvas(canvasid,pass,mode)`  
This function reads your message from image in canvas `canvasid`. Before calling this function, make sure some image is loaded in `canvasid`. Usually, this function will be called in callback function of `loadIMGtoCanvas`  
+ **To successfully read message, `pass` and `mode` should be same as what you use in `writeMsgToCanvas`. And the image in `canvasid` should be the result image of `writeMsgToCanvas`.**  
+ All parameters have same meaning as in `writeMsgToCanvas`.  
+ Return value is a string or `null`  
  + `null` means fail to read message. It might caused by wrong password, wrong image or wrong parameters.  
  + returned string is the message retrieved. It might be some meaningless characters on error.  
  
## Advance Usage  

`writeMsgToCanvas` and `readMsgFromCanvas` functions can be replaced by `writeMsgToCanvas_single(canvasid,msg,pass,dct,copy,lim)` and `readMsgFromCanvas_single(canvasid,pass,dct,copy,lim)`,
which have the same effects but you can specify more parameters. `dct` is a boolean meaning whether DCT mode should be used. If `dct=true`, `copy` and `lim` will be considered by these two functions.

`copy` means how many copies of this message should be stored. It's usually an odd integer so that as long as more than half of the data is correct, the result is correct. A high `copy` value means
higher robustness but less data capacity.

`lim` means how much difference between a bit '0' and a bit '1' should reflect on the image. Higher `lim` provides better robustness but the image will look more different from the original one.

Refer to `main.js` for examples setting `copy` and `lim` parameters.

## Compression Robustness for DCT  

### Raw image and data  
Image before steganography (268KB in PNG format):  
![dandelionclock](https://cloud.githubusercontent.com/assets/4648756/15265727/6b29773e-1941-11e6-9245-3275ff0afcf2.jpg)  

Data:  
```
你好，世界！
HELLO WORLD!
¡HOLA MUNDO!
مرحبا بالعالم!
BONJOUR LE MONDE!
こんにちは世界！
ПРИВЕТ МИР!
```
  
### Compression Ratio 9.4% (25.2KB)  
![result 3](https://user-images.githubusercontent.com/4648756/30573644-0e8ffc94-9cc3-11e7-8157-f807db294a51.jpg)  
Retrieved data correct for mode 2,3,4,5!  
  
### Compression Ratio 4.2% (11.2KB)  
![optimized-result 3](https://user-images.githubusercontent.com/4648756/30573739-8d233670-9cc3-11e7-854a-0834869524e3.jpg)  
Result for mode 3:
```
你好，乖界！
HELLO WORLD!
¡HOLA MUNDO!
مرحبا بالعالم!
BONJOUR LE0MONDE!
こんにちは世界！
ПРИВЕТ МИР! 
```

Data retrieved correctly for mode 5 as compression ratio 4.2% 
 
### Compression Ratio 1.5% (4.10KB)  
![result 3 1](https://user-images.githubusercontent.com/4648756/30573930-a1495390-9cc4-11e7-9ac6-f0746898711c.jpg)  
Result for mode 5:  
```
你好，世畄！
HELLO WORLDኂሏLA MUNDK!
مرحبا باɄعالم!BONJOUR LE MON@G 
こんにちは世畈︁
ПРИВՐ⠐쐘Р!
```  

  
## Coding Example  
See `example/` folder   
  
## Copyright  
Jeffery Zhao  
License: GNU **A**GPL v3.0 or later (MIT License allowed for non-commercial purposes)  
The copyright for Crypto-JS is reserved by its authors.  
