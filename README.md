# Android 开发之答题卡检测  
## 写在最前 ##
做这个项目的初衷是为了来年的实习做准备，自己想着能做出点东西来，当作实习的敲门砖，或者至少让自己的简历看起来更丰富。因为我希望以后能过从事安卓的开发，所以就想着通过项目来自学安卓开发。想到做这个项目，也是因为我之前的项目有用过 opencv，所以就想把这两样东西结合起来，就有了这个点子。  


## 最初的设想 ##
先介绍一下这个项目设想吧。项目主要是想实现多张答题卡的同时检测，并记录不同答题卡的信息，包括得分，错误题目等。想象一下，你是一个英语老师，带三个班，每一次的测验，你都得改150份题目，而且这些题目大部分都是选择题，改完之后也许还希望统计每一题的正确率是多少。再如果，有某一个学生是你重点关注的对象，你希望知道他到底哪些知识薄弱，所以你想知道他每次测验做错的题目是哪些方面的题目，就得记录他每次测验的错题，然后再逐题分析。如果只有一二十个学生，这些工作量不算什么，但是如果有上百个学生，估计每次测验完你只想把所以题目赶紧改完。  
这时候如果有一个软件，每次可以扫描多张答题卡，能自动记录每张答题卡的个人信息（比如学号什么的），能自动帮你计算每一张答题卡的得分，帮你统计每一题的正确率，并且记录每一个学生多次测验的错题，肯定会很方便。  
  
## 效果图 ##
![答题卡_54题](https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/scan_1_paper.gif)  
<img src="https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/Screenshot_54_choose_pic.png" width="50%" height="50%"><img src="https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/Screenshot_54_wrap_pic.png" width="50%" height="50%"><img src="https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/Screenshot_54_err_pic.png" width="50%" height="50%"><img src="https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/Screenshot_54_corr_pic.png" width="50%" height="50%"><img src="https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/Screenshot_54_save_data.png" width="50%" height="50%">         
![答题卡_21题](https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/scan_2_paper.gif)  
<img src="https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/Screenshot_21_choose_pics.png" width="50%" height="50%"><img src="https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/Screenshot_21_wrap_pics.png" width="50%" height="50%"><img src="https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/Screenshot_21_err_pics.png" width="50%" height="50%"><img src="https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/Screenshot_21_corr_pics.png" width="50%" height="50%"><img src="https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/Screenshot_21_save_data.png" width="50%" height="50%"> 
## 答题卡的设计 ##
这个部分是最基础的部分，后面核心的检测代码得围绕着这个部分来写。  
###目标功能：  
>用户可以创建自己的答题卡，通过弹窗和滑动条，选择答题卡的名字、大小、题目数量、正确答案等信息。根据这些信息，自动生成对应的二维码，之后的答题卡检测可以通过检测二维码得到这些信息。  


###已实现功能：  
>没有加入弹窗和滑动条来让用户输入正确答案和修改题目数量，只能在程序里修改题目数量、答题卡的名字和正确答案。然后可以根据提供的这些信息，自动生成二维码，同时把正确答案存入数据库中。不把正确答案加入二维码中有两个原因：1.如果题目过多，会让二维码携带大量的信息，这会导致二维码变得复杂，增加解码的难度。2.二维码中携带正确答案后，所有的人都可以通过扫描该二维码得到正确答案的信息，这对于出题老师来说不是什么好事。    

![答题卡_54题](https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/AnswerSheet_54.png)  
这些答题卡的样式很简单。左上角第一个框是学生的名字；下边的框是存放二维码；右边的图取框，是学生的学号；再右边是学科。对于不同题目数量的答题卡，我们根据题目数量，计算出所需的行数，再算出边框的高度。二维码的生成，我选择用 zxing 的 qr 二维码来进行封装，我们可通过 builder 来修改二维码的信息，不同的信息之间，用“：”来分隔，这样方便之后的提取。新建答题卡成功后，会在把答题卡储存在 sd 卡里，方便打印，因为设计的时候，答题卡的大小是根据 A4 纸来设计的，所以复制图片后直接打印就可以了。  
代码不是很复杂，就不啰嗦了，直接参见 CreateAnswerSheet.java 中。  

## 答题卡的检测 ##
这个部分可以算是整个代码的核心部分了。答题卡的检测可以分为两个部分，第一个部分用 ndk 方法调用 native 层的 opencv 代码，实现每一份答题卡的识别、答题卡图取的识别和答题卡中二维码位置的识别。第二个部分是在 java 层实现的，主要功能为：压缩原图片，获取合适大小的检测图片（不在 native 层实现是为了避免传送过大的文件而浪费内存），然后把压缩后的图片传给 native 层；native 层检测结束后，获取检测到的二维码，然后进行解码，得到正确答案，再与 native 层得到的学生图取的答案进行比较。  
###已实现功能：  
>能够同时检测多张答题卡，对于每一张答题卡，能够检测出答题卡选项的图取情况和学生的学号，并解码每一张图片里的二维码。  


###opencv 的检测部分：
* 确认图片中是否包含答题卡   
检测的第一步就是需要确定载入的图片中是否包含我们的答题卡。这里，我通过检测图片中是否包含合适大小的四边形来确认，这个方法很粗糙，所以得到的结果自然不尽人意，简单来说，这种方法会把所有的合适大小的四边形当作答题卡，比如一本书、一张纸或者其他的什么东西。要解决这个问题，就需要加入二维码的检测：检测到四边形后，开始检查该四边形内是否有二维码，如果有，检测该二维码，就可以该矩形是否是答题卡了。  
    
* canny 检测  
  想要检测四边形，就需要得到合适的轮廓，找轮廓之前，先进行 canny() 检测，得到合适的边缘。考虑不同的检测环境，我们不能直接用 canny() 函数来检测边缘。因为在不同的光源下，得到的边缘可能不是我们想要的，举一个例子：  
  ![canny_source](https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/example_canny_source.png)   
图片中，有一块区域是光线比较强的，如果我们直接用 canny() 来获取边缘，我们会得到以下的结果：  
![canny_result](https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/example_canny_result.png)  
在这个图片中，注意观察和光线交界的那一条边，它有些地方是不闭合的，也就是说我们的 A4 纸的边缘其实不是一条直线。如果我们直接利用这个图像来寻找最大和第二大的轮廓的话，我们会得到很诡异的情况：  
![canny_contours](https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/canny_draw_contours.png)  
这是因为用 canny() 方法得到的边缘，如果目标物体本身的轮廓不是特别清晰的话，我们的到的边缘实际上会存在断点的，所以我们会在 canny() 方法前需要先处理图像。这就是说直接使用 canny() 检测，一些情况下是可行的，但是对于一些特殊的环境，效果会很差。  
为了排除不同光线的干扰，我们需要采取别的方法。之前我有考虑过通过颜色的检测来区分黑色和其他颜色，从而得到合适的四边形。但是这个方法比直接检测边缘还不靠谱。因为在不同的光源下，我们人眼能够准确的区分黑色和其他的颜色，但是利用 rgb 色系 或者 hsv 色系检测的话，得到的结果会让人大失所望。  
最后确认的方法是：在用 canny() 边缘检测之前，我们需要处理图像，找出包含高水平梯度和低垂直梯度的图像，然后二值化，接着进行膨胀和腐蚀的操作，最后在进行 canny() 操作，就可以得到较为清晰的轮廓。  
![new_canny](https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/new_canny_contours.png)  
可以看到，处理之后图像即使是在光线较强的那一块区域里，也能得到完整的一条直线。这样再提取最大和第二大的轮廓，我们可以得到以下的结果   
![new_canny](https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/new_canny_result.png)  
* 四边形的判定  
opencv 里，有 findContours() 可以让我们提取出 canny() 检测之后的轮廓，函数中有一个参数可以设置查找的方法，其中有一个方法是只查找最外层的轮廓,例如这样：  
![contours_src](https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/findContours_src.jpg)
![contours_result](https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/findContours_result.jpg)   
可是这样的到的结果有一个问题，就是得到的轮廓其实是白纸的最外层轮廓，所以我们的到了这个轮廓之后还要进行第二次检测。上图中我们可以清楚的看到第三大的轮廓就是答题卡的黑色边框，但是为什么不利用轮廓的面积来确定答题卡的轮廓，而是再进行一次检测呢？这样做有两个原因：1.如果用户扫描的时候，没有把白纸的轮廓完全包含，那么第三大的轮廓就不是我们需要寻找的轮廓。2.对于多张图片的扫描，用轮廓面积来确定目标轮廓，很容易出错。  
有了外层轮廓，我们就接着检测轮廓的面积，如果轮廓面积过小，我们认为，这个轮廓不是我们要找的答题卡。确定了合适的轮廓之后，可以直接调用 approxPolyDP() 来判断的到的轮廓是否是四边形，如果轮廓是四边形，我们提取它的四个角点。接着检测这四个角点之内的图像，看他里面的图像是否也包含一个合适的四边形（面积大于图像的百分之七十），如果有，我们判定第二个检测到的轮廓是我们的目标轮廓，如果没有，我们判定第一个检测到的轮廓是目标轮廓。 
对于两张以上答题卡的查找，只要把上面的重复上面的步骤就可以了。   
* 图像的转置  
确定了四边形后，我们提取它的四个角点，然后构建一个转置矩阵，把四个角点的图像转置到我们新的图像中。利用到的方法有 getPerspectiveTransform() 和 warpPerspective()。  
* 查找二维码  
虽然 zxing 的二维码检测可以检测整张图片内是否包含二维码，如果有包含，直接就会解码出结果。但是我们如果直接解码整张图片会浪费大量的内存，因为我们的二维码其实只占整张图像 0.18 倍左右。还有一个问题，对于多张图片的检测，如果图片包含两张以上二维码，将会出错。所以我们必须得到我们的二维码的图片。  
二维码的查找在得到转置图像后进行，主要的步骤有：灰度处理、二值化、膨胀、腐蚀、查找最大的轮廓、提取角点、和图像转置。  
* 选项和学号图取的检测  
这个部分也是在得到转置图像后进行，核心思想其实很简单：对于每一个选项，我们取一个合适的检测框，计算在该框内，图取的像素总数和该框的像素总数，如果图取的像素占了像素总数的一定量后，我们判定该选项已经被图取。目前选项只支持单选题，也就是说，如果一个问题同时图取了两个答案，我们判定这个问题没有被图取。  
这个部分有两个要点，一是图像的处理：进行图像二值化之前，我们需要调整一下图像的亮度，如果没有这个步骤，亮度会很大程度上影响二值化的结果，从而导致检测出错。二是检测框的选区：我们可以参考答题卡创建时，每一个选项的位置，但是作为检测我们需要把检测框选得大一些，这样可以有一定的容错率。   


## 流程图 ##
![process](https://github.com/TuXin-GitHub/DetectAnswerSheets/blob/master/image/process.png) 