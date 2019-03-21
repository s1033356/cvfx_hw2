# 【CV_hw2】Style Transfer Methods

## Outline
* [FastPhotoStyle](#FastPhotoStyle)
* [Neural Style](#Neural-Style)
* [Multimodel Unsupervised Image-to-Image Translation](#Multimodel-Unsupervised-Image-to-Image-Translation)
* [Image Quilting for Texture Synthesis and Transfer](#Image-Quilting-for-Texture-Synthesis-and-Transfer)
* [Conclusion](#Conclusion)
* [Appendix](#Appendix)
* [Reference](#Reference)

## FastPhotoStyle
 
FastPhotoStyle 主要是通過Stylization和Smoothing兩個步驟進行。<br>
Mapping function：![](https://i.imgur.com/DTLtWXi.png)。<br>
![](https://i.imgur.com/ECppjUR.png)
*<p align="center">Illustration of the method</p>*

### Steps:

1. Stylization transform：Y=F<sub>1</sub>(I<sub>C</sub>,I<sub>S</sub>)，將I<sub>S</sub>(style image)上的風格轉移到I<sub>C</sub>(content image)上

   * Differences between PhotoWCT (Based on WCT) and WCT
   
     1. decoder裡面用Unpooling代替Unsampling。(Unpooling層可以保留更好的局部細節。)
     2. 將特徵提取層中的pooling位置信息引入對稱的層中。
![](https://i.imgur.com/zKs6K8d.png)<br>
*<p align="center">Comparison between PhotoWCT and WCT</p>*

2. Smoothing transform : F<sub>2</sub>(Y,I<sub>C</sub>),將上一步合成的圖片做平滑處理，消除F<sub>1</sub>過程中帶來的風格不一致的問題。 

   * 局部區域相似内容的pixel有相似的内容
   * 消除與整體風格較大的偏離
   
   ![](https://i.imgur.com/HsLtbMG.png)<br>
   <br>*y<sub>i</sub>:pixel color in Y<br>
   <br>r<sub>i</sub>:pixel color in smoothed output R<br>
   <br>d<sub>ii</sub>:diagonal element in the degree matrix D of W <br>
   <br>&emsp;(W is an affinity matrix of all pixels as nodes in a graph )<br>
   <br>λ:control the balance*
   <br>![](https://i.imgur.com/u4gYQhM.png)
   <br>![](https://i.imgur.com/iP7zd1O.gif)
   <br>![](https://i.imgur.com/PiHUwPs.gif)
*<p align="center">Smoothing step</p>*


### Result

 Example 1: Without using segmentation masks

 Example 2: With manually generated semantic label maps
Label maps:```labelme```

 Example 3: With automatically generated semantic label maps
 
* Label model: pretrained Semantic Segmentation model
* Dataset: MIT ADE20K dataset


| Content Image|mual-label_con |auto-label_con |
|:-------:|:----------:|:------:|
|![](https://i.imgur.com/A4G456v.jpg)|![](https://i.imgur.com/ATAkDIk.png)|![](https://i.imgur.com/Cf5nGrr.jpg)|
|![](https://i.imgur.com/rjaENcI.jpg)|![](https://i.imgur.com/sm390OM.png)|![](https://i.imgur.com/16zNsfW.jpg)|
|![](https://i.imgur.com/MhrIEQm.jpg)|![](https://i.imgur.com/5Cs19zD.png)|![](https://i.imgur.com/EuTg38l.jpg)|

*<p align="center">comparison with labeled content images</p>*

| Style Image|mual-label_ref |auto-label_ref |
|:-------:|:----------:|:------:|
|![](https://i.imgur.com/qhowXq6.jpg)|![](https://i.imgur.com/UQBIubB.png)|![](https://i.imgur.com/VmbtLmj.jpg)|
|![](https://i.imgur.com/b2QfzAa.jpg)|![](https://i.imgur.com/IDKbvnJ.png)|![](https://i.imgur.com/nE97Ajm.jpg)|
|![](https://i.imgur.com/tt78dnL.jpg)|![](https://i.imgur.com/jdxyebA.png)|![](https://i.imgur.com/rRUJj9t.jpg)|

*<p align="center">comparison with labeled style images</p>* 

**Photo to Monet**

| ex1-result_img|ex2-result_img |ex3-result_img |
|:-------:|:----------:|:------:|
|![](https://i.imgur.com/1qfcZix.png)|![](https://i.imgur.com/x0y2vlx.png)|![](https://i.imgur.com/CqXL6bI.png)|
|![](https://i.imgur.com/63C4FUf.png)|![](https://i.imgur.com/kZk3b2i.png)|![](https://i.imgur.com/JQBIq6m.png)|
|![](https://i.imgur.com/Jwam0Az.png)|![](https://i.imgur.com/Oeinin5.png)|![](https://i.imgur.com/vp4m9bt.png)|

*<p align="center">comparison with result images(photo2Monet) </p>*

**Monet to photo**

| ex1-result_img|ex2-result_img |ex3-result_img |
|:-------:|:----------:|:------:|
|![](https://i.imgur.com/QH8AfXl.png)|![](https://i.imgur.com/8QH2d8U.png)|![](https://i.imgur.com/HzaWjWO.png)|
|![](https://i.imgur.com/ZFhn0eU.png)|![](https://i.imgur.com/8Jvz8Yl.png)|![](https://i.imgur.com/bhIWuzo.png)|
|![](https://i.imgur.com/TlTl69P.png)|![](https://i.imgur.com/DTop9J8.png)|![](https://i.imgur.com/OmvyVXb.png)|

*<p align="center"> comparison with result images(Monet2photo) </p>*

&emsp;&emsp;在生成label images的兩種方式中，自動生成label images的方法（ex3）比較偏向以色域來切割segment，當content image和style image對應segment的差異較大時，效果不如ex1和ex2。<br>
&emsp;&emsp;FastPhotoStyle效果還是略偏向於color transfer，在photo2Monet的轉換中，style transfer並不是很明顯;而在Monet2photo的轉換中，色彩轉移較寫實，效果更好。




---

## Neural Style
Leon Gatys的Neural Style Transfer的思路是通過CNN（VGG-16）分別抽取content img、painting的feature maps。然後用content img的feature maps reconstrut出目標content；用painting的feature maps reconstrut出目標的style。根據生成圖的conten與目標內容的差異來optimize content；用生成圖與目標畫style的差異來optimize style。

![](https://i.imgur.com/127PQpN.png)
*<p align="center">Convolutional Neural Network (CNN)</p>*
![](https://i.imgur.com/hJTuaKq.png)
*<p align="center">Neural Style Transfer process flow diagram (CNN)</p>*


### Steps
![](https://i.imgur.com/pyGVz9c.png)
1. Content Loss<br>
  取任意圖像和目標圖像作為CNN的input，為了使兩圖的content相似，求得其二在Convolutional layer第l層的response，最小化2-範數誤差(Content Loss)：<br>
  ![](https://i.imgur.com/hHYAn6y.png)<br>
  這一誤差可以對本層response的每一元素求導：<br>
  ![](https://i.imgur.com/hJTuaKq.png)<br>
  求導後使用back-propagation方法，利用其更新輸入的圖像，使其和目標圖像的content靠近。<br>
2. Style Loss<br>
![](https://i.imgur.com/MM4nonW.png)<br>
![](https://i.imgur.com/YE22BGA.png)<br>

3. Total Loss


![](https://i.imgur.com/BjK3W4a.png)


### Result
| Content Image|Target Image |Result |
|:-------:|:----------:|:------:|
|![](https://i.imgur.com/HKVhjer.jpg)|![](https://i.imgur.com/9kS5xvC.jpg)|![](https://i.imgur.com/0WRjyHn.png)|
|![](https://i.imgur.com/eq29Mwd.jpg)|![](https://i.imgur.com/xegIFZ3.jpg)|![](https://i.imgur.com/11iWesF.png)|
|![](https://i.imgur.com/SmDclH7.jpg)|![](https://i.imgur.com/BiZcVEN.jpg)|![](https://i.imgur.com/rwF1F0n.png)|
*<p align="center">Neural Style Representation</p>*




Leon Gatys的Style Transfer演算法結果直觀，理論簡潔在github上有各種平臺的源碼實現： 
- 基於Torch的[Neural-Style](https://github.com/jcjohnson/neural-style) 
- 基於Tensorflow的[Neural Art](https://github.com/woodrush/neural-art-tf)
- 基於Caffe的[Style Transfer](https://github.com/fzliu/style-transfer)。



---

## Multimodel Unsupervised Image to Image Translation
&emsp;&emsp;Multimodel Unsupervised Image-to-Image Translatio(MUNIT)假設圖片的latent space是由content space和style space組成，並且假設不同domain的圖片可以有相同的content space。Content代表不同domain享有的共同特徵(ex:眼睛鼻子嘴巴鬍鬚)，Style則代表不同class間的變異度(ex:家貓/石虎特徵上的不同)。在這些假設之下訓練Encoder將圖片轉換成content code和style code；Decoder根據一組content code和style code生成圖片。此外我們也能在style space中做隨機取樣以生成多張具有相同content但是不同style的圖片。
<br/><center>![](https://i.imgur.com/zNCZ3AX.jpg)</center><br/>
<br/><center>*Image can be encoded to style code and content code, and different domain might share the same content space. Encoder of domain **i** has a decoder to generate an image of domain **i** from a content code in the shared content space and a style code in the style space of domain **i***</center><br/>


### Steps
1. Train encoders `E_1` `E_2`, decoders`G_1` `G_2`, and discriminators`D_1` `D_2` to optimize the objective function
    <br/>![](https://i.imgur.com/qdONbiZ.jpg)<br/>
    which consists of image reconstruction loss, latent reconstruction loss, and adversarial loss.
    * **Image reconstruction loss**
        <br/>![](https://i.imgur.com/bZO7XQs.jpg)<br/>
        Image reconstruction loss contrains the encoders and decoders to construct image back after encoding and decoding it.
        
    * **Latent reconstruction loss**
        <br/>![](https://i.imgur.com/O6iT1S3.jpg)<br/>
        Latent reconstruction loss constrains the the consistency of the latent code in content space and style space. For example, the content code of an image `x` and the content code of a generated image `x'` should be the same.
        
    * **Adversarial loss**
        <br/>![](https://i.imgur.com/YXRAq40.jpg)<br/>
        Adversarial loss makes generators distinguish between fake images and real images and makes decoders generate fake images that are indistinguishable to the generator.
        
2. With the trained encoders, decoders`G_1` `G_2`, and images `img1` `img2`, we can apply encoders to withdraw the content code`c_1, c_2` and style code `s_1, s_2` of `img1``img2`.Then We can use `c_1` and a random style code as input to decoders`G_2` and transfer `img1` to the domain of `img2` with diversity. Alse, we can use use `c_1` and `s_2` as input to decoder`G_2` and transfer `img1` to the style of `img2`

        
### Results
**Monet to Photo**
<br/>![](https://i.imgur.com/N0TwDPl.jpg)
![](https://i.imgur.com/lKhqOL9.jpg)
![](https://i.imgur.com/l9V6rnv.jpg)
![](https://i.imgur.com/EK9Qftw.jpg)
![](https://i.imgur.com/8i0x0Na.jpg)
![](https://i.imgur.com/Y25i1Is.jpg)
![](https://i.imgur.com/nmsKiXE.jpg)<br/>
*<center> Monet to Photo.(Left) Monet painting. (Middle) Monet to photo with a fixed style. (Right) Monet to photo with random styles. </center>*

&emsp;&emsp;在content space是monet domain和photo domain享有的共同特徵這項假設之下，我們觀察生成圖片發現content code只能保有原始圖片的構圖及輪廓而缺少輪廓的semantic meaning。因此當我們使用莫內的畫的content code以及從photo的style space中取樣產生的style code生成圖片時，生成出的圖片會保有content code的結構，但是缺乏相似結構在原圖中的semantic meaning。例如：*Fig 3-1* 中生成圖片保有原圖中樹叢的結構，但是把樹叢變成山脈；前方坡地變成雲海；湖變成雲海中間的洞。

<br/>![](https://i.imgur.com/OOrxIHh.jpg)
![](https://i.imgur.com/zHlrBjn.jpg)
![](https://i.imgur.com/DNfx49B.jpg)<br/>
*<center> Fig 3-1: 生成圖片與原圖有相似的用紅線畫起來的結構. </center>*

&emsp;&emsp;除此之外，生成出的圖片整體色調以及光線很漂亮。尤其是在處理像是極光、雲海這種平滑的自然影像。

**Photo to Monet**
<br/>![](https://i.imgur.com/n2Yldh8.jpg)
![](https://i.imgur.com/vu55FyW.jpg)
![](https://i.imgur.com/T5ZhVcG.jpg)
![](https://i.imgur.com/T9ZvefT.jpg)
![](https://i.imgur.com/fcUjK4i.jpg)
![](https://i.imgur.com/wBf7Rjc.jpg)<br/>
*<center> Photo to Monet.(Left) Photo. (Middle) Photo to monet with a 
fixed style. (Right) Photo to monet with random styles. </center>*

&emsp;&emsp;我們也可以在photo to monet中觀察到前面提到保有輪廓，但是缺乏semantic meaning這項問題。然而跟從畫生成自然影像相較之下，因為是生成畫的關係，對於缺少semantice meaning這件事的就反而沒有那麼重要。

**Monet to Photo with Reference Style**

| Contents | Style 1 | Style 2 | Style 3 |
| --- | --- | --- | --- |
| *(Vertical)* content *(Horizontal)* style | ![](https://i.imgur.com/2t6Tj2h.jpg) | ![](https://i.imgur.com/cbBWqjy.jpg) | ![](https://i.imgur.com/NEhF6bt.jpg) |
| ![](https://i.imgur.com/sODB8Yl.jpg) | ![](https://i.imgur.com/QyithXG.jpg) | ![](https://i.imgur.com/obgtQbY.jpg) | ![](https://i.imgur.com/y0g9JZw.jpg) |

*<center> Monet to photo with reference styles. </center>*

| Contents | Style 1 | Style 2 | Style 3 |
| --- | --- | --- | --- |
| *(Vertical)* content *(Horizontal)* style | ![](https://i.imgur.com/28w5fQl.jpg) | ![](https://i.imgur.com/zbrVKV1.jpg) | ![](https://i.imgur.com/UnBATIv.jpg) |
| ![](https://i.imgur.com/r9uCKcO.jpg) | ![](https://i.imgur.com/rGUPnpo.jpg) | ![](https://i.imgur.com/QRWpEu8.jpg) | ![](https://i.imgur.com/0DGuZLQ.jpg) |
| ![](https://i.imgur.com/yvtv8Bt.jpg) | ![](https://i.imgur.com/OQFz3TS.jpg) | ![](https://i.imgur.com/0FpEd4I.jpg) | ![](https://i.imgur.com/21uiAaw.jpg) |
| ![](https://i.imgur.com/5clONHQ.jpg) | ![](https://i.imgur.com/aAxLngb.jpg) | ![](https://i.imgur.com/g68xMOy.jpg) | ![](https://i.imgur.com/L5vv4cM.jpg) |

*<center> Monet to photo taken by ourselves with reference styles. </center>*


| Contents | Style 1 | Style 2 | Style 3 |
| --- | --- | --- | --- |
| *(Vertical)* content *(Horizontal)* style | ![](https://i.imgur.com/hIaBtmr.jpg) | ![](https://i.imgur.com/XXmHmJk.jpg) | ![](https://i.imgur.com/flqNhVN.jpg) |
| ![](https://i.imgur.com/gX686lO.jpg) | ![](https://i.imgur.com/MAHXQVO.jpg) | ![](https://i.imgur.com/vJVAbsH.jpg) | ![](https://i.imgur.com/VeyYDhX.jpg) |
| ![](https://i.imgur.com/6s2BOvJ.jpg) | ![](https://i.imgur.com/mYDcP6Q.jpg) | ![](https://i.imgur.com/TAkob97.jpg) | ![](https://i.imgur.com/PoUBkCD.jpg) |
| ![](https://i.imgur.com/qLa9IZ6.jpg) | ![](https://i.imgur.com/b1LdcLj.jpg) | ![](https://i.imgur.com/rAnIGNU.jpg) | ![](https://i.imgur.com/zT33DKg.jpg) |

*<center> Photo taken by ourselves to monet with reference styles. </center>*

| Contents | Random Style 1 | Random Style 2 | Random Style 3 |
| --- | --- | --- | --- |
| ![](https://i.imgur.com/DQO8q6V.jpg) | ![](https://i.imgur.com/oGmEq9c.jpg) | ![](https://i.imgur.com/gHIyO6I.jpg) | ![](https://i.imgur.com/xMl7dMg.jpg) |
|![](https://i.imgur.com/6qDDbZT.jpg)|![](https://i.imgur.com/L8FOCjT.jpg)|![](https://i.imgur.com/CaPm4g8.jpg)|![](https://i.imgur.com/D0cXW2i.jpg)|
|![](https://i.imgur.com/eCHRx9t.jpg)|![](https://i.imgur.com/GyVxLfi.jpg)|![](https://i.imgur.com/c51nKoU.jpg)|![](https://i.imgur.com/Uo8XG6E.jpg)|

*<center> Photo taken by ourselves to monet with random styles. </center>*

&emsp;&emsp;上面提到的results中，style code都是從style space中隨機取樣，因此我們測試將content code與reference style image的style code結合以生成圖片。但是從result可以發現生成結果比隨機取樣差很多。轉換後的照片只能在將色調轉換到referenced image上，但是無法符合reference image所在的domain的特徵。
&emsp;&emsp;我們認為可能是由encoder產生的style code上的缺陷導致失敗的結果。從Adversarial loss以及pytorch code上發現目的是讓生成出的影像和真實的影像沒有差異的adversarial loss中都是以random sampled style code作為style code並生成影像，並沒有將encoded style code加入adversarial loss之中。這項差異可能使decoder無法根據encoded style code產生出好的結果，或是encoder無法產生好的style code。



---

## Image Quilting for Texture Synthesis and Transfer
&emsp;&emsp;Image Quilting比對target image中的patch與source texture中每一個patch的相似性，並將source texture中最相似的patch貼到生成影像上。在貼到生成影像的過程中為了降低視覺上的衝突，Image Quilting在要生成的patch與已生成區域的重複區塊中找到差異最小的路徑。找到路徑後再以路徑為基準將patch接上已生成區域。
<br/><center>![](https://i.imgur.com/OgcOv9z.jpg)</center><br/>
<center>*接合方式不同導致視覺效果上的差異。(左)沒有重疊區域 (中)有重疊區域 (右)使用差異最小的路徑*</center>

&emsp;&emsp;在Photo to Monet的例子中，我們使用使用VGG 19對`conv1_1` `conv3_1` `conv5_1`做style reconstruction以生成莫內的texture。source中的patch與source texture，將自己拍的照片當作target image用Image Quilting做texture transfer

### Steps
1. Do gradient ascent on *conv1_1*, *conv3_1*, and *conv5_1* to generated textures. 
2. Use the generated textures as source textures. Apply Image Quilting to transfer source textures to target images.

### Results
|Target Image|Source Texture:Conv1_1|Source Texture:Conv3_1| Source Texture:Conv5_1|
| --- | --- | --- | --- |
| *(Vertical)* target *(Horizontal)* source |![](https://i.imgur.com/XCB9XyM.jpg)|![](https://i.imgur.com/xq7YyJo.jpg)|![](https://i.imgur.com/qsIU5Jo.jpg)|
|![](https://i.imgur.com/hMXmNFN.jpg)|![](https://i.imgur.com/15xfHck.jpg)|![](https://i.imgur.com/3IrL8CY.jpg)|![](https://i.imgur.com/HqBQdic.jpg)|

|Target Image|Source Texture:Conv1_1|Source Texture:Conv3_1| Source Texture:Conv5_1|
| --- | --- | --- | --- |
| *(Vertical)* target *(Horizontal)* source |![](https://i.imgur.com/xJyCCrz.jpg)|![](https://i.imgur.com/vdBGSDT.jpg)|![](https://i.imgur.com/yaOlxRv.jpg)|
|![](https://i.imgur.com/yHJpMYG.jpg)|![](https://i.imgur.com/mhp7Cg6.jpg)|![](https://i.imgur.com/sEW5mjd.jpg)|![](https://i.imgur.com/EzLZnbh.jpg)|

|Target Image|Source Texture:Conv1_1|Source Texture:Conv3_1| Source Texture:Conv5_1|
| --- | --- | --- | --- |
| *(Vertical)* target *(Horizontal)* source |![](https://i.imgur.com/tSj1y7T.jpg)|![](https://i.imgur.com/f96XV0C.jpg)|![](https://i.imgur.com/1xfwusb.jpg)|
|![](https://i.imgur.com/jWct2Cv.jpg)|![](https://i.imgur.com/QDVGUWi.jpg)|![](https://i.imgur.com/EzvJn4w.jpg)|![](https://i.imgur.com/5F4CWA7.jpg)|

*<center> Photo taken by ourselves to monet using Image Quilting. </center>*

&emsp;&emsp;雖然在接上patch中有根據最小差異路徑做剪接，但是在生成的圖片之仍然可以看到明顯patch的痕跡。原本估計比較`conv5_1`生成比較複雜的texture經過Image Quilting後可以得到效果比較好的圖片，不過實驗後發現效果和使用`conv1_1``conv3_1`texture的生成圖片相差不遠。

---

## Conclusion

&emsp;&emsp;**MUNIT**對random style code生成的圖片缺乏semantic meaning但是能保持原圖片的結構，在生成自然影像上可以產生漂亮的色調跟光線；Munit對referenced style code無法生成對應domain裡的圖片，我們認為可能是在adversarial loss中缺少對referensed style code做評分的關係。

&emsp;&emsp;**Image Quilting**使用非learning的方式做texture transfer，即使有對每個patch銜接邊界做最佳化，生成出的圖片仍有明顯的patch痕跡。

---

## Appendix
<br/><center>![](https://i.imgur.com/O4xOKaN.jpg)</center><br/>

---

## Reference
[1] Li, Y., Liu, M.Y., Li, X., Yang, M.H., Kautz, J.: A closed-form solution to photorealistic image stylization. In: ECCV, 2018.<br>
[2]X. Huang, M.-Y. Liu, S. Belongie, and J. Kautz, “Multi-modal unsupervised image-to-image translation,” arXiv preprint arXiv:1804.04732, 2018.<br>
[3]A. Efros and W.T. Freeman. Image quilting for texture synthesis and transfer. In Proc. ACM Conf. Comp. Graphics (SIGGRAPH), pages 341–346, Eugene Fiume, August 2001.<br>
