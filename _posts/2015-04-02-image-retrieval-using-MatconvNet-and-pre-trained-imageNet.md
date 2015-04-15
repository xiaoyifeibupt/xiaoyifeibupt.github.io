---
layout: post
title: Image retrieval using MatconvNet and pre-trained imageNet
categories: [image retrieval]
---

>MatConvNet is a MATLAB toolbox implementing Convolutional Neural Networks (CNNs) for computer vision applications. It is simple, efficient, and can run and learn state-of-the-art CNNs. Several example CNNs are included to classify and encode images.

MatConvNet是Andrea Vedaldi用Matlab开发的一个卷积网络工具包，相比于[Caffe](caffe.berkeleyvision.org)，这个工具包配置比较简单，而且最近这两年，vgg小组在深度学习领域也是成绩斐然。关于MatConvNet的文档，可以查看[MatConvNet Convolutional Neural Networks for MATLAB]((http://arxiv.org/pdf/1412.4564.pdf)以及[在线文档](http://www.vlfeat.org/matconvnet/)。这里我们主要讲讲怎么配置MatConvNet以及怎样利用在imageNet上已经训练好的模型抽取特征并进行图像检索。

##MatConvnet配置

首先，下载MatConvNet，怎么下载这个自己看着办，如果你有github的账号，推荐你star一下它，既然都用它了，不给个star好意思么(哈哈~)。下载完后，解压，移到某处，本小子自己将它放在`D:\matlabTools\`目录下，打开matlab，进入到`D:\matlabTools\matconvnet-1.0-beta10`目录下，然后在matlab命令窗下输入以下命令进行mex编译：

```matlab
addpath matlab
vl_compilenn
```

如果没出什么问题的话，会在你的`matconvnet-1.0-beta10\matlab`文件夹下多出几个文件夹，其中最重要的是mex，mex文件夹里有编译完成的mex文件`vl_imreadjpeg.mexw64`,`vl_nnconv.mexw64`,`vl_nnnormalize.mexw64`,`vl_nnpool.mexw64`说明编译成功。

**注意**：如果编译失败，可能的原因是你的matlab版本有点低（我在matlab2012b上没有编译成功），这个我查看了一下github上得issures，发觉也有人碰到这样的问题。另外根据Andrea Vedaldi在[error in compiling CPU version with windows 7 + matlab 2014a](https://github.com/vlfeat/matconvnet/issues/92)说的：

>Hi, we never tested a 32-bit build. Is there a particular reason why you are not running MATLAB 64 bit? Note that MATLAB 32 bit will be phased out by Mathwork in one of the upcoming releases. There does not seem much incentive in creating a 32 bit version of the code, although I am sure it could be done with  relatively little effort. Also, in most applications of deep learning 4GB of addressable memory seem a little too little.

所以建议使用64位的matlab，此外在编译的时候，确认mex是否在matlab命令窗里可用，不行的话就setup吧，我用的是vs2010的编译器。

上面这一步完成了，基本就配置完了，下面就测试一下MatConvNet吧。测试的脚本见[http://www.vlfeat.org/matconvnet/pretrained/](http://www.vlfeat.org/matconvnet/pretrained/)给出的脚本例子，即：

```matlab
% setup MtConvNet in MATLAB
run matlab/vl_setupnn

% download a pre-trained CNN from the web
urlwrite('http://www.vlfeat.org/sandbox-matconvnet/models/imagenet-vgg-f.mat', ...
  'imagenet-vgg-f.mat') ;
net = load('imagenet-vgg-f.mat') ;

% obtain and preprocess an image
im = imread('peppers.png') ;
im_ = single(im) ; % note: 255 range
im_ = imresize(im_, net.normalization.imageSize(1:2)) ;
im_ = im_ - net.normalization.averageImage ;

% run the CNN
res = vl_simplenn(net, im_) ;

% show the classification result
scores = squeeze(gather(res(end).x)) ;
[bestScore, best] = max(scores) ;
figure(1) ; clf ; imagesc(im) ;
title(sprintf('%s (%d), score %.3f',...
net.classes.description{best}, best, bestScore)) ;
```

上面用的是`urlwrite`来下载`imagenet-vgg-f.mat`的，这里强烈推荐你单独下载，然后把`urlwrite`下载的那一行去掉，`load`时指向你放置的`imagenet-vgg-f.mat`具体位置即可。测试如果顺利的话，就可以进入下一节我们真正关心的图像减速话题了。


##用已训练模型抽取特征

在抽取特征之前，有必要稍微先来了解一下`imagenet-vgg-f`这个模型。这里稍微啰嗦一下上面的那个测试脚本，`im_ = imresize(im_, net.normalization.imageSize(1:2))`将图像缩放到统一尺寸，即224*224的大小，这点你可以看看net中`normalization.imageSize`，而且还必须为彩色图像。**res**有22个struct，从第17到20的struct分别是4096位，最后第21到22个struct是1000维的，是4096维经过softmax后的结果，这里我们要用的是第20个struct的数据(自己测过第19个struct，检索效果比采用第20个struct的特征差）。这个网络有8层构成，从第6层到第8层都是全连接层。关于这个网络的结构，暂时到这里。

大致了解了这个网络结构后，我们便可以使用该网络抽取图像的特征了，抽取特征的代码(**完整的图像检索代码见文末最后给出的链接**)如下：

```matlab
% Author: Yong Yuan
% Homepage: yongyuan.name

clear all;close all;clc;

run D:/matlabTools/matconvnet-1.0-beta10/matlab/vl_setupnn %换成你的vl_setupnn.m具体路径

%% Step 1 lOADING PATHS
path_imgDB = './database/';
addpath(path_imgDB);
addpath tools;

net = load('imagenet-vgg-f.mat') ; %换成你下载的imagenet-vgg-f.mat具体路径
 
%% Step 2 LOADING IMAGE AND EXTRACTING FEATURE
imgFiles = dir(path_imgDB);
imgNamList = {imgFiles(~[imgFiles.isdir]).name};
clear imgFiles;
imgNamList = imgNamList';

numImg = length(imgNamList);
feat = [];
k = 0;
rgbImgList = {};

for i = 1:numImg
   oriImg = imread(imgNamList{i, 1}); 
   tmpNam = imgNamList{i, 1};
   if size(oriImg, 3) == 3
       k = k+1;
       im_ = single(oriImg) ; % note: 255 range
       im_ = imresize(im_, net.normalization.imageSize(1:2)) ;
       im_ = im_ - net.normalization.averageImage ;
       res = vl_simplenn(net, im_) ;
       featVec = res(19).x;
       featVec = featVec(:);
       feat = [feat; featVec'];
       fprintf('extract %d image\n\n', i);
       rgbImgList{k,1} = tmpNam;
   else
       rgbImg(:,:,1) = oriImg;
       rgbImg(:,:,2) = oriImg;
       rgbImg(:,:,3) = oriImg;
       k = k+1;
       im_ = single(rgbImg) ; % note: 255 range
       im_ = imresize(im_, net.normalization.imageSize(1:2)) ;
       im_ = im_ - net.normalization.averageImage ;
       res = vl_simplenn(net, im_) ;
       featVec = res(19).x;
       featVec = featVec(:);
       feat = [feat; featVec'];
       fprintf('extract %d image\n\n', i);
       rgbImgList{k,1} = tmpNam;
       clear rgbImg;
   end
end

feat = normalize1(feat);
save('Feat4096Norml.mat','feat', 'rgbImgList', '-v7.3');
```

在倒数第二行，对特征进行了L2归一化，方便后面用余弦距离度量，L2归一化方法如下：

```matlab
function [X] = normalize1(X)
% X:n*d

for i=1:size(X,1)
    if(norm(X(i,:))==0)
        
    else
        X(i,:) = X(i,:)./norm(X(i,:));
    end
end
```

上面便是特征抽取的代码，特征抽取完后，我们便可以采用进行检索了。匹配时采用的是余弦距离度量方式，至于后面的检索部分的代码，相比于前面特征抽取的过程，更显灵活，怎么处理可以随自己的喜欢了，所以这里就不再对代码进行列举了。下面是我在**corel1k**外加上Caltech256抽取几类构成1333张图像库做的一个飞机检索结果，时间所限，就不测大的图像库了。

![airplane-image-retrieval]({{ site.url }}/images/posts/2015-04-02/airplane-image-retrieval.jpg)

最后，整个图像检索的代码已放在github上了，感兴趣的同学可以去[**CNN-for-Image-Retrieval**](https://github.com/willard-yuan/CNN-for-Image-Retrieval)，有github的同学不要吝啬你的star哦，这个代码我会随时完善更新，比如添加计算mAP的代码。